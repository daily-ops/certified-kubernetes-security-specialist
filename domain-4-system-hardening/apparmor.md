#### Check status of apparmor:
```sh
systemctl status apparmor
```
#### Sample Script:
```sh
mkdir /tmp/apparmor
cd /tmp/apparmor
```
```sh
nano testscript.sh
```
```sh
#!/bin/bash
touch /tmp/file.txt
echo "New File created"

rm -f /tmp/file.txt
echo "New file removed"
```
```sh
chmod +x testscript.sh
```
#### Install Apparmor Utils:
```sh
apt install apparmor-utils
```
#### Generate a new profile:
```sh
aa-genprof ./testscript.sh
```
```sh
./testscript.sh (from new tab)
```
#### Verify the new profile:
```sh
cat /etc/apparmor.d/tmp.apparmor.testscript.sh
aa-status
```
<details>
<summary>Output</summary>
```
ansible@nuc01:/tmp/apparmor$ sudo aa-status | grep test
   /tmp/apparmor/testscript.sh
   /tmp/apparmor/testscript.sh//null-/usr/bin/rm
   /tmp/apparmor/testscript.sh//null-/usr/bin/touch
```
</details>

#### Disable a profile:
```sh
ln -s /etc/apparmor.d/tmp.apparmor.testscript.sh /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/tmp.apparmor.testscript.sh
```
