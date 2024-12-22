#### Reference Websites:

https://jsonformatter.curiousconcept.com/#

#### Step 1 - Create Sample Audit Policy File:
```sh
nano /root/certificates/logging.yaml
```
```sh
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

#### Step 2 - Audit Configuration:
```sh
nano /etc/systemd/system/kube-apiserver.service
```
```sh
--audit-policy-file=/root/certificates/logging.yaml --audit-log-path=/var/log/api-audit.log --audit-log-maxage=7  --audit-log-maxbackup=5  --audit-log-maxsize=100 
```
```sh
systemctl daemon-reload
systemctl restart kube-apiserver
```
#### Step 3 Run Some Queries using Bob user

```sh
kubectl get secret --server=https://127.0.0.1:6443 --client-certificate /root/certificates/bob.crt --certificate-authority /root/certificates/ca.crt --client-key /root/certificates/bob.key
```
#### Step 4: Verification
```sh
cd /var/log
grep -i bob api-audit.log
```
Found audit log entries.
```
root@kubernetes:/var/lib/etcd# grep -i bob /var/log/api-audit.log
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"da5d1997-b4c5-44a4-b015-5eacc25d6432","stage":"RequestReceived","requestURI":"/api/v1/namespaces/default/secrets?limit=500","verb":"list","user":{"username":"bob","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.24.2 (linux/amd64) kubernetes/f66044f","objectRef":{"resource":"secrets","namespace":"default","apiVersion":"v1"},"requestReceivedTimestamp":"2024-12-13T05:28:39.999903Z","stageTimestamp":"2024-12-13T05:28:39.999903Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"da5d1997-b4c5-44a4-b015-5eacc25d6432","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/secrets?limit=500","verb":"list","user":{"username":"bob","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.24.2 (linux/amd64) kubernetes/f66044f","objectRef":{"resource":"secrets","namespace":"default","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-12-13T05:28:39.999903Z","stageTimestamp":"2024-12-13T05:28:40.005508Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
root@kubernetes:/var/lib/etcd# 
```
Showing the file name when `maxbackup` is in place (which can be omitted).
```
ansible@controlplane:~$ ls -al /var/log/kubernetes/audit/
total 3568
drwxr-xr-x 2 root root    4096 Dec 19 01:40 .
drwxr-xr-x 3 root root    4096 Dec 19 00:23 ..
-rw------- 1 root root 2409234 Dec 19 01:39 kube-apiserver-audit-2024-12-19T01-40-10.502.log
-rw------- 1 root root 1048243 Dec 19 01:40 kube-apiserver-audit-2024-12-19T01-40-29.272.log
-rw------- 1 root root  182712 Dec 19 01:40 kube-apiserver-audit.log
```
Chances are the oldest files will be rolled off and disappeared.
```
ansible@controlplane:~$ ls -al /var/log/kubernetes/audit/
total 3124
drwxr-xr-x 2 root root    4096 Dec 19 01:46 .
drwxr-xr-x 3 root root    4096 Dec 19 00:23 ..
-rw------- 1 root root 1048243 Dec 19 01:40 kube-apiserver-audit-2024-12-19T01-40-29.272.log
-rw------- 1 root root 1048355 Dec 19 01:43 kube-apiserver-audit-2024-12-19T01-43-38.215.log
-rw------- 1 root root 1048543 Dec 19 01:46 kube-apiserver-audit-2024-12-19T01-46-00.947.log
-rw------- 1 root root   37711 Dec 19 01:46 kube-apiserver-audit.log
```
