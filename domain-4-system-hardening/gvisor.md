### Documentation:

https://kubernetes.io/docs/concepts/containers/runtime-class/

#### Step 1 - Download and install runsc and its shim from repository on each node:
```sh
(
  set -e
  ARCH=$(uname -m)
  URL=https://storage.googleapis.com/gvisor/releases/release/latest/${ARCH}
  wget ${URL}/runsc ${URL}/runsc.sha512 \
    ${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512
  sha512sum -c runsc.sha512 \
    -c containerd-shim-runsc-v1.sha512
  rm -f *.sha512
  chmod a+rx runsc containerd-shim-runsc-v1
  sudo mv runsc containerd-shim-runsc-v1 /usr/local/bin
)
```

#### Step 2 - Configure containerd on each worker node
```sh
cat <<EOF | sudo tee /etc/containerd/config.toml
version = 2
[plugins."io.containerd.runtime.v1.linux"]
  shim_debug = true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
EOF
systemctl restart containerd
```

####  Step 2 - Adding gVisor runtime class:
```sh
kubectl apply -f - <<EOF
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
EOF
```

#### Step 3 - Create pod with gvisor as runtime class
```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx-gvisor
spec:
  runtimeClassName: gvisor
  containers:
  - image: nginx
    name: nginx
EOF
```

#### Step 4 - Create one more pod for seeing the difference in dmesg output:
```sh
kubectl run nginx-default --image=nginx
```

#### Step 5 - Verify dmesg output
```sh
kubectl exec -it nginx-gvisor -- bash
dmesg
exit
```
<details>
<summary>Output</summary>

```sh
root@nginx-gvisor:/# dmesg
[    0.000000] Starting gVisor...
[    0.115907] Adversarially training Redcode AI...
[    0.240274] Accelerating teletypewriter to 9600 baud...
[    0.729529] Singleplexing /dev/ptmx...
[    1.137703] Verifying that no non-zero bytes made their way into /dev/zero...
```
</details>

```sh
kubectl exec -it nginx-default -- bash
```

<details>
<summary>Output</summary>

```
root@nginx-default:/# dmesg
dmesg: read kernel buffer failed: Operation not permitted
```
</details>