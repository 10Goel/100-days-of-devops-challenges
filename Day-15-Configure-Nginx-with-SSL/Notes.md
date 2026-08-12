# Day 15 — Detailed Notes

## 1. Core Concept

Day 15 focuses on **Nginx, HTTPS/SSL configuration, Linux service management, TLS certificates, and end-to-end web-server validation**.

The overall workflow was:

```text
Connect to App Server
        ↓
Verify Nginx Installation
        ↓
Locate SSL Certificate and Key
        ↓
Inspect Certificate
        ↓
Create SSL Directory
        ↓
Move Certificate and Key
        ↓
Configure HTTPS Server Block
        ↓
Create index.html
        ↓
Validate Nginx Configuration
        ↓
Start Nginx
        ↓
Local HTTPS Test
        ↓
Remote HTTPS Test
        ↓
HTTP 200 OK
```

---

## 2. Why This Task Matters

Modern applications frequently need to be exposed securely over HTTPS.

A DevOps engineer may be responsible for:

- Installing a web server.
- Configuring TLS certificates.
- Managing private keys.
- Creating virtual/server blocks.
- Validating configuration files.
- Managing services through systemd.
- Testing endpoints locally and remotely.

A successful deployment therefore requires more than simply installing Nginx.

The complete workflow is:

```text
Application
    ↓
Web Server
    ↓
TLS Configuration
    ↓
HTTPS Listener
    ↓
Network Connectivity
    ↓
Remote Validation
```

---

## 3. Nginx

Nginx is a high-performance web server and reverse proxy.

In this challenge, Nginx was used as the HTTPS web server on:

```text
stapp03
```

The installed version was:

```text
nginx/1.20.1
```

The primary configuration file was:

```text
/etc/nginx/nginx.conf
```

Additional configuration was placed under:

```text
/etc/nginx/conf.d/
```

---

## 4. Nginx Document Root

The default document root used in the challenge was:

```text
/usr/share/nginx/html
```

The required page was:

```text
/usr/share/nginx/html/index.html
```

with:

```text
Welcome!
```

Nginx uses the document root to locate static web content requested by clients.

Conceptually:

```text
HTTPS Request
      ↓
Nginx
      ↓
root /usr/share/nginx/html
      ↓
index.html
      ↓
Welcome!
```

---

## 5. SSL/TLS

HTTPS is HTTP transported over TLS.

TLS provides:

- Encryption
- Server authentication through certificates
- Integrity protection

The challenge supplied:

```text
/tmp/nautilus.crt
/tmp/nautilus.key
```

These represent:

```text
nautilus.crt → SSL/TLS certificate
nautilus.key → private key
```

The files were moved to:

```text
/etc/nginx/ssl/
```

Result:

```text
/etc/nginx/ssl/nautilus.crt
/etc/nginx/ssl/nautilus.key
```

---

## 6. Certificate vs Private Key

These two files serve different purposes.

### Certificate

The certificate contains the server's public identity information and public key.

It can be inspected with:

```bash
sudo openssl x509 -in /etc/nginx/ssl/nautilus.crt -noout -subject -issuer -dates
```

### Private Key

The private key is used by the server during TLS operations.

It must be protected and should not be exposed unnecessarily.

Conceptually:

```text
Certificate
    ↓
Public information
    ↓
Can be distributed as part of TLS

Private Key
    ↓
Secret cryptographic material
    ↓
Must be protected
```

---

## 7. Inspecting an SSL Certificate

OpenSSL can display certificate information:

```bash
sudo openssl x509 -in /tmp/nautilus.crt -noout -subject -issuer -dates
```

Useful fields include:

```text
subject
issuer
notBefore
notAfter
```

This is useful when troubleshooting:

- Wrong certificate
- Wrong hostname
- Expired certificate
- Unexpected issuer
- Certificate deployment problems

---

## 8. Why Move SSL Files Out of `/tmp`?

The challenge initially provided:

```text
/tmp/nautilus.crt
/tmp/nautilus.key
```

`/tmp` is intended for temporary files.

For a web-server configuration, the files were moved to:

```text
/etc/nginx/ssl/
```

This creates a clearer and more maintainable layout:

```text
/etc/nginx/
└── ssl/
    ├── nautilus.crt
    └── nautilus.key
```

The certificate and private key can then be referenced directly by Nginx.

---

## 9. Nginx Configuration Structure

The main Nginx configuration already contained:

```nginx
include /etc/nginx/conf.d/*.conf;
```

This means Nginx automatically loads configuration files placed inside:

```text
/etc/nginx/conf.d/
```

A dedicated configuration file was therefore created:

```text
/etc/nginx/conf.d/ssl.conf
```

This is preferable to unnecessarily changing the main configuration file.

---

## 10. HTTPS Server Block

The HTTPS configuration used was:

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

## 11. Understanding `listen 443 ssl`

The directive:

```nginx
listen 443 ssl;
```

means Nginx listens for HTTPS traffic on TCP port:

```text
443
```

The `ssl` parameter enables TLS processing for that server block.

The IPv6 equivalent is:

```nginx
listen [::]:443 ssl;
```

Conceptually:

```text
Client
  |
  | TCP 443
  v
Nginx
  |
  | TLS
  v
HTTPS Application
```

---

## 12. Understanding `server_name`

The configuration used:

```nginx
server_name _;
```

The underscore is commonly used in a default/catch-all style server block.

It allows the server block to handle requests without requiring a specific hostname match.

---

## 13. Understanding `root`

The directive:

```nginx
root /usr/share/nginx/html;
```

sets the document root.

When a client requests:

```text
/
```

Nginx looks under:

```text
/usr/share/nginx/html/
```

and uses the configured index file:

```nginx
index index.html;
```

---

## 14. Understanding `ssl_certificate`

The directive:

```nginx
ssl_certificate /etc/nginx/ssl/nautilus.crt;
```

tells Nginx which certificate to present to HTTPS clients.

---

## 15. Understanding `ssl_certificate_key`

The directive:

```nginx
ssl_certificate_key /etc/nginx/ssl/nautilus.key;
```

tells Nginx where the corresponding private key is located.

The certificate and key must form a valid pair for the TLS configuration to work correctly.

---

## 16. The `location /` Block

The configuration included:

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

This instructs Nginx to:

1. Try the requested URI as a file.
2. Try it as a directory.
3. Return `404` if neither exists.

This is a simple and explicit static-content configuration.

---

## 17. Nginx Configuration Validation

Before starting or restarting Nginx, configuration should be validated:

```bash
sudo nginx -t
```

A successful result is:

```text
syntax is ok
test is successful
```

This is one of the most important operational habits when working with Nginx.

### Why?

A configuration syntax error can prevent Nginx from starting or reloading.

The workflow should therefore be:

```text
Edit Configuration
       ↓
nginx -t
       ↓
PASS?
   /       \
 NO         YES
 |           |
Fix         Start/Reload
```

---

## 18. Troubleshooting the Configuration Error

During the challenge, the first configuration validation failed because of a typo:

```text
unknown directive "server_name_"
```

The intended directive was:

```nginx
server_name _;
```

After correcting the typo:

```bash
sudo nginx -t
```

returned:

```text
syntax is ok
test is successful
```

### Lesson

Do not restart a service blindly after configuration changes.

First validate the configuration and correct only the reported issue.

---

## 19. systemctl and Nginx

Nginx is managed by systemd.

Common commands include:

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

For this challenge:

```bash
sudo systemctl start nginx
```

was used.

Then:

```bash
sudo systemctl status nginx --no-pager
```

confirmed:

```text
Active: active (running)
```

---

## 20. Active vs Enabled

These are two different service states.

### Active

```bash
sudo systemctl is-active nginx
```

Answers:

> Is Nginx running right now?

Expected:

```text
active
```

### Enabled

```bash
sudo systemctl is-enabled nginx
```

Answers:

> Is Nginx configured to start automatically during boot?

Expected:

```text
enabled
```

A production-ready service generally needs the appropriate combination of:

```text
active + enabled
```

---

## 21. Local HTTPS Testing

After Nginx was started, HTTPS was tested locally:

```bash
curl -Ik https://localhost/
```

The server returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
```

This confirmed that Nginx was successfully accepting HTTPS requests.

---

## 22. Why `curl -k` Was Used

The command used:

```bash
curl -Ik https://localhost/
```

The important options are:

```text
-I
↓
Request response headers only

-k
↓
Allow insecure TLS connection
```

The `-k` option tells curl not to reject the connection merely because the certificate is not trusted by the client's CA store.

This is common when testing lab certificates or self-signed/internal certificates.

> `-k` should be used deliberately for testing. It does not make an untrusted certificate trusted.

---

## 23. Understanding `HTTP/1.1 200 OK`

The response:

```text
HTTP/1.1 200 OK
```

indicates that the HTTP request was successfully processed.

In this challenge, it confirmed that:

```text
Client
   ↓
HTTPS
   ↓
Nginx
   ↓
Requested Resource
```

was functioning correctly.

---

## 24. Local vs Remote Testing

A local test:

```bash
curl -Ik https://localhost/
```

proves that Nginx works from the target server itself.

However, it does not prove that another machine can reach the service.

Therefore, the challenge required a remote test from the jump host:

```bash
curl -Ik https://stapp03/
```

This provides end-to-end validation.

---

## 25. Final Mental Model

Remember Day 15 as:

```text
SSL Certificate + Private Key
             ↓
      /etc/nginx/ssl/
             ↓
       Nginx HTTPS Config
             ↓
       listen :443 ssl
             ↓
       nginx -t
             ↓
      Start Nginx Service
             ↓
      Local HTTPS Test
             ↓
       Remote HTTPS Test
             ↓
          HTTP 200
             ↓
         Welcome!
```

---

## 26. Important Takeaways

### `nginx -t`

Use it to validate Nginx configuration:

```bash
sudo nginx -t
```

### `systemctl`

Use it to manage Nginx:

```bash
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl restart nginx
```

### `openssl`

Use it to inspect certificates:

```bash
sudo openssl x509 -in /etc/nginx/ssl/nautilus.crt -noout -subject -issuer -dates
```

### `curl`

Use it to test HTTPS:

```bash
curl -Ik https://localhost/
curl -Ik https://stapp03/
```

---

## 27. Security and Operational Lessons

### Protect private keys

The private key:

```text
/etc/nginx/ssl/nautilus.key
```

is sensitive cryptographic material.

It should not be unnecessarily exposed, copied, or committed to Git.

### Validate before service changes

Always use:

```bash
sudo nginx -t
```

before restarting or reloading Nginx after configuration changes.

### Prefer targeted configuration

The existing `/etc/nginx/nginx.conf` already included:

```text
/etc/nginx/conf.d/*.conf
```

so a dedicated configuration file was used rather than unnecessarily rewriting the main configuration.

### Test from the actual client path

A local test is useful, but the final remote test is essential when validating network accessibility.

---

## 28. Real-World Applications

The same concepts apply to:

```text
Nginx
  ↓
HTTPS :443
  ↓
TLS Certificate
  ↓
Application
```

They are also relevant to:

- Reverse proxies
- Load balancers
- API gateways
- Kubernetes Ingress
- Internal corporate HTTPS services
- Public web applications
- Microservice front ends

---

## 29. Final DevOps Lesson

The most important lesson from Day 15 is:

> **A secure web-server deployment must be validated at multiple layers: configuration, service, TLS, local connectivity, and remote connectivity.**

The final workflow was:

```text
Configuration
     ↓
Syntax Validation
     ↓
Service State
     ↓
TLS Endpoint
     ↓
Local Test
     ↓
Remote Test
     ↓
HTTP 200 OK
```

Day 15 demonstrates an important DevOps principle:

> **Make the configuration change, validate it safely, start the service, and verify the complete request path end-to-end.**

---

## 🏁 Day 15 Status

```text
Nginx Installation       → SUCCESS
SSL Certificate          → DEPLOYED
Private Key              → DEPLOYED
HTTPS Configuration      → SUCCESS
Configuration Test       → SUCCESS
Nginx Service            → RUNNING
Local HTTPS Test         → 200 OK
Remote HTTPS Test        → 200 OK
Challenge                → COMPLETED
```
