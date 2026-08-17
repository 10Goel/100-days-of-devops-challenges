# Day 20 — Nginx + PHP-FPM Application Setup

## 📌 Challenge Overview

Day 20 of the **100 Days of DevOps Challenge** focused on deploying and configuring a PHP-based application on **App Server 3 (`stapp03`)** in the Stratos Datacenter.

The objective was to install **Nginx** and **PHP-FPM 8.2**, configure Nginx to serve the application on **TCP port 8092**, and connect Nginx to PHP-FPM through the required Unix socket.

## 🎯 Requirements

The task required the following:

- Install Nginx on App Server 3.
- Configure Nginx to listen on port `8092`.
- Use `/var/www/html` as the Nginx document root.
- Install PHP-FPM version `8.2`.
- Configure PHP-FPM to use:
  `/var/run/php-fpm/default.sock`
- Configure Nginx and PHP-FPM to work together.
- Do not modify the existing `index.php` and `info.php` files.
- Verify the application from the jump host using:

```bash
curl http://stapp03:8092/index.php
```

## 🖥️ Environment

| Component | Details |
|---|---|
| Datacenter | Stratos DC |
| Server | App Server 3 |
| Hostname | `stapp03` |
| User | `banner` |
| Web Server | Nginx 1.20.1 |
| Application | PHP-based |
| PHP Version | 8.2 |
| Web Port | `8092` |
| Document Root | `/var/www/html` |
| PHP-FPM Socket | `/var/run/php-fpm/default.sock` |
| Verification Host | Jump Host |

## 🔧 Implementation

### 1. Nginx

Nginx was already available on the system and was verified with:

```bash
nginx -v
```

Output confirmed:

```text
nginx version: nginx/1.20.1
```

Nginx was configured to:

- Listen on port `8092`
- Serve content from `/var/www/html`
- Process PHP requests through PHP-FPM

### 2. PHP-FPM 8.2

PHP 8.2 was available through the CentOS Stream 9 AppStream module.

PHP-FPM was installed and configured to use:

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

An existing ACL setting caused PHP-FPM to ignore the socket owner/group directives. The conflicting `listen.acl_users` setting was disabled so that the required socket ownership and permissions could take effect.

### 3. Nginx ↔ PHP-FPM Integration

A dedicated Nginx virtual server configuration was created at:

```text
/etc/nginx/conf.d/php-app.conf
```

The configuration routes PHP requests to:

```text
unix:/var/run/php-fpm/default.sock
```

while keeping the application document root at:

```text
/var/www/html
```

## 🧪 Validation

PHP-FPM configuration was validated using:

```bash
sudo php-fpm -t
```

Result:

```text
configuration file /etc/php-fpm.conf test is successful
```

PHP-FPM was then started and enabled.

The Nginx configuration was tested with:

```bash
sudo nginx -t
```

The application was finally tested from the jump host:

```bash
curl http://stapp03:8092/index.php
```

Successful response:

```text
Welcome to xFusionCorp Industries!
```

## 🔄 Request Flow

```text
Jump Host
    |
    | HTTP :8092
    v
App Server 3 (stapp03)
    |
    v
Nginx
    |
    | FastCGI via Unix Socket
    v
/var/run/php-fpm/default.sock
    |
    v
PHP-FPM 8.2
    |
    v
/var/www/html/index.php
```

## 🧠 Key Learnings

- Nginx can act as a reverse gateway for PHP applications while PHP-FPM executes the PHP code.
- PHP-FPM commonly communicates with Nginx through a Unix socket.
- The Nginx `fastcgi_pass` directive must point to the exact PHP-FPM socket.
- The `SCRIPT_FILENAME` FastCGI parameter tells PHP-FPM which PHP file to execute.
- Nginx's `root` directive determines the application's document root.
- `nginx -t` and `php-fpm -t` should be used before restarting production services.
- Unix socket ownership and permissions are important for Nginx/PHP-FPM communication.
- Existing PHP application files should not be modified when the task specifically requires infrastructure configuration only.

## ✅ Final Status

**Day 20 — Completed Successfully**

The PHP application was successfully served through Nginx on port `8092` and executed by PHP-FPM 8.2 using the required Unix socket.

---
