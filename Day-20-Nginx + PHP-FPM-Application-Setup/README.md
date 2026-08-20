# Day 20 — Nginx + PHP-FPM Application Setup

## 📌 Challenge Overview

Day 20 of the **100 Days of DevOps Challenge** focused on deploying and configuring a PHP-based application on **App Server 2 (`stapp02`)** in the **Stratos Datacenter**.

The Nautilus application development team is planning to launch a new PHP-based application on Nautilus infra. The objective was to install **Nginx**, configure **PHP-FPM version 8.1**, and connect the two components through a Unix socket so the application could be served on **TCP port 8097**.

## 🎯 Requirements

The task required the following:

- Install Nginx on App Server 2.
- Configure Nginx to listen on port `8097`.
- Use `/var/www/html` as the Nginx document root.
- Install PHP-FPM version `8.1`.
- Configure PHP-FPM to use the Unix socket:
  `/var/run/php-fpm/default.sock`
  (create the parent directory if it doesn't exist).
- Configure Nginx and PHP-FPM to work together.
- Do not modify the existing `index.php` and `info.php` files.
- Verify the application from the jump host using:

```bash
curl http://stapp02:8097/index.php
```

## 🖥️ Environment

| Component | Details |
|---|---|
| Datacenter | Stratos DC |
| Server | App Server 2 |
| Hostname | `stapp02` |
| Web Server | Nginx |
| Application | PHP-based |
| PHP-FPM Version | 8.1 |
| Web Port | `8097` |
| Document Root | `/var/www/html` |
| PHP-FPM Socket | `/var/run/php-fpm/default.sock` |
| Verification Host | Jump Host |

## 🔧 Implementation

### 1. Nginx

Nginx was installed on App Server 2 and configured to:

- Listen on port `8097`
- Serve content from `/var/www/html`
- Route PHP requests to PHP-FPM

### 2. PHP-FPM 8.1

PHP-FPM 8.1 was installed via the appropriate module/repo for the OS and configured to use:

```text
/var/run/php-fpm/default.sock
```

The PHP-FPM pool was configured with:

```text
user = nginx
group = nginx
listen = /var/run/php-fpm/default.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

The parent directory `/var/run/php-fpm` was created since it did not already exist, ensuring PHP-FPM could bind the socket successfully.

### 3. Nginx ↔ PHP-FPM Integration

The Nginx server block was configured to route requests ending in `.php` to PHP-FPM over the Unix socket, while keeping the document root at `/var/www/html`:

```nginx
server {
    listen 8097;
    server_name stapp02;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
    }
}
```

## 🧪 Validation

PHP-FPM configuration was validated using:

```bash
sudo php-fpm -t
```

Nginx configuration was validated using:

```bash
sudo nginx -t
```

Both services were enabled and started so they would persist across reboots.

The application was finally tested from the jump host:

```bash
curl http://stapp02:8097/index.php
```

A successful, valid PHP response confirmed the setup was working end to end.

## 🔄 Request Flow

```text
Jump Host
    |
    | HTTP :8097
    v
App Server 2 (stapp02)
    |
    v
Nginx
    |
    | FastCGI via Unix Socket
    v
/var/run/php-fpm/default.sock
    |
    v
PHP-FPM 8.1
    |
    v
/var/www/html/index.php
```

## 🧠 Key Learnings

- Nginx acts as the web-facing server while PHP-FPM handles PHP execution behind the scenes.
- Nginx and PHP-FPM commonly communicate over a Unix domain socket rather than a TCP port.
- The Nginx `fastcgi_pass` directive must point to the exact same socket path configured in PHP-FPM's `listen` directive.
- The `SCRIPT_FILENAME` FastCGI parameter tells PHP-FPM which script to execute.
- Socket parent directories must exist beforehand, or PHP-FPM will fail to start.
- Socket ownership (`listen.owner`, `listen.group`, `listen.mode`) must allow the Nginx process to read/write to the socket.
- `nginx -t` and `php-fpm -t` should always be run before restarting services in production.
- Pre-provided application files should never be modified when the task only asks for infrastructure configuration.

## ✅ Final Status

**Day 20 — Completed Successfully**

The PHP application was successfully served through Nginx on port `8097` on App Server 2, with PHP-FPM 8.1 executing the PHP code via the required Unix socket.

---
