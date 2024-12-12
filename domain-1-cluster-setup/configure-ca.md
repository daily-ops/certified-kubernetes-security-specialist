#### Step 1 - Creating a private key for Certificate Authority:
```sh
mkdir /root/certificates
cd /root/certificates
```
```sh
openssl genrsa -out ca.key 2048
```
NOTES
- -out redirect output to file instead of terminal.
- 2048 = Size of key in bits

#### Step 2 -  Creating CSR:
```sh
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
The generated CSR.
```
root@kubernetes:~# cat ca.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICXTCCAUUCAQAwGDEWMBQGA1UEAwwNS1VCRVJORVRFUy1DQTCCASIwDQYJKoZI
hvcNAQEBBQADggEPADCCAQoCggEBAIqwdtsZEt8bZ5d9kjifEHikt+ISwVgJIEb+
4pv4YhM9gBbi349LWYuF2WXO5Wf2ilLDfWRyMMsRqiuuVST77AflV7pvxgWZH5PQ
T5pWxvssAXTeCHFEes2e9FLCsNMeL/8rFjTc5BwWiGMLoIwkOu3okEtOBkOFR/YN
mfLwsDzm2X+8YVRwGexAhtZRXC/70m3U0x8nWjky/NSyksR7bFHFeLFfJKLI/fGg
DHozgcbe0D7tGgYyqQ7UBJZI96p6WoPna9gRhvYrk1RT5lb+6O9qaSKB3rnkN1VT
GdXIUnmDNerK6ptZaZGZ3XKtjLz+Q5Ft6rg9gu58xOTgjBdGCIUCAwEAAaAAMA0G
CSqGSIb3DQEBCwUAA4IBAQAp96mEPs9qbdi0/xsF4703D44xUAcr8CXfUhsGVOVH
3RW/kxp5AKmIlcHpx3Mnn3Iwqa7oqOQNsjuQc75QLNqS61JM7yufeUPYs2TD7vS0
uybRSbUT+Hv2Z8+Y/gXc0Nllhrxic/OzIlD28w7uUTzf0AS7bK24KnYzF56Ob1Ly
C0OMQGzBmqr9hv9dXWYnYUPpPUbLyduN31l7YbGPSgQy3YE7A3o/tUUuxLAyz6aZ
/XQ6ygKKG44lJKNER3i3jT0XLfWDWCuothu2hXte1wInDjFSETb3VojPUKXRdscH
WaTkzR6QpC8kL3OhB65HGQiiIDEyDPTaM7M4uclFwUEG
-----END CERTIFICATE REQUEST-----
```
#### Step 3 - Self-Sign the CSR:
```sh
openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 90
```
The generated certificate.
```
root@kubernetes:~# cat ca.crt
-----BEGIN CERTIFICATE-----
MIICtzCCAZ8CFD+nZkCmwUmyxVDOF/K6fX7tWiQ0MA0GCSqGSIb3DQEBCwUAMBgx
FjAUBgNVBAMMDUtVQkVSTkVURVMtQ0EwHhcNMjQxMjEyMTAwMTQ0WhcNMjUwMzEy
MTAwMTQ0WjAYMRYwFAYDVQQDDA1LVUJFUk5FVEVTLUNBMIIBIjANBgkqhkiG9w0B
AQEFAAOCAQ8AMIIBCgKCAQEAirB22xkS3xtnl32SOJ8QeKS34hLBWAkgRv7im/hi
Ez2AFuLfj0tZi4XZZc7lZ/aKUsN9ZHIwyxGqK65VJPvsB+VXum/GBZkfk9BPmlbG
+ywBdN4IcUR6zZ70UsKw0x4v/ysWNNzkHBaIYwugjCQ67eiQS04GQ4VH9g2Z8vCw
PObZf7xhVHAZ7ECG1lFcL/vSbdTTHydaOTL81LKSxHtsUcV4sV8kosj98aAMejOB
xt7QPu0aBjKpDtQElkj3qnpag+dr2BGG9iuTVFPmVv7o72ppIoHeueQ3VVMZ1chS
eYM16srqm1lpkZndcq2MvP5DkW3quD2C7nzE5OCMF0YIhQIDAQABMA0GCSqGSIb3
DQEBCwUAA4IBAQBRWg8fndhLprD9PVut7FmbhyzU4obs8R4vI+EJvpIfyQ/ic7a0
hzr/nlwMD91H9ucBE91Wy8rDYTcMn4eF1buIdAzCEGHkMFx8Vs0pKSlK5gYUUFQZ
teJPdq+NeGh1FGGCbmdN6R27NPz0RyOQZIF38rQlei68UM8NJAFPymsKa1PH+kWe
5tDRj8n7ZBdpM5Fmg5FHT7yLIT2ZlZxVOqTRUZ+1NfjIu6/nhMUJ9i6haQqWxwDD
NofoO7HZ6TS4qKiNZWdPZBjb0jg//UPMB4R5GtWyRxRw6vzel4Ew/Wvl4Ig8jxNR
o8/pKpX1hH4jcBmJjd6nRjy/5vrPSRWVzCQU
-----END CERTIFICATE-----
```
#### Step 4 - Remove the CSR
```sh
rm -f ca.csr
```
