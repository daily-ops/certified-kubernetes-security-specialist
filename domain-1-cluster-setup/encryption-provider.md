#### Important notes
- `etcd` provides no encryption mechanism
- The encryption needs to be done at API server though Encryption Provider

#### Documentation Referred:

https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

##### Step 1: Create a new secret
```sh
kubectl create secret generic new-secret -n default --from-literal=user=secretpassword --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```
```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```

##### Step 2: Find the Secret in ETCD in Plain-Text
```sh
cd /root/certificates
```
```sh
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert ./apiserver.crt --key ./apiserver.key get /registry/secrets/default/new-secret | hexdump -C
```

```
root@kubernetes:~/certificates# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert ./apiserver.crt --key ./apiserver.key get /registry/secrets/default/new-secret | hexdump -C
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6e 65 77 2d 73 65  |s/default/new-se|
00000020  63 72 65 74 0a 6b 38 73  00 0a 0c 0a 02 76 31 12  |cret.k8s.....v1.|
00000030  06 53 65 63 72 65 74 12  d6 01 0a b3 01 0a 0a 6e  |.Secret........n|
00000040  65 77 2d 73 65 63 72 65  74 12 00 1a 07 64 65 66  |ew-secret....def|
00000050  61 75 6c 74 22 00 2a 24  33 61 65 62 31 30 61 36  |ault".*$3aeb10a6|
00000060  2d 61 35 65 63 2d 34 39  63 30 2d 38 31 34 34 2d  |-a5ec-49c0-8144-|
00000070  65 34 37 32 64 30 37 30  65 36 39 32 32 00 38 00  |e472d070e6922.8.|
00000080  42 08 08 da fe ee ba 06  10 00 7a 00 8a 01 61 0a  |B.........z...a.|
00000090  0e 6b 75 62 65 63 74 6c  2d 63 72 65 61 74 65 12  |.kubectl-create.|
000000a0  06 55 70 64 61 74 65 1a  02 76 31 22 08 08 da fe  |.Update..v1"....|
000000b0  ee ba 06 10 00 32 08 46  69 65 6c 64 73 56 31 3a  |.....2.FieldsV1:|
000000c0  2d 0a 2b 7b 22 66 3a 64  61 74 61 22 3a 7b 22 2e  |-.+{"f:data":{".|
000000d0  22 3a 7b 7d 2c 22 66 3a  75 73 65 72 22 3a 7b 7d  |":{},"f:user":{}|
000000e0  7d 2c 22 66 3a 74 79 70  65 22 3a 7b 7d 7d 42 00  |},"f:type":{}}B.|
000000f0  12 16 0a 04 75 73 65 72  12 0e 73 65 63 72 65 74  |....user..secret|
00000100  70 61 73 73 77 6f 72 64  1a 06 4f 70 61 71 75 65  |password..Opaque|
00000110  1a 00 22 00 0a                                    |.."..|
00000115
```

```sh
cd /var/lib/etcd
grep -R "secretpassword" .
```

```
root@kubernetes:/var/lib/etcd# grep -R "secretpassword" .
grep: ./member/snap/db: binary file matches
grep: ./member/wal/0000000000000000-0000000000000000.wal: binary file matches
root@kubernetes:/var/lib/etcd# 
```
#### Step 3 - Create Encryption Key:
```sh
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
echo $ENCRYPTION_KEY
```
#### Step 4 - Create Encryption Config:
```sh
cat > encryption-at-rest.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
#### Step 5 - Copy configuration to appropriate path:
```sh
mkdir /var/lib/kubernetes
mv encryption-at-rest.yaml /var/lib/kubernetes
```
#### Step 6 - Start kube-apiserver with appropriate flag:

  ```sh
nano /etc/systemd/system/kube-apiserver.service
```
```sh
--encryption-provider-config=/var/lib/kubernetes/encryption-at-rest.yaml
```
```sh
systemctl daemon-reload
systemctl restart kube-apiserver
systemctl status kube-apiserver
```
The status output.
```
root@kubernetes:~/certificates# systemctl daemon-reload
systemctl restart kube-apiserver
systemctl status kube-apiserver
● kube-apiserver.service - Kubernetes API Server
     Loaded: loaded (/etc/systemd/system/kube-apiserver.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-12-13 05:04:53 UTC; 16ms ago
       Docs: https://github.com/kubernetes/kubernetes
   Main PID: 5635 (kube-apiserver)
      Tasks: 4 (limit: 2309)
     Memory: 776.0K
        CPU: 4ms
     CGroup: /system.slice/kube-apiserver.service
             └─5635 /usr/local/bin/kube-apiserver --advertise-address=159.65.147.161 --etcd-cafile=/root/certificate>

Dec 13 05:04:53 kubernetes systemd[1]: Started Kubernetes API Server.
```
#### Step 7 - Create a new Secret
```sh
kubectl create secret generic db-secret -n default --from-literal=dbadmin=dbpasswd --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```
```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```
##### Step 8: Verify if you can find secret

```sh
cd /root/certificates
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert ./apiserver.crt --key ./apiserver.key get /registry/secrets/default/db-secret | hexdump -C
```
The output shows no plain text.
```
root@kubernetes:~/certificates# cd /root/certificates
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert ./apiserver.crt --key ./apiserver.key get /registry/secrets/default/db-secret | hexdump -C
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 64 62 2d 73 65 63  |s/default/db-sec|
00000020  72 65 74 0a 6b 38 73 3a  65 6e 63 3a 61 65 73 63  |ret.k8s:enc:aesc|
00000030  62 63 3a 76 31 3a 6b 65  79 31 3a fc 80 40 b7 84  |bc:v1:key1:..@..|
00000040  b3 81 ea 8b 3f 55 a0 e2  a9 4a 87 d2 ce 3f 98 8a  |....?U...J...?..|
00000050  85 2d 34 a0 05 23 d4 6f  41 a0 63 3d 96 c8 d2 d3  |.-4..#.oA.c=....|
00000060  42 4d 5a e2 ec a2 b6 dd  a7 37 83 31 93 f2 03 52  |BMZ......7.1...R|
00000070  44 ff 72 ef 5a 74 c6 25  d5 b4 07 16 2d 8e 64 b3  |D.r.Zt.%....-.d.|
00000080  18 49 4c a6 b0 cf 58 c1  42 6b 57 5d d0 85 f3 9f  |.IL...X.BkW]....|
00000090  da 4e 48 e1 79 9a 8c a8  a5 c4 6a 53 8b 3a 0a ef  |.NH.y.....jS.:..|
000000a0  e1 3d 89 f4 a4 c9 50 a8  2c 2c 09 6a ed 15 1b b2  |.=....P.,,.j....|
000000b0  15 a6 9f e6 9d 26 9e 88  eb 38 ec d2 34 3e f9 06  |.....&...8..4>..|
000000c0  f3 c3 fc 54 1f 33 42 2a  80 1e 4a 48 17 30 ab a4  |...T.3B*..JH.0..|
000000d0  56 78 6c 9c 25 fe 79 e4  19 0b 5c 34 84 9c 8f 49  |Vxl.%.y...\4...I|
000000e0  91 f3 6e 33 3e ba b8 03  5c 31 5c 3d 1f ae ce ec  |..n3>...\1\=....|
000000f0  14 dd fd 1a 54 38 01 21  95 f2 f1 67 b4 ef 17 0e  |....T8.!...g....|
00000100  ef f8 05 52 2d 05 88 01  1e 09 c5 d3 ea c2 78 44  |...R-.........xD|
00000110  30 4d b6 57 89 75 dd 16  6a 3b 5a c2 83 09 2a 6e  |0M.W.u..j;Z...*n|
00000120  b7 87 11 47 69 3f d7 83  3b 2d 50 34 8e 91 03 ba  |...Gi?..;-P4....|
00000130  fd 88 92 b4 fc b9 23 c6  82 a7 c0 0a              |......#.....|
0000013c
root@kubernetes:~/certificates# 
```
```sh
cd /var/lib/etcd
grep -R "dbpasswd" .
```
No plain text stored in the data files.
```
root@kubernetes:~/certificates# cd /var/lib/etcd
grep -R "dbpasswd" .
root@kubernetes:/var/lib/etcd# 
```