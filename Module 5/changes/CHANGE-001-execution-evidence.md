# CHANGE-001 Execution Evidence

## Context
The committed change plan expected `/etc/nginx/sites-available/lab-tls`.
During pre-flight inspection, that file was missing and Nginx was only listening on port 80.
To complete the lab safely, the `lab-tls` HTTPS site was created, backed up, updated to TLS 1.3 only, tested, and reloaded after validation.

## Pre-flight Inspection
```text
ubuntu
ip-172-31-41-183
lab-tls-missing
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

```text
## ports
LISTEN 0      511          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=7406,fd=5),("nginx",pid=7405,fd=5),("nginx",pid=6990,fd=5))
LISTEN 0      511             [::]:80            [::]:*    users:(("nginx",pid=7406,fd=6),("nginx",pid=7405,fd=6),("nginx",pid=6990,fd=6))
```

## Execution Attempt and Validation Failure
The first validation failed before reload because the expected snakeoil certificate was not present.
This did not affect the running service because `nginx -t` failed before `systemctl reload nginx`.

```text
## Pre-flight
2026-06-04T09:52:59+00:00
## Create baseline lab-tls site with TLSv1.2 TLSv1.3
## Step 1 Backup
total 20
drwxr-xr-x 2 root root 4096 Jun  4 09:52 .
drwxr-xr-x 8 root root 4096 Jun  4 09:44 ..
-rw-r--r-- 1 root root 2412 Mar 27 14:26 default
-rw-r--r-- 1 root root  402 Jun  4 09:52 lab-tls
-rw-r--r-- 1 root root  402 Jun  4 09:52 lab-tls.backup
## Step 2 Modify TLS config
10:    ssl_protocols TLSv1.3;
11:    ssl_prefer_server_ciphers off;
## Step 3 Test config
2026/06/04 09:52:59 [emerg] 8156#8156: cannot load certificate "/etc/ssl/certs/ssl-cert-snakeoil.pem": BIO_new_file() failed (SSL: error:80000002:system library::No such file or directory:calling fopen(/etc/ssl/certs/ssl-cert-snakeoil.pem, r) error:10000080:BIO routines::no such file)
nginx: configuration file /etc/nginx/nginx.conf test failed
```

## Completed Execution
A self-signed lab certificate was generated at `/etc/nginx/ssl/lab-tls.crt` and `/etc/nginx/ssl/lab-tls.key`, then the config was validated and reloaded.

```text
## Remediation: create lab self-signed cert
## Current lab-tls config
2:    listen 443 ssl default_server;
3:    listen [::]:443 ssl default_server;
8:    ssl_certificate /etc/nginx/ssl/lab-tls.crt;
9:    ssl_certificate_key /etc/nginx/ssl/lab-tls.key;
10:    ssl_protocols TLSv1.3;
11:    ssl_prefer_server_ciphers off;
## nginx -t after cert remediation
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
## reload
## TLS 1.3 verification
Protocol: TLSv1.3
    Protocol  : TLSv1.3
    Protocol  : TLSv1.3
## TLS 1.2 blocked verification
40578A4913770000:error:0A00042E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:../ssl/record/rec_layer_s3.c:918:SSL alert number 70
Protocol: TLSv1.2
    Protocol  : TLSv1.2
## service check
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
Date: Thu, 04 Jun 2026 09:53:26 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Thu, 04 Jun 2026 08:08:07 GMT
Connection: keep-alive
ETag: "6a213267-267"
Accept-Ranges: bytes
```

## Clean TLS Verification
```text
## TLS 1.3 brief
CONNECTION ESTABLISHED
Protocol version: TLSv1.3
Ciphersuite: TLS_AES_256_GCM_SHA384
Verification error: self-signed certificate
## TLS 1.2 brief
409707E5457F0000:error:0A00042E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:../ssl/record/rec_layer_s3.c:918:SSL alert number 70
```
