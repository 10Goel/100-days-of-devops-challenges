# Day 20 — Notes

## 📚 Topic

**Nginx + PHP-FPM Integration**

Day 20 introduced the practical deployment of a PHP application using Nginx as the web server and PHP-FPM as the FastCGI application processor.

---

## 1. What is Nginx?

Nginx is a high-performance web server and reverse proxy.

In this task, Nginx was responsible for:

- Accepting HTTP requests.
- Listening on TCP port `8092`.
- Serving files from `/var/www/html`.
- Passing PHP requests to PHP-FPM.

The important configuration was:

```nginx
listen 8092;
root /var/www/html;
```

---

## 2. What is PHP-FPM?

PHP-FPM stands for **PHP FastCGI Process Manager**.

Nginx does not execute PHP code itself. When a request targets a PHP file, Nginx passes the request to PHP-FPM.

The flow is:

```text
Client
  |
  v
Nginx
  |
  | FastCGI
  v
PHP-FPM
  |
  v
PHP Script
```

---

## 3. Why PHP-FPM is Required

Nginx is designed primarily to serve static content and proxy requests.

For a PHP application:

```text
index.php
```

Nginx needs another component to interpret and execute the PHP code.

That component is PHP-FPM.

---

## 4. Unix Socket

The challenge specifically required PHP-FPM to listen on:

```text
/var/run/php-fpm/default.sock
```

Instead of communicating over a TCP port such as:

```text
127.0.0.1:9000
```

Nginx communicates with PHP-FPM through the Unix domain socket:

```text
unix:/var/run/php-fpm/default.sock
```

The matching Nginx configuration was:

```nginx
fastcgi_pass unix:/var/run/php-fpm/default.sock;
```

The PHP-FPM and Nginx configurations must reference the **same socket path**.

---

## 5. PHP-FPM Socket Permissions

The PHP-FPM pool was configured with:

```text
user = nginx
group = nginx
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

This allows the Nginx process to access the PHP-FPM socket.

### Important issue encountered

The original configuration contained:

```text
listen.acl_users = apache,nginx
```

PHP-FPM reported:

```text
ACL set, listen.owner = 'nginx' is ignored
ACL set, listen.group = 'nginx' is ignored
```

This happened because ACL-based access was already configured.

The conflicting ACL setting was disabled:

```text
;listen.acl_users = apache,nginx
```

After restarting PHP-FPM, the socket permissions could follow the `listen.owner`, `listen.group`, and `listen.mode` configuration.

---

## 6. Nginx FastCGI Configuration

The PHP location block was:

```nginx
location ~ \.php$ {
    include /etc/nginx/fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
}
```

### `location ~ \.php$`

Matches requests ending in `.php`.

### `fastcgi_pass`

Specifies where Nginx should send PHP requests.

### `SCRIPT_FILENAME`

Tells PHP-FPM the actual PHP file that needs to be executed.

With:

```text
root /var/www/html;
```

a request for:

```text
/index.php
```

maps to:

```text
/var/www/html/index.php
```

---

## 7. Document Root

The challenge required:

```text
/var/www/html
```

Therefore Nginx was configured with:

```nginx
root /var/www/html;
```

The existing application files were already present:

```text
/var/www/html/index.php
/var/www/html/info.php
```

They were deliberately left unchanged.

---

## 8. Port 8092

The challenge specifically required Nginx to listen on:

```text
8092
```

Configuration:

```nginx
listen 8092;
```

This means the application can be accessed through:

```text
http://stapp03:8092/
```

---

## 9. Why Configuration Testing Matters

Before restarting services, configuration files should be validated.

### PHP-FPM

```bash
sudo php-fpm -t
```

### Nginx

```bash
sudo nginx -t
```

These commands detect configuration syntax problems before they cause service failures.

---

## 10. Service Management

PHP-FPM was enabled and started with:

```bash
sudo systemctl enable --now php-fpm
```

Nginx was enabled and started with:

```bash
sudo systemctl enable --now nginx
```

`enable` ensures the service starts automatically after boot, while `--now` starts it immediately.

---

## 11. Troubleshooting `ss`

The `ss` command was initially unavailable on App Server 3.

`ss` is provided by the `iproute` package.

It can be installed with:

```bash
sudo dnf install -y iproute
```

Useful commands:

```bash
sudo ss -lx
```

for Unix sockets, and:

```bash
sudo ss -lntp
```

for TCP listening sockets.

---

## 12. Complete Request Flow

The final working architecture was:

```text
curl
  |
  | HTTP request
  | http://stapp03:8092/index.php
  v
Nginx :8092
  |
  | PHP request
  | FastCGI
  v
/var/run/php-fpm/default.sock
  |
  v
PHP-FPM 8.2
  |
  v
/var/www/html/index.php
  |
  v
PHP response
  |
  v
Nginx
  |
  v
Client
```

---

## 13. Verification

The final test was performed from the jump host:

```bash
curl http://stapp03:8092/index.php
```

The successful response was:

```text
Welcome to xFusionCorp Industries!
```

This confirmed that:

- Network connectivity worked.
- Nginx was reachable on port `8092`.
- Nginx was serving the correct application.
- PHP requests were reaching PHP-FPM.
- PHP-FPM was executing `index.php`.
- The PHP application returned the expected response.

---

## 🧠 Key Takeaways

1. **Nginx serves and routes HTTP requests; PHP-FPM executes PHP.**
2. **Nginx and PHP-FPM can communicate through a Unix socket.**
3. **The socket path must match on both sides.**
4. **Socket ownership and permissions can affect Nginx → PHP-FPM communication.**
5. **`fastcgi_pass` connects Nginx to PHP-FPM.**
6. **`SCRIPT_FILENAME` tells PHP-FPM which PHP script to execute.**
7. **`nginx -t` and `php-fpm -t` are essential validation commands.**
8. **Existing application files should not be changed when the task only requires infrastructure configuration.**
9. **A successful `curl` response from the jump host validates the complete application path.**

---

## ✅ Day 20 Status

**Completed Successfully**

The PHP application was deployed behind Nginx on App Server 3 and successfully accessed through port `8092` using PHP-FPM 8.2.
