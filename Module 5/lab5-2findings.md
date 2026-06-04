# Lab 5.2 Findings: Change Management Simulation

## Lab Summary
This lab practised a Normal Change process on a live lab VM by planning, executing, verifying, and rolling back a TLS change for Nginx.

- Change ID: `CHANGE-001`
- Change type: Normal Change
- Target VM: `34.239.137.83`
- Service: Nginx HTTPS lab site
- Planned config file: `/etc/nginx/sites-available/lab-tls`
- Change objective: Upgrade the lab HTTPS service from TLS 1.2/TLS 1.3 support to TLS 1.3 only
- Rollback objective: Restore the backed-up Nginx configuration after a deliberately introduced bad directive

## Submission Links
- Change plan: [CHANGE-001-TLS-Upgrade.md](https://github.com/ayushpd1/cogs-lab-instasafe/blob/main/Module%205/changes/CHANGE-001-TLS-Upgrade.md)
- Execution evidence: [CHANGE-001-execution-evidence.md](https://github.com/ayushpd1/cogs-lab-instasafe/blob/main/Module%205/changes/CHANGE-001-execution-evidence.md)
- Rollback evidence: [CHANGE-001-rollback-evidence.md](https://github.com/ayushpd1/cogs-lab-instasafe/blob/main/Module%205/changes/CHANGE-001-rollback-evidence.md)
- Master findings file: [lab5-2findings.md](https://github.com/ayushpd1/cogs-lab-instasafe/blob/main/Module%205/lab5-2findings.md)

## Change Plan
The change plan was written before execution and committed first.

- Change ID: `CHANGE-001`
- Type: Normal Change
- Description: Update Nginx TLS configuration to prefer TLS 1.3 by disabling TLS 1.2 on the lab HTTPS site
- Justification: TLS 1.3 is faster and more secure than TLS 1.2 and aligns the lab with current best practice
- Rollback trigger: TLS verification failure, Nginx validation failure, or service becoming unreachable
- Rollback time: 2 minutes
- Maintenance window: Immediate, lab environment only

Execution steps from the plan:
1. Backup current Nginx site configuration.
2. Update the `ssl_protocols` line in the Nginx site config.
3. Test config with `sudo nginx -t`.
4. Reload Nginx.
5. Verify TLS 1.3 is working.
6. Verify TLS 1.2 is blocked.

Rollback steps from the plan:
1. Restore the backup config with `sudo cp /etc/nginx/sites-available/lab-tls.backup /etc/nginx/sites-available/lab-tls`.
2. Validate and reload Nginx with `sudo nginx -t && sudo systemctl reload nginx`.
3. Confirm service recovery with `curl -k https://localhost`.

## Git Commit Trail
The required Git trail shows the plan was committed before execution, followed by execution evidence and rollback evidence.

```text
2e5bbcc CHANGE-001: Rollback executed — BAD CONFIG DETECTED — ROLLED BACK
d205157 CHANGE-001: TLS upgrade executed — COMPLETED
ce8303e CHANGE-001: Change plan — TLS upgrade — PENDING APPROVAL
e5ee946 Add files via upload
abe76ad Create lab5-1findings.md
```

## Pre-flight Inspection
The expected `/etc/nginx/sites-available/lab-tls` file was not present at the start. Nginx was healthy but only listening on port 80, so the lab TLS site was created safely before performing the planned TLS change.

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

## Execution Evidence
The `lab-tls` site was created and backed up before applying the TLS change.

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

The first validation failed before reload because the referenced snakeoil certificate was not present. This was safely caught by `nginx -t`, so the running service was not reloaded into a bad state.

To complete the lab, a self-signed lab certificate was generated and the Nginx site was updated to use it.

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

Clean TLS verification confirmed TLS 1.3 success and TLS 1.2 rejection.

```text
## TLS 1.3 brief
CONNECTION ESTABLISHED
Protocol version: TLSv1.3
Ciphersuite: TLS_AES_256_GCM_SHA384
Verification error: self-signed certificate
## TLS 1.2 brief
409707E5457F0000:error:0A00042E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:../ssl/record/rec_layer_s3.c:918:SSL alert number 70
```

## Rollback Evidence
A bad config was deliberately introduced by changing `ssl_protocols` to `ssl_protocolz`. This caused `nginx -t` to fail, which triggered rollback from the backup.

```text
## Rollback simulation start
2026-06-04T09:55:03+00:00
## Introduce bad config
10:    ssl_protocolz TLSv1.3;
## nginx -t expected failure
2026/06/04 09:55:03 [emerg] 8472#8472: unknown directive "ssl_protocolz" in /etc/nginx/sites-enabled/lab-tls:10
nginx: configuration file /etc/nginx/nginx.conf test failed
## Rollback triggered: restore from backup
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
Rollback duration seconds: 0
## Restored config TLS lines
10:    ssl_protocols TLSv1.2 TLSv1.3;
## Service restored
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
## Rollback simulation end
2026-06-04T09:55:03+00:00
```

## Final Result
The Normal Change process was completed successfully.

- The change plan was written and committed before execution.
- Nginx TLS configuration was updated and validated with `nginx -t`.
- TLS 1.3 was confirmed working.
- TLS 1.2 was confirmed blocked by protocol-version alert.
- Rollback was triggered using a deliberately bad directive.
- Backup restore completed successfully in under 2 minutes.
- Service recovery was confirmed using `curl -k https://localhost`.
- Git history contains the required Plan -> Execute -> Rollback commit sequence.
