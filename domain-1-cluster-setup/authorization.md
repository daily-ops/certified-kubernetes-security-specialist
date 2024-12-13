
#### Step 1 - Enable AlwaysDeny Authorization Mode

```sh
nano /etc/systemd/system/kube-apiserver.service
```
```sh
--authorization-mode=AlwaysDeny 
```

```sh
systemctl daemon-reload
systemctl restart kube-apiserver
```
```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/alice.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/alice.key
```
A valid certificate is not sufficient for the `AlwaysDeny` authorization mode.
```
root@kubernetes:~/certificates# kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/alice.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/alice.key
Error from server (Forbidden): secrets is forbidden: User "alice" cannot list resource "secrets" in API group "" in the namespace "default": Everything is forbidden.
root@kubernetes:~/certificates# 
```
#### Step 2 - Enable RBAC Authorization Mode

```sh
nano /etc/systemd/system/kube-apiserver.service
```
```sh
--authorization-mode=RBAC
```
```sh
systemctl daemon-reload
systemctl restart kube-apiserver
```
```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/alice.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/alice.key
```
#### Step 3 - Create Super User

```sh
cd /root/certificates
openssl genrsa -out bob.key 2048
openssl req -new -key bob.key -subj "/CN=bob/O=system:masters" -out bob.csr
openssl x509 -req -in bob.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out bob.crt -extensions v3_req  -days 1000
```
#### Step 4 Verification:

```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```
In constrast of `alice` user above, the `bob` user with `system:masters` group is authorized to perform the operation.
```
root@kubernetes:~/certificates# kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
No resources found in default namespace.
root@kubernetes:~/certificates# 
```