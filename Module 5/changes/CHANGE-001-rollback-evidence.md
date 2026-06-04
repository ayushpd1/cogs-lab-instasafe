# CHANGE-001 Rollback Evidence

## Rollback Trigger
A bad directive was deliberately introduced by changing `ssl_protocols` to `ssl_protocolz`.
This caused `sudo nginx -t` to fail, triggering rollback from `/etc/nginx/sites-available/lab-tls.backup`.

## Terminal Evidence
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

## Result
Rollback completed successfully and restored the HTTPS service from backup.
