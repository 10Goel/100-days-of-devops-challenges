# Day 15 — Commands Reference

This file documents the important commands used during the Day 15 **Nginx HTTPS/SSL configuration challenge** and explains **what each command does, why it is used, and where it fits into the deployment workflow**.

---

# 1. SSH Into App Server 3

```bash
ssh banner@stapp03
```

## Purpose

Connects from the jump host to App Server 3.

General syntax:

```bash
ssh USER@HOST
```

For this challenge:

```text
USER = banner
HOST = stapp03
```

---

# 2. Verify the Hostname

```bash
hostname
```

## Purpose

Confirms that the current shell is connected to the intended server.

Expected:

```text
stapp03
```

---

# 3. Install Nginx

```bash
sudo yum install nginx -y
```

## Purpose

Installs the Nginx web server on RHEL/Rocky/CentOS-style systems.

In this challenge, the package was already installed, so the package manager reported:

```text
Nothing to do.
Complete!
```

---

# 4. Verify Nginx Version

```bash
nginx -v
```

## Purpose

Displays the installed Nginx version.

Observed during the challenge:

```text
nginx version: nginx/1.20.1
```

---

# 5. Verify the SSL Files

```bash
ls -l /tmp/nautilus.crt /tmp/nautilus.key
```

## Purpose

Confirms that the certificate and private key supplied by the challenge exist.

Expected files:

```text
/tmp/nautilus.crt
/tmp/nautilus.key
```

---

# 6. Inspect the SSL Certificate

```bash
sudo openssl x509 -in /tmp/nautilus.crt -noout -subject -issuer -dates
```

## Purpose

Uses OpenSSL to inspect certificate metadata.

Useful fields include:

```text
subject
issuer
notBefore
notAfter
```

This is useful for checking certificate identity and validity dates before deployment.

---

# 7. Create the Nginx SSL Directory

```bash
sudo mkdir -p /etc/nginx/ssl
```

## Purpose

Creates a dedicated directory for the Nginx TLS certificate and private key.

The `-p` option creates the directory if necessary without failing when it already exists.

---

# 8. Move the SSL Certificate

```bash
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/nautilus.crt
```

## Purpose

Moves the provided certificate from the temporary directory into the Nginx SSL directory.

---

# 9. Move the SSL Private Key

```bash
sudo mv /tmp/nautilus.key /etc/nginx/ssl/nautilus.key
```

## Purpose

Moves the provided private key into the dedicated Nginx SSL directory.

---

# 10. Verify the SSL Directory

```bash
sudo ls -l /etc/nginx/ssl/
```

## Purpose

Confirms that the certificate and private key were moved successfully.

Expected files:

```text
nautilus.crt
nautilus.key
```

---

# 11. Inspect the Main Nginx Configuration

```bash
sudo cat /etc/nginx/nginx.conf
```

## Purpose

Displays the main Nginx configuration.

An important directive already present in the configuration was:

```nginx
include /etc/nginx/conf.d/*.conf;
```

This allows additional server configurations to be stored under:

```text
/etc/nginx/conf.d/
```

---

# 12. Create the HTTPS Configuration

```bash
sudo vi /etc/nginx/conf.d/ssl.conf
```

## Purpose

Creates a dedicated Nginx configuration file for HTTPS.

Configuration used:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

# 13. Create the Required Web Page

```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

## Purpose

Creates the required `index.html` file under the Nginx document root.

Required content:

```text
Welcome!
```

---

# 14. Verify the Web Page

```bash
cat /usr/share/nginx/html/index.html
```

## Purpose

Confirms that the required page content was written correctly.

Expected:

```text
Welcome!
```

---

# 15. Validate Nginx Configuration

```bash
sudo nginx -t
```

## Purpose

Tests Nginx configuration syntax and verifies that the configuration can be loaded.

Successful output:

```text
syntax is ok
test is successful
```

---

# 16. Troubleshooting a Configuration Error

The first configuration test failed because the server block contained:

```text
server_name_
```

instead of:

```nginx
server_name _;
```

Nginx reported:

```text
unknown directive "server_name_"
```

After correcting the directive, run:

```bash
sudo nginx -t
```

again.

The corrected configuration returned:

```text
syntax is ok
test is successful
```

---

# 17. Start Nginx

```bash
sudo systemctl start nginx
```

## Purpose

Starts the Nginx service immediately.

---

# 18. Check Nginx Status

```bash
sudo systemctl status nginx --no-pager
```

## Purpose

Displays the current Nginx service state without opening a pager.

Desired state:

```text
Active: active (running)
```

---

# 19. Check Whether Nginx Is Active

```bash
sudo systemctl is-active nginx
```

## Purpose

Provides a concise service health check.

Expected:

```text
active
```

---

# 20. Check Whether Nginx Is Enabled

```bash
sudo systemctl is-enabled nginx
```

## Purpose

Checks whether Nginx is configured to start automatically during system boot.

Expected:

```text
enabled
```

---

# 21. Enable Nginx at Boot

```bash
sudo systemctl enable nginx
```

## Purpose

Configures Nginx to start automatically when the system boots.

This is useful for persistent application availability.

---

# 22. Restart Nginx

```bash
sudo systemctl restart nginx
```

## Purpose

Stops and starts Nginx again.

Use this after making a valid configuration change when a restart is appropriate.

Always validate first:

```bash
sudo nginx -t
```

---

# 23. Test HTTPS Locally

```bash
curl -Ik https://localhost/
```

## Purpose

Tests the HTTPS endpoint directly from `stapp03`.

Important options:

```text
-I
↓
Request headers only

-k
↓
Ignore certificate trust verification
```

Expected response:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
```

---

# 24. Test HTTPS From the Jump Host

First return to the jump host:

```bash
exit
```

Then run:

```bash
curl -Ik https://stapp03/
```

## Purpose

Performs the final end-to-end HTTPS test using the App Server 3 hostname.

Expected:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
```

This is the final challenge validation.

---

# 25. Test the Actual Page Content

If required to inspect the response body rather than only headers:

```bash
curl -k https://stapp03/
```

Expected page content:

```text
Welcome!
```

---

# 26. Check Listening Port 443

```bash
sudo ss -lntp | grep :443
```

## Purpose

Checks whether a process is listening on TCP port `443`.

Expected to show Nginx listening on HTTPS.

Conceptually:

```text
LISTEN ... :443 ... nginx
```

---

# 27. View Nginx Logs

### Error log

```bash
sudo tail -f /var/log/nginx/error.log
```

### Access log

```bash
sudo tail -f /var/log/nginx/access.log
```

## Purpose

Logs are useful when troubleshooting:

- Configuration issues
- TLS errors
- Request failures
- Permission problems
- Unexpected HTTP responses

---

# 28. Complete Day 15 Command Flow

```text
SSH INTO stapp03
       ↓
hostname
       ↓
Verify Nginx
       ↓
nginx -v
       ↓
Check SSL Files
       ↓
ls -l /tmp/nautilus.crt /tmp/nautilus.key
       ↓
Inspect Certificate
       ↓
openssl x509
       ↓
Create SSL Directory
       ↓
mkdir -p /etc/nginx/ssl
       ↓
Move Certificate + Key
       ↓
Create HTTPS Configuration
       ↓
/etc/nginx/conf.d/ssl.conf
       ↓
Create index.html
       ↓
nginx -t
       ↓
Start Nginx
       ↓
systemctl status nginx
       ↓
Local HTTPS Test
       ↓
curl -Ik https://localhost/
       ↓
Exit to Jump Host
       ↓
Remote HTTPS Test
       ↓
curl -Ik https://stapp03/
       ↓
HTTP 200 OK
```

---

# 29. Quick Command Cheat Sheet

| Command | Purpose |
|---|---|
| `ssh banner@stapp03` | Connect to App Server 3 |
| `hostname` | Verify target hostname |
| `sudo yum install nginx -y` | Install Nginx |
| `nginx -v` | Display Nginx version |
| `ls -l /tmp/nautilus.crt /tmp/nautilus.key` | Verify SSL files |
| `sudo openssl x509 ...` | Inspect certificate |
| `sudo mkdir -p /etc/nginx/ssl` | Create SSL directory |
| `sudo mv ...` | Move certificate/key |
| `sudo ls -l /etc/nginx/ssl/` | Verify SSL files |
| `sudo cat /etc/nginx/nginx.conf` | Inspect main Nginx config |
| `sudo vi /etc/nginx/conf.d/ssl.conf` | Create HTTPS configuration |
| `sudo nginx -t` | Validate Nginx configuration |
| `sudo systemctl start nginx` | Start Nginx |
| `sudo systemctl status nginx` | Check Nginx status |
| `sudo systemctl is-active nginx` | Check active state |
| `sudo systemctl is-enabled nginx` | Check boot enablement |
| `sudo systemctl enable nginx` | Enable Nginx at boot |
| `sudo ss -lntp \| grep :443` | Check HTTPS listener |
| `curl -Ik https://localhost/` | Test local HTTPS |
| `curl -Ik https://stapp03/` | Test remote HTTPS |
| `curl -k https://stapp03/` | Test page content |
| `sudo tail -f /var/log/nginx/error.log` | Monitor Nginx errors |
| `sudo tail -f /var/log/nginx/access.log` | Monitor Nginx requests |

---

# 30. Final Result

The Day 15 command sequence successfully produced:

```text
Nginx Installed
      ↓
SSL Certificate + Key Deployed
      ↓
HTTPS Server Block Configured
      ↓
Nginx Configuration Validated
      ↓
Nginx Running
      ↓
HTTPS :443
      ↓
Local Test → 200 OK
      ↓
Remote Test → 200 OK
```

**Day 15 completed successfully.**
