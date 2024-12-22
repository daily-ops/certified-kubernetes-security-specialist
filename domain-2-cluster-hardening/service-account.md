#### List service account in all namespaces
```sh
kubectl get serviceaccount --all-namespaces
```

#### Create a Test App Pod
```sh
kubectl run app-pod --image=nginx 
kubectl get pods
```
#### Verify Namespace associated with Pod
```sh
kubectl describe pod app-pod
```
#### Verify Mounted Token in Pod
```sh
kubectl exec -it app-pod -- bash

cd /var/run/secrets/kubernetes.io/serviceaccount/

ls

cat token
```
#### Connect to Kubernetes Cluster using Token
```sh
token=$(cat token)
echo $token

kubectl cluster-info (from outside of Pod)

curl -k -H "Authorization: Bearer $token" https://control-plane-url-here/api/v1
```

```
TOKEN=`kubectl exec -it nginx -- cat /var/run/secrets/kubernetes.io/serviceaccount/token`
~/cks-tf/apps$ curl -k -H "Authorization: Bearer $TOKEN" https://XXXXXXXXX:6443/api/v1 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{
  "kind": "APIResourceList",
1  "groupVersion": "v1",
0  "resources": [
0    {
       "name": "bindings",
1      "singularName": "binding",
0      "namespaced": true,
7      "kind": "Binding",
4      "verbs": [
2    0 10742    0     0   582k      0 --:--:-- --:--:-- --:--:--  582k
(23) Failed writing body
```
