#### Create a Sample Profile:
```sh
apparmor_parser -q <<EOF
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
EOF
```
#### Verify the status of the profile:
```sh
$ sudo aa-status | grep example -A 5 -B 5
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /{,usr/}sbin/dhclient
   cri-containerd.apparmor.d
   k8s-apparmor-example-deny-write
   lsb_release
   man_filter
   man_groff
   nvidia_modprobe
   nvidia_modprobe//kmod
```

#### Sample YAML File based on Host PID:
```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-apparmor-example-deny-write
  containers:
  - name: hello
    image: busybox
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
EOF
```

#### Verification Stage:
```sh
kubectl exec -it hello-apparmor -- sh
touch /tmp/file.txt
```
<details>
<summary>Output</summary>

```
$ kubectl exec -it hello-apparmor -- sh
/ # touch /tmp/file.txt
touch: /tmp/file.txt: Permission denied
/ # 
```
</details>