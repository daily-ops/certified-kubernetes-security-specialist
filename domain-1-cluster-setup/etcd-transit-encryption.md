
#### Step 1 - Install utilities
```sh
apt-get -y install tcpdump net-tools
```

#### Step 2 - Capture Plain Text ETCD Traffic

First tab:
```sh
cd /tmp
etcd
```

This will start up etcd listening on localhost port 2379. 
```
vagrant@kubernetes:~$ etcd
{"level":"info","ts":"2024-12-12T09:31:57.284Z","caller":"etcdmain/etcd.go:73","msg":"Running: ","args":["etcd"]}
{"level":"warn","ts":"2024-12-12T09:31:57.284Z","caller":"etcdmain/etcd.go:105","msg":"'data-dir' was empty; using default","data-dir":"default.etcd"}
{"level":"info","ts":"2024-12-12T09:31:57.285Z","caller":"etcdmain/etcd.go:116","msg":"server has been already initialized","data-dir":"default.etcd","dir-type":"member"}
{"level":"info","ts":"2024-12-12T09:31:57.285Z","caller":"embed/etcd.go:131","msg":"configuring peer listeners","listen-peer-urls":["http://localhost:2380"]}
{"level":"info","ts":"2024-12-12T09:31:57.286Z","caller":"embed/etcd.go:139","msg":"configuring client listeners","listen-client-urls":["http://localhost:2379"]}
```

Second tab:
```sh
tcpdump -i lo -X  port 2379
```
The option `-i` is for interface name.
The option `-X` is to print data of each packet in hex and ASCII.
```
vagrant@kubernetes:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:95:e1:8a:38:bd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84615sec preferred_lft 84615sec
    inet6 fe80::95:e1ff:fe8a:38bd/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:02:6b:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.180/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe02:6b10/64 scope link 
       valid_lft forever preferred_lft forever
```

Third tab:
```sh
etcdctl put test123 aaaaaaaaaaaaaaaa
```
This will result in plain text being displayed in the tcpdump output stream.
```
09:37:47.147641 IP localhost.40326 > localhost.2379: Flags [P.], seq 156:197, ack 31, win 512, options [nop,nop,TS val 1304386512 ecr 1304386511], length 41
        0x0000:  4500 005d 6b5f 4000 4006 d139 7f00 0001  E..]k_@.@..9....
        0x0010:  7f00 0001 9d86 094b 4bca 2ecb cd4e df80  .......KK....N..
        0x0020:  8018 0200 fe51 0000 0101 080a 4dbf 5bd0  .....Q......M.[.
        0x0030:  4dbf 5bcf 0000 2000 0100 0000 0100 0000  M.[.............
        0x0040:  001b 0a07 7465 7374 3132 3312 1061 6161  ....test123..aaa
        0x0050:  6161 6161 6161 6161 6161 6161 61         aaaaaaaaaaaaa
09:37:47.147676 IP localhost.2379 > localhost.40326: Flags [.], ack 197, win 512, options [nop,nop,TS val 1304386512 ecr 1304386512], length 0
        0x0000:  4500 0034 1b6e 4000 4006 2154 7f00 0001  E..4.n@.@.!T....
        0x0010:  7f00 0001 094b 9d86 cd4e df80 4bca 2ef4  .....K...N..K...
        0x0020:  8010 0200 fe28 0000 0101 080a 4dbf 5bd0  .....(......M.[.
        0x0030:  4dbf 5bd0                                M.[.
```
#### Step 2 - Creating the ETCD Certificate Key:
```sh
cd /root/certificates
openssl genrsa -out etcd.key 2048
```
NOTES
- -out redirect output to file instead of terminal.
- 2048 = Size of key in bits
#### Step 3 - Find the IP of your server

```sh
ip a
```
List of addresses.
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:95:e1:8a:38:bd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84615sec preferred_lft 84615sec
    inet6 fe80::95:e1ff:fe8a:38bd/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:02:6b:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.180/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe02:6b10/64 scope link 
       valid_lft forever preferred_lft forever
```

#### Step 4 - Creating Configuration for ETCD CSR:
```sh
cat > etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = [IP-1]
IP.2 = 127.0.0.1
EOF
```
NOTES:
- [IP-1] = 192.168.0.180 as per `ip a` output above.

#### Step 4 - Creating CSR:
```sh
openssl req -new -key etcd.key -subj "/CN=etcd" -out etcd.csr -config etcd.cnf
```
The generated CSR.
```
root@kubernetes:~# cat etcd.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIIClDCCAXwCAQAwDzENMAsGA1UEAwwEZXRjZDCCASIwDQYJKoZIhvcNAQEBBQAD
ggEPADCCAQoCggEBAKMiRwmp1takzYwq0WYsgmnTe+UYLkfmr4wc6WXK6m4Yc3DL
9KNlzKc8hcdsE6HOVHuWM+yM5iOgcPmoeBCZ98iNJhOy3h5k3IPsLxw9IBdYspAB
M+VmhK31caVacFkqzuyCN76mS2g6RU+RlJ3t0buUUmjIludrYXJx/BSYN17nQJPm
yXtMbfqHdKclEdyctOSmLbCs/3eSaRibF7s6Iz/JX8eOzvyuK8TlDiTPdUjzQejq
WzGezEpBgTSKC3FPXcs4Q9M6s218HTKT2r3t/juM0UWq6DUoBLlJcotWRvJIOOHr
5Q66uHR2cVJeR7p59oH3VbSB8mhFP/D37tPO3v0CAwEAAaBAMD4GCSqGSIb3DQEJ
DjExMC8wCQYDVR0TBAIwADALBgNVHQ8EBAMCBeAwFQYDVR0RBA4wDIcEwKgAtIcE
fwAAATANBgkqhkiG9w0BAQsFAAOCAQEAhfJAhP5YYuhEyYjz8GZy1lX3Pv1Q/AQP
1vIR3WtfQVJLljT3q/CdkkIIIvbv9R0CIExVofBn5RC2fVqnIhgFV1xPRO59o+BP
wwdYVCzhSPZ8l4hHgT6/ZuHMSjkSDA9Q3ln6CeE1/9l/1wtG4aklJ0hNXo8qRhpI
mPBpdT/t4/qcGKhSIZSFWvJMsEvKRhOKxKybWkzT3HAWRrkNl0gUgMlFS/GFenea
V0nTtJ63otZPDzctGJqPh3sFTeXNersxlOTr+gALzyHdqbQ89w5t7brD0JzWLrAE
9a26bGEv4S8cl1VTRGNN+IuayH51uB9mUNjiPiD3NQb3UU5JL3jQoA==
-----END CERTIFICATE REQUEST-----
```
#### Step 4 - Signing CSR with Certificate Authority Certificate:
```sh
openssl x509 -req -in etcd.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd.crt -extensions v3_req -extfile etcd.cnf -days 60
```
The generated certificate.
```
root@kubernetes:~# cat etcd.crt 
-----BEGIN CERTIFICATE-----
MIIDRjCCAi6gAwIBAgIUD+YxaMHFL6xNT7jr8seIBHEHvc0wDQYJKoZIhvcNAQEL
BQAwGDEWMBQGA1UEAwwNS1VCRVJORVRFUy1DQTAeFw0yNDEyMTIxMDA2MzNaFw0y
NTAxMTExMDA2MzNaMA8xDTALBgNVBAMMBGV0Y2QwggEiMA0GCSqGSIb3DQEBAQUA
A4IBDwAwggEKAoIBAQCjIkcJqdbWpM2MKtFmLIJp03vlGC5H5q+MHOllyupuGHNw
y/SjZcynPIXHbBOhzlR7ljPsjOYjoHD5qHgQmffIjSYTst4eZNyD7C8cPSAXWLKQ
ATPlZoSt9XGlWnBZKs7sgje+pktoOkVPkZSd7dG7lFJoyJbna2FycfwUmDde50CT
5sl7TG36h3SnJRHcnLTkpi2wrP93kmkYmxe7OiM/yV/Hjs78rivE5Q4kz3VI80Ho
6lsxnsxKQYE0igtxT13LOEPTOrNtfB0yk9q97f47jNFFqug1KAS5SXKLVkbySDjh
6+UOurh0dnFSXke6efaB91W0gfJoRT/w9+7Tzt79AgMBAAGjgZAwgY0wCQYDVR0T
BAIwADALBgNVHQ8EBAMCBeAwFQYDVR0RBA4wDIcEwKgAtIcEfwAAATAdBgNVHQ4E
FgQUySkReuj0uoPEIYbAWfPKxj3fFhowPQYDVR0jBDYwNKEcpBowGDEWMBQGA1UE
AwwNS1VCRVJORVRFUy1DQYIUP6dmQKbBSbLFUM4X8rp9fu1aJDQwDQYJKoZIhvcN
AQELBQADggEBAIUf25XCjUeHgDIZ/XDTxQJ1KWr/xneSGjp1XWf5qCgmdUqSN+vd
iYK5BFPPRVUJ5mRfdjMPV4C2lpSjqU+Sg2YZn0XbNblGJGDvv8yWf1uBU8tZt5Cc
ckecrFogO6tgePTRvRRVBWo17YZU1xWdFDMpuKLeeT899dKR0Tl/Ro29/Jj3p1UF
oQp0wxHQJC3GyF2IBo2QJ3FiUBTsNkTaAKwxKmbrmZQnqi8VU8KI1c7oxl8dmlBs
56MAQD/DFTlDE3GTEOiA0YjwH8K+j0SH8rUN3XbmI6rWWXIN+blm6xwc8EQSbcH0
4JDl9QVIUyw6anlHEZeVnFTt8sqcciGT06k=
-----END CERTIFICATE-----
```

#### Step 5 - Start ETCD Server with Certificates:
```sh
etcd --cert-file=/root/certificates/etcd.crt --key-file=/root/certificates/etcd.key --advertise-client-urls=https://127.0.0.1:2379 --listen-client-urls=https://127.0.0.1:2379
```

#### Step 6 - Verify
```sh
etcdctl put test123 aaaaaaaaaaaaaaaa
```

#### Run TCPDUMP to capture Traffic:
```sh
tcpdump -i lo -X  port 2379
```

#### Step 6 - 
```sh
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false test123 aaaaaaaaaaaaaaaa

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false get test123
```
No plain text visible in the output stream.
