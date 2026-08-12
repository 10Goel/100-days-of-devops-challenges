# Day 15 — Configure Nginx with HTTPS/SSL on App Server 3

## 📌 Overview

Day 15 of the DevOps challenge focuses on preparing **App Server 3 (`stapp03`)** in the **Stratos Datacenter** for application deployment.

The task required installing/configuring **Nginx**, deploying the provided SSL certificate and private key, configuring HTTPS, creating a simple web page, and validating secure connectivity from the **jump host**.

> **Objective:** Serve the application page over HTTPS using the provided `nautilus.crt` certificate and `nautilus.key` private key.

---

## 🎯 Objectives

The challenge required:

- Install and configure Nginx on `stapp03`.
- Locate the provided SSL files:
  - `/tmp/nautilus.crt`
  - `/tmp/nautilus.key`
- Move the certificate and key to an appropriate secure location.
- Configure Nginx to listen on HTTPS/TCP `443`.
- Create an `index.html` containing:
  ```text
  Welcome!
  ```
- Validate the Nginx configuration.
- Start Nginx.
- Verify HTTPS locally.
- Perform the final HTTPS test from the jump host using the App Server 3 hostname.

---

## 🏗️ Environment

| Component | Details |
|---|---|
| Datacenter | Stratos Datacenter |
| Jump Host | `jump-host` |
| Target Server | `stapp03` |
| Target User | `banner` |
| Web Server | Nginx |
| Nginx Version | `1.20.1` |
| HTTPS Port | `443/TCP` |
| Document Root | `/usr/share/nginx/html` |
| SSL Directory | `/etc/nginx/ssl` |
| Certificate | `/etc/nginx/ssl/nautilus.crt` |
| Private Key | `/etc/nginx/ssl/nautilus.key` |
| Web Page | `/usr/share/nginx/html/index.html` |
| Required Content | `Welcome!` |
| Final Test | `curl -Ik https://stapp03/` |

---

## 🔐 SSL Files

The challenge provided the following files:

```text
/tmp/nautilus.crt
/tmp/nautilus.key
```

They were moved to:

```text
/etc/nginx/ssl/nautilus.crt
/etc/nginx/ssl/nautilus.key
```

This keeps TLS material in a dedicated Nginx SSL directory instead of leaving it under `/tmp`.

The certificate was inspected using OpenSSL before deployment:

```bash
sudo openssl x509 -in /tmp/nautilus.crt -noout -subject -issuer -dates
```

---

## ⚙️ Nginx Configuration

The main Nginx configuration already included:

```nginx
include /etc/nginx/conf.d/*.conf;
```

Therefore, a dedicated HTTPS server block was created at:

```text
/etc/nginx/conf.d/ssl.conf
```

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

### Why use a separate configuration file?

Using `/etc/nginx/conf.d/ssl.conf` keeps the HTTPS configuration modular and avoids unnecessary modification of the main `/etc/nginx/nginx.conf`.

---

## 🌐 Web Content

The required page was created under the Nginx document root:

```text
/usr/share/nginx/html/index.html
```

Content:

```text
Welcome!
```

Verification:

```bash
cat /usr/share/nginx/html/index.html
```

Expected:

```text
Welcome!
```

---

## 🧪 Configuration Validation

Before starting Nginx, the configuration was tested with:

```bash
sudo nginx -t
```

The final validation returned:

```text
syntax is ok
test is successful
```

### Troubleshooting Note

The first configuration test failed because the server block contained a typo:

```text
server_name_
```

instead of:

```text
server_name _
```

After correcting the directive, `nginx -t` passed successfully.

This demonstrates an important operational practice:

> **Always validate a web-server configuration before starting or restarting the service.**

---

## 🚀 Nginx Service

Nginx was started with:

```bash
sudo systemctl start nginx
```

Service status was verified with:

```bash
sudo systemctl status nginx --no-pager
```

The final state was:

```text
Active: active (running)
```

---

## 🔎 Local HTTPS Validation

HTTPS was first tested directly on `stapp03`:

```bash
curl -Ik https://localhost/
```

The server returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
Content-Type: text/html
```

This confirmed that:

- Nginx was running.
- TCP/443 was accepting HTTPS traffic.
- The TLS configuration was functional.
- The web server successfully returned the requested resource.

The `-k` option was used because the provided lab certificate is not necessarily trusted by the local CA trust store.

---

## 🌍 Final Remote Validation

After the local test succeeded, the final validation was performed from the jump host:

```bash
curl -Ik https://stapp03/
```

The request returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
```

This verified end-to-end HTTPS connectivity:

```text
Jump Host
    |
    | HTTPS / TCP 443
    v
stapp03
    |
    v
Nginx
    |
    v
SSL/TLS
    |
    v
/usr/share/nginx/html/index.html
    |
    v
Welcome!
```

---

## ✅ Validation Checklist

Before considering the challenge complete:

- [x] Connected to `stapp03`.
- [x] Verified Nginx installation.
- [x] Verified `/tmp/nautilus.crt`.
- [x] Verified `/tmp/nautilus.key`.
- [x] Moved certificate and key to `/etc/nginx/ssl/`.
- [x] Created `/etc/nginx/conf.d/ssl.conf`.
- [x] Configured Nginx for HTTPS on port `443`.
- [x] Created `/usr/share/nginx/html/index.html`.
- [x] Added `Welcome!` to `index.html`.
- [x] Validated configuration using `nginx -t`.
- [x] Started Nginx.
- [x] Confirmed Nginx is `active (running)`.
- [x] Tested HTTPS locally.
- [x] Tested HTTPS remotely from the jump host.
- [x] Received `HTTP/1.1 200 OK`.

---

## 📚 Concepts Practiced

- Nginx installation and configuration
- Linux service management
- `systemctl`
- Nginx server blocks
- HTTPS
- SSL/TLS certificates
- Private keys
- OpenSSL certificate inspection
- Nginx document root
- TCP port `443`
- Configuration validation with `nginx -t`
- `curl`
- Local vs remote connectivity testing
- Modular Nginx configuration
- Production-style web server deployment

---

## 🧠 Key DevOps Lessons

### 1. Validate before restarting

Always run:

```bash
sudo nginx -t
```

before restarting or starting Nginx after configuration changes.

### 2. Keep configuration modular

Instead of unnecessarily modifying the main configuration, use:

```text
/etc/nginx/conf.d/
```

for additional server configurations.

### 3. HTTPS requires both certificate and private key

Nginx needs:

```nginx
ssl_certificate ...
ssl_certificate_key ...
```

to terminate TLS connections.

### 4. Local success is not enough

A local test proves that the service works on the target host, but the final remote test proves that the application is reachable through the actual network path.

### 5. Verify end-to-end

The complete validation flow was:

```text
Configuration
      ↓
nginx -t
      ↓
Nginx Service
      ↓
Local HTTPS Test
      ↓
Remote HTTPS Test
      ↓
HTTP 200 OK
```

---

## 🏁 Final Result

**Day 15 was completed successfully.**

Nginx on `stapp03` was configured to serve the application over HTTPS using the provided SSL certificate and private key. The service was validated locally and successfully accessed from the jump host.

```text
stapp03
   ↓
Nginx
   ↓
HTTPS :443
   ↓
SSL Certificate
   ↓
index.html
   ↓
Welcome!
   ↓
HTTP 200 OK
```

> **Final DevOps Lesson:**  
> **Secure application deployment is not complete until the configuration, service state, TLS endpoint, and remote connectivity have all been verified.**
