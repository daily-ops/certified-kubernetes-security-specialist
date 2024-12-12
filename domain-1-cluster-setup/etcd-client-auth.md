Imagine we require passport with photo and fingerprints to identify ourselves when entering or exiting a country, similarly, in order to consume APIs offered by `etcd`, it should require a mechanism to identify the client who is coming in to consume and retrieve data stored in etcd to protect and allow only known and trusted clients with the valid identity document issued by Kubernetes cluster therefore a client certificate is required to interact with `etcd`.

#### Step 1 - Generate Client Certificate and Client Key:
```sh
cd /root/certificates
```
```sh
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/CN=client" -out client.csr
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -extensions v3_req  -days 60
```
The generated certificate.
```
root@kubernetes:~/certificates# cat client.crt 
-----BEGIN CERTIFICATE-----
MIICsDCCAZgCFB1zW736gdwnJSeK/4lvgTKcU4i2MA0GCSqGSIb3DQEBCwUAMBgx
FjAUBgNVBAMMDUtVQkVSTkVURVMtQ0EwHhcNMjQxMjEyMjExMDEwWhcNMjUwMjEw
MjExMDEwWjARMQ8wDQYDVQQDDAZjbGllbnQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQCuCnNbJPxTv18H8V219nCkCZqq96qYVVCcc/t097SzRdMoqPaC
7XacTNdcXt4a3OuhXPn4p+ccFnrvVNGtJXJhJziyUlce0Mc9Yn/W0I+aPx7MCeiD
A6V3b0bGaSNrWaAuTn01Qy4DCESGLwW3ChB9eQlBTmI0aQRzuCQRVImJN5LE5as7
ch6wANMBt6Wj4NhZiyexFMHlQObqM2t6RIvng2exsE5CbyYokK694K6XWeI2nZtW
QxlxEOF85edsDkAKhzHSQzlR+2YpqTOVOqIVZ8bzmwA71MGh+N109Y/cpmypCYP/
H/qg3KqX8OkKZ27F87xe3C7XaxREcb51B4PbAgMBAAEwDQYJKoZIhvcNAQELBQAD
ggEBAHeUbZIiUAvy58g5EqwJDdNZHQZYZil/lQas98rpxXo2L29OS0Mtnaxmx2oI
lzyn8pdicQvsX1XRWMkF5zafHYuKMLLFfoVK482wXJWhiCwbTGxRLnd7ErPJuqDE
6Eqa3feM6sFNMg33BbtbSVgg/0Dpm+AkgA7CnCodTV/fw0ybphAoWHSkBJwK4O/Q
MiLB6rXdriTG2Svjj0Hmw4eVm4C3PjW8YfLDaWkaz4eup60wHUNmMYEztXoGG/Gm
OW0edGJ1fPEDDfG0GKvwjFln00J+YnKxTuBnNShScXTsyaIpI7GmPUQmH/eJ8fUw
4pFMstM/N8SCs1w2pk4MQZd5AWM=
-----END CERTIFICATE-----
```

#### Step 2 - Start ETCD Server:
```sh
etcd --cert-file=/root/certificates/etcd.crt --key-file=/root/certificates/etcd.key --advertise-client-urls=https://127.0.0.1:2379 --client-cert-auth --trusted-ca-file=/root/certificates/ca.crt  --listen-client-urls=https://127.0.0.1:2379
```
#### Step 3 - Authenticate with Certificates:
The access to `etcd` service is failing when attempt to set data without client certificate via `etcdctl` as shown with `bad certificate` error.
```sh
root@kubernetes:~/certificates# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false  put testclient data
{"level":"warn","ts":"2024-12-12T21:12:32.672Z","logger":"etcd-client","caller":"v3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0003b2a80/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: last connection error: connection error: desc = \"transport: authentication handshake failed: remote error: tls: bad certificate\""}
Error: context deadline exceeded
root@kubernetes:~/certificates# 
```
The error due to the missing client certificate is highly visible on the server side.
```
{"level":"warn","ts":"2024-12-12T21:12:28.693Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"127.0.0.1:45942","server-name":"","error":"tls: client didn't provide a certificate"}
```
The `etcd` service is accessible with the generated client certificate as the identity.
```sh
root@kubernetes:~/certificates# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert /root/certificates/client.crt --key /root/certificates/client.key put testclient data
OK

root@kubernetes:~/certificates# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false --cert /root/certificates/client.crt --key /root/certificates/client.key get testclient 
testclient
data
root@kubernetes:~/certificates# 
```
