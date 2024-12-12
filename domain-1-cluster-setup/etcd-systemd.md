

#### Step 1: Create Data Directory for ETCD

```sh
mkdir /var/lib/etcd
chmod 700 /var/lib/etcd
```

#### Step 2: Created Systemd file for ETCD
```sh
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --cert-file=/root/certificates/etcd.crt \\
  --key-file=/root/certificates/etcd.key \\
  --trusted-ca-file=/root/certificates/ca.crt \\
  --client-cert-auth \\
  --listen-client-urls https://127.0.0.1:2379 \\
  --advertise-client-urls https://127.0.0.1:2379 \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
#### Step 3: Start ETCD
```sh
systemctl start etcd
```
#### Step 4: Verify the status
```sh
systemctl status etcd
```
The output.
```
root@kubernetes:~# systemctl start etcd
root@kubernetes:~# systemctl status etcd
● etcd.service - etcd
     Loaded: loaded (/etc/systemd/system/etcd.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-12-12 22:06:03 UTC; 8s ago
       Docs: https://github.com/coreos
   Main PID: 4501 (etcd)
      Tasks: 7 (limit: 2309)
     Memory: 6.7M
        CPU: 129ms
     CGroup: /system.slice/etcd.service
             └─4501 /usr/local/bin/etcd --cert-file=/root/certificates/etcd.crt --key-file=/root/certificates/etcd.k>

Dec 12 22:06:04 kubernetes etcd[4501]: {"level":"info","ts":"2024-12-12T22:06:04.397Z","logger":"raft","caller":"etc>
Dec 12 22:06:04 kubernetes etcd[4501]: {"level":"info","ts":"2024-12-12T22:06:04.399Z","caller":"etcdserver/server.g>
```
#### Step 5: Check ETCD Logs
```sh
journalctl -u etcd

systemctl restart systemd-journald.service
```