As noted earlier, clients of Kubernetes cluster do not interact directly with `etcd`. The API server exposes services for client to consume and using `etcd` as distributed storage, therefore the API server itself is a client of `etcd` server. In order to communicate with an already secured etcd service previously configured, API server is required to have client certificate in which signed by the same Kubernetes cluster CA as `etcd` service for validation process.

#### Step 1: Download Kubernetes Server Binaries
```sh
cd /root/binaries
wget https://dl.k8s.io/v1.24.2/kubernetes-server-linux-amd64.tar.gz
tar -xzvf kubernetes-server-linux-amd64.tar.gz
cd /root/binaries/kubernetes/server/bin/
cp kube-apiserver kubectl /usr/local/bin/
```

#### Step 2 - Generate Client Certificate for API Server (ETCD Client):
```sh
cd /root/certificates
```
```sh
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -extensions v3_req  -days 1000
```
The certificate.
```
root@kubernetes:~/certificates# cat apiserver.crt 
-----BEGIN CERTIFICATE-----
MIICuDCCAaACFHB+D12YE0Ogdy1NTyNIhK88kF+8MA0GCSqGSIb3DQEBCwUAMBgx
FjAUBgNVBAMMDUtVQkVSTkVURVMtQ0EwHhcNMjQxMjEyMjMyMzIxWhcNMjUwMjEw
MjMyMzIxWjAZMRcwFQYDVQQDDA5rdWJlLWFwaXNlcnZlcjCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAKUUUTyEBmhD01f46VEvs9b+ccD4Oc+yV4Qj1KNU
3PgxOnKGHIAInzbLGnWtzuFzMy5XPY9BimWbAMDm4cF3IrAV12hbKd2UHLbSCmAe
dXs1xKN/rbH4mKPgP3HYfUGSJ1lVcau6AKYumXxux0oIDCwl2wxcjjqqu2HQkDjE
yR6QbePu/yfYL8wNFkVCpzUv8/W1ET/46mQfw13yu9/YQ+f2HNrrVIzlrH5CW1/p
eWOOJYfqdoJitVtzfaHmm4yZRJT9ji2i05kwdyAsAOqhrAJoFHvUJSwkvpGrkOlj
tXUMqFldQ4p8ywjjOZICt73SbuiH+GCn1JHJTbgaHmzCRm0CAwEAATANBgkqhkiG
9w0BAQsFAAOCAQEAKPz/yJO/+4DewV15zHYnbJ2vCcqwtJaMAM1ZSs8Y66Z0mWZ+
YdhJqa/fE6ogREVkRwgwEXPrPGsBQ+QK/gzBnCsAxyTOpqRduS6hNoccba8UwU/p
nF9PrFQ0Y0HsLHPKBaO68aZbjBPqWTqLD1zAFHpuEznmOmQtVzMUtOi/ObhswAgy
OP/KBMJ0J8GZfW2A2JNGPwr4cDw3rurDxbwUNVAiKuFecazJ+1Zr2mqMFvM4fp7C
+e+8ZJq5VrqWndUnn7P4F5ET2WrewQjo0DzaP8BiCE5cyBoOpRlv+mTNLX88vDTC
gsHKWO4pqAJDAWxHERtyITjmWrhjjufN0TN53g==
-----END CERTIFICATE-----
```

#### Step 3 - Generate Service Account Certificates
```sh
openssl genrsa -out service-account.key 2048
openssl req -new -key service-account.key -subj "/CN=service-accounts" -out service-account.csr
openssl x509 -req -in service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out service-account.crt -days 100
```
The certificate.
```
root@kubernetes:~/certificates# cat cat service-account.crt 
cat: cat: No such file or directory
-----BEGIN CERTIFICATE-----
MIICujCCAaICFDZSGJplhIIvUx4V+vJcJLCcjOkQMA0GCSqGSIb3DQEBCwUAMBgx
FjAUBgNVBAMMDUtVQkVSTkVURVMtQ0EwHhcNMjQxMjEyMjMyOTA0WhcNMjUwMjEw
MjMyOTA0WjAbMRkwFwYDVQQDDBBzZXJ2aWNlLWFjY291bnRzMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsv5eYcWSfSo1EFbhHmdz652KT4Wns2c4bb01
sSnoSPUH44OI8dXHvsDjfgn9ZBonH8LVdIBcGz7sv5lFMvQETXyrhDCe23XMzqRy
RhTfsTSFDXjpcVvMFutaO1QB4mu1el/5pGi2I2QM8AJrzircKB4IS1J0gfIAgnxn
lXPLBoHwjrhQTNBpFPLGcE/DX/oJTDPZO3SdjVnwXe0zt0704jrKAlttXfOlHtg/
Dm/6tuVfXdubowHgtHdNeMhphHqcqFd1WRwK1LuPVM/9RE41tgIfwT/TOjewzd+S
buLAsIgzFx5pALDm21h9GVxm1j7bFipod9KdBlhOlIkBPK5AbQIDAQABMA0GCSqG
SIb3DQEBCwUAA4IBAQAE3eDrNgBtU+HRvgJb0pq1g39xbCZjWv1MdlORbA2HLXqE
39T9ueqmFNFC9LzYsTZDKc7R2thk1xUn3IOCtPLRP9H2nEonCtF6atKWuDjjfzym
SYfpAhoTAX8gbuHUcy4Ffkp9L2dstjvj9Fyew2H2B5fsOR6EIF6sObS0n86wuA6s
YYq4RCzcLgVkw1Fb4bKt4Mc7GXppPK6D1LpM73F3a2RikYQcfEFqmu+Wdpw6rm72
CJHQb2xmTOIjZdc3McOqIbO37wegCYK/T6U9Frurrh9YJXMxHmCjCEltDWpHIelC
jNs7xZtHhbGMX299/yTNXWjgyTtWHvcAMhWilFO6
-----END CERTIFICATE-----
```
#### Step 4 - Start kube-apiserver:
```sh
/usr/local/bin/kube-apiserver --advertise-address=159.65.147.161 --etcd-cafile=/root/certificates/ca.crt --etcd-certfile=/root/certificates/apiserver.crt --etcd-keyfile=/root/certificates/apiserver.key --service-cluster-ip-range 10.0.0.0/24 --service-account-issuer=https://127.0.0.1:6443 --service-account-key-file=/root/certificates/service-account.crt --service-account-signing-key-file=/root/certificates/service-account.key --etcd-servers=https://127.0.0.1:2379
```
#### Step 5 - Verify

```sh
netstat -ntlp
curl -k https://localhost:6443
```
The `netstat` command shows the API server is listening on the port `6443` as expected.
```
root@kubernetes:~/certificates# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN      4501/etcd           
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      4501/etcd           
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      2171/systemd-resolv 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2205/sshd: /usr/sbi 
tcp6       0      0 :::6443                 :::*                    LISTEN      4605/kube-apiserver 
tcp6       0      0 :::22                   :::*                    LISTEN      2205/sshd: /usr/sbi 
root@kubernetes:~/certificates# 
```
The `curl` command returned with unauthorized error but the API server is reacheable and servicing.
```
root@kubernetes:~/certificates# curl -k https://localhost:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}root@kubernetes:~/certificates# 
```

#### Step 6 - Integrate Systemd with API server

Change the IP address in --advertise-address

```sh
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
--advertise-address=159.65.147.161 \
--etcd-cafile=/root/certificates/ca.crt \
--etcd-certfile=/root/certificates/apiserver.crt \
--etcd-keyfile=/root/certificates/apiserver.key \
--etcd-servers=https://127.0.0.1:2379 \
--service-account-key-file=/root/certificates/service-account.crt \
--service-cluster-ip-range=10.0.0.0/24 \
--service-account-signing-key-file=/root/certificates/service-account.key \
--service-account-issuer=https://127.0.0.1:6443 

[Install]
WantedBy=multi-user.target
EOF
```

#### Step 7 - Start kube-api server
```sh
systemctl start kube-apiserver
systemctl status kube-apiserver
```
The status of kube-apiserver service shows successfully started.
```
root@kubernetes:~/certificates# systemctl status kube-apiserver.service 
● kube-apiserver.service - Kubernetes API Server
     Loaded: loaded (/etc/systemd/system/kube-apiserver.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-12-12 23:36:06 UTC; 2s ago
       Docs: https://github.com/kubernetes/kubernetes
   Main PID: 4748 (kube-apiserver)
      Tasks: 9 (limit: 2309)
     Memory: 280.9M
        CPU: 2.427s
     CGroup: /system.slice/kube-apiserver.service
             └─4748 /usr/local/bin/kube-apiserver --advertise-address=159.65.147.161 --etcd-cafile=/root/certificate>

Dec 12 23:36:09 kubernetes kube-apiserver[4748]: I1212 23:36:09.456782    4748 establishing_controller.go:76] Starti>
Dec 12 23:36:09 kubernetes kube-apiserver[4748]: I1212 23:36:09.456872    4748 nonstructuralschema_controller.go:192>
```
#### Step 8 - Verify (again)

```sh
netstat -ntlp
curl -k https://localhost:6443
```
Same result shown indicating the service is running properly.
```
root@kubernetes:~/certificates# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN      4501/etcd           
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      4501/etcd           
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      2171/systemd-resolv 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2205/sshd: /usr/sbi 
tcp6       0      0 :::6443                 :::*                    LISTEN      4748/kube-apiserver 
tcp6       0      0 :::22                   :::*                    LISTEN      2205/sshd: /usr/sbi 
root@kubernetes:~/certificates# curl -k https://localhost:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}root@kubernetes:~/certificates# 
```