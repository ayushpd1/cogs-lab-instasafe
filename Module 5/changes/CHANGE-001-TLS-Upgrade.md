# Change ID: CHANGE-001

## Type
Normal Change

## Description
Update Nginx TLS configuration to prefer TLS 1.3 by disabling TLS 1.2 on the lab HTTPS site.

## Justification
TLS 1.3 is faster and more secure than TLS 1.2 and aligns the lab environment with current best practice.

## Target System
- VM: `34.239.137.83`
- Service: Nginx HTTPS lab site
- Config file: `/etc/nginx/sites-available/lab-tls`

## Execution Steps
1. Backup current Nginx site configuration.
2. Update the `ssl_protocols` line in the Nginx site config.
3. Test config with `sudo nginx -t`.
4. Reload Nginx.
5. Verify TLS 1.3 is working.
6. Verify TLS 1.2 is blocked.

## Rollback Trigger
Rollback will be triggered if TLS verification fails, Nginx configuration validation fails, or the site becomes unreachable.

## Rollback Steps
1. Restore the backup config:
   `sudo cp /etc/nginx/sites-available/lab-tls.backup /etc/nginx/sites-available/lab-tls`
2. Validate and reload Nginx:
   `sudo nginx -t && sudo systemctl reload nginx`
3. Confirm service recovery:
   `curl -k https://localhost`

## Rollback Time
2 minutes

## Maintenance Window
Immediate. Lab environment only, no customer impact.

## Approval Status
Pending approval before execution.
