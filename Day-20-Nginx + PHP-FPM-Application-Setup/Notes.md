# Day 20 — Notes

## 📚 Topic

**Nginx + PHP-FPM Integration**

Day 20 introduced the practical deployment of a PHP application using Nginx as the web server and PHP-FPM as the FastCGI application processor, on App Server 2 in the Stratos Datacenter.

---

## 1. What is Nginx?

Nginx is a high-performance web server and reverse proxy.

In this task, Nginx was responsible for:

- Accepting HTTP requests.
- Listening on TCP port `8097`.
- Serving files from `/var/www/html`.
- Passing PHP requests to PHP-FPM.

The important configuration was:

```nginx
listen 8097;
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

## 5. Creating the Socket's Parent Directory

The task explicitly noted that the parent directory for the socket may not exist. Since `/var/run/php-fpm` did not exist by default, it had to be created manually before PHP-FPM could bind the socket:

```bash
sudo mkdir -p /var/run/php-fpm
```

Without this step, PHP-FPM fails to start with an error indicating it cannot create the socket file, since the target directory doesn't exist.

---

## 6. PHP-FPM Socket Permissions

The PHP-FPM pool was configured with:

```text
user = nginx
group = nginx
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

This allows the Nginx worker process to read from and write to the PHP-FPM socket.

### Important issue to watch for

If the configuration contains an ACL setting such as:

```text
listen.acl_users = apache,nginx
```

PHP-FPM will report:

```text
ACL set, listen.owner = 'nginx' is ignored
ACL set, listen.group = 'nginx' is ignored
```

This happens because ACL-based access takes priority over `listen.owner`/`listen.group`. If encountered, the conflicting line should be disabled:

```text
;listen.acl_users = apache,nginx
```

After restarting PHP-FPM, the socket permissions then follow the `listen.owner`, `listen.group`, and `listen.mode` values as expected.

---

## 7. Nginx FastCGI Configuration

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

Specifies where Nginx should send PHP requests — in this case, the Unix socket.

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

## 8. Document Root

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

They were deliberately left unchanged, per the task requirements.

---

## 9. Port 8097

The challenge specifically required Nginx to listen on:

```text
8097
```

Configuration:

```nginx
listen 8097;
```

This means the application can be accessed through:

```text
http://stapp02:8097/
```

---

## 10. Why Configuration Testing Matters

Before restarting services, configuration files should always be validated.

### PHP-FPM

```bash
sudo php-fpm -t
```

### Nginx

```bash
sudo nginx -t
```

These commands detect configuration syntax problems before they cause service failures or downtime.

---

## 11. Service Management

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

## 12. Troubleshooting `ss`

The `ss` command may be unavailable on a minimal CentOS install.

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

## 13. SELinux and Firewall Considerations

Depending on the server's hardening level, two additional blockers can prevent the application from being reachable even after Nginx and PHP-FPM are correctly configured:

- **SELinux** (check with `getenforce`) — if enforcing, contexts must be set on `/var/www/html` and the socket, and the `httpd_can_network_connect` boolean may need to be enabled.
- **firewalld** — the target port (`8097`) must be explicitly opened if the firewall is active.

If `getenforce` returns `Disabled`, these SELinux-specific steps can be skipped entirely.

---

## 14. Complete Request Flow

The final working architecture was:

```text
curl
  |
  | HTTP request
  | http://stapp02:8097/index.php
  v
Nginx :8097
  |
  | PHP request
  | FastCGI
  v
/var/run/php-fpm/default.sock
  |
  v
PHP-FPM 8.1
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

## 15. Verification

The final test was performed from the jump host:

```bash
curl http://stapp02:8097/index.php
```

A valid PHP response confirmed that:

- Network connectivity worked.
- Nginx was reachable on port `8097`.
- Nginx was serving the correct application.
- PHP requests were reaching PHP-FPM.
- PHP-FPM was executing `index.php`.
- The PHP application returned the expected response.

---

## 16. Deep Dive: PHP-FPM Sockets Explained

### TCP vs Unix Socket

PHP-FPM can be configured to listen in two different ways:

```text
listen = 127.0.0.1:9000      # TCP socket
listen = /var/run/php-fpm/default.sock   # Unix domain socket
```

| | TCP Socket | Unix Socket |
|---|---|---|
| Communication path | Through the network stack (loopback) | Through the filesystem, no network stack involved |
| Performance | Slightly slower (network overhead) | Faster, since it bypasses TCP/IP entirely |
| Access control | Controlled by IP/port binding | Controlled by filesystem permissions (owner/group/mode) |
| Use case | PHP-FPM on a different host than Nginx | PHP-FPM and Nginx on the same host (most common) |

Since Nginx and PHP-FPM run on the **same server** in this task, a Unix socket is the natural and required choice — it's faster and was explicitly mandated by the requirements.

### How the Socket File Works

When PHP-FPM starts, it creates a special file at the path given in `listen`. This isn't a regular file — it's a Unix domain socket file, visible with `ls -l` but only usable for local inter-process communication, not for storing data.

```bash
sudo ls -l /var/run/php-fpm/default.sock
```

The `s` at the start of the permission string (e.g. `srw-rw----`) confirms it's a socket, not a regular file.

If PHP-FPM is not running, this file will not exist at all — its presence is itself a basic health check.

### Why Ownership Matters

Nginx workers run as a specific user (commonly `nginx` or `www-data`). For Nginx to write to and read from the PHP-FPM socket, that same user (or a group it belongs to) must have permission on the socket file. This is why:

```text
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

...matters — `0660` means the owner and group get read/write access, but others get none. If Nginx runs as a *different* user than what's set here, every request will fail at the socket level, usually surfacing as a "502 Bad Gateway" in the browser or curl output.

### Why the Parent Directory Must Exist First

PHP-FPM does **not** create missing parent directories for the socket path. If `/var/run/php-fpm/` doesn't exist, PHP-FPM fails silently in the logs with something like "unable to bind listening socket" and the service won't start. Creating the directory ahead of time avoids this entirely:

```bash
sudo mkdir -p /var/run/php-fpm
```

Note also that `/var/run` is often a `tmpfs` (in-memory filesystem) that gets wiped on reboot. In production, this directory creation is typically handled by a `systemd-tmpfiles` rule or the PHP-FPM package's own service file, so it's worth checking whether the directory persists after a reboot in a real environment.

---

## 17. Common Issues and Mistakes (and How to Avoid Them)

### ❌ Mistake 1: Socket path mismatch between Nginx and PHP-FPM

**Symptom:** `502 Bad Gateway` when accessing the site.

**Cause:** The `fastcgi_pass` value in the Nginx config doesn't exactly match the `listen` value in PHP-FPM's pool config (e.g. a typo, or one uses `/var/run/php-fpm/default.sock` while the other uses `/run/php-fpm/default.sock`).

**Fix:** Always copy-paste the exact socket path between both files, or `grep` both configs and compare directly:

```bash
grep "listen =" /etc/php-fpm.d/www.conf
grep "fastcgi_pass" /etc/nginx/conf.d/php-app.conf
```

---

### ❌ Mistake 2: Forgetting to create the socket's parent directory

**Symptom:** PHP-FPM fails to start; `systemctl status php-fpm` shows a failed state.

**Cause:** The directory in the socket path (e.g. `/var/run/php-fpm/`) doesn't exist yet.

**Fix:** Create it before starting the service:

```bash
sudo mkdir -p /var/run/php-fpm
```

---

### ❌ Mistake 3: ACL settings silently overriding owner/group

**Symptom:** Socket permissions don't match what was configured in `listen.owner`/`listen.group`, even though those lines look correct.

**Cause:** A leftover or default `listen.acl_users` directive takes precedence over `listen.owner`/`listen.group` when POSIX ACLs are in play.

**Fix:** Check the PHP-FPM error log or startup output for lines like `ACL set, listen.owner = 'nginx' is ignored`, and comment out the conflicting line:

```bash
sudo sed -i 's|^listen\.acl_users|;listen.acl_users|' /etc/php-fpm.d/www.conf
```

---

### ❌ Mistake 4: Nginx and PHP-FPM running as different users

**Symptom:** `502 Bad Gateway`, and the Nginx error log shows `Permission denied` when connecting to the socket.

**Cause:** PHP-FPM's pool `user`/`group` (or `listen.owner`/`listen.group`) don't match the user Nginx workers run as.

**Fix:** Confirm the Nginx worker user:

```bash
grep "^user" /etc/nginx/nginx.conf
```

Then make sure PHP-FPM's `user`, `group`, `listen.owner`, and `listen.group` all match that same value.

---

### ❌ Mistake 5: Editing the wrong PHP-FPM pool file

**Symptom:** Changes don't seem to take effect no matter what's edited.

**Cause:** Multiple pool `.conf` files can exist under `/etc/php-fpm.d/`, or multiple PHP versions are installed side-by-side (e.g. PHP 7.4 and PHP 8.1), and the wrong one is being edited or the wrong version's service is running.

**Fix:** Confirm the active PHP-FPM version and its actual config path:

```bash
php-fpm -v
systemctl status php-fpm --no-pager
ls /etc/php-fpm.d/
```

---

### ❌ Mistake 6: Not testing configuration before restarting services

**Symptom:** A restart brings the whole service down instead of just failing to apply a bad config.

**Cause:** Skipping `nginx -t` or `php-fpm -t` before reloading/restarting.

**Fix:** Always validate first, restart second:

```bash
sudo php-fpm -t && sudo systemctl restart php-fpm
sudo nginx -t && sudo systemctl restart nginx
```

---

### ❌ Mistake 7: Forgetting `try_files` or misconfiguring the PHP location block

**Symptom:** Static files load, but any `.php` file returns a blank page, a raw file download, or a 404.

**Cause:** The `location ~ \.php$` block is missing, misplaced (e.g. placed after a catch-all `location /` block that intercepts requests first), or `fastcgi_param SCRIPT_FILENAME` is missing/incorrect.

**Fix:** Keep the PHP block distinct and ensure `SCRIPT_FILENAME` resolves correctly by checking `root` matches the actual file location:

```nginx
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
```

---

### ❌ Mistake 8: Modifying the provided application files

**Symptom:** Task validation fails even though the server appears to work.

**Cause:** `index.php` or `info.php` were edited, moved, or overwritten, even though the task explicitly said not to touch them.

**Fix:** Only touch Nginx/PHP-FPM configuration files. Leave anything under `/var/www/html` that was pre-provided completely untouched — verify with `ls -l` and avoid running any command that could overwrite the directory.

---

### ❌ Mistake 9: SELinux or firewall silently blocking access

**Symptom:** Everything looks correctly configured, services are running, sockets exist, but `curl` still times out or is refused from the jump host — while `curl localhost` on the server itself works fine.

**Cause:** SELinux contexts on `/var/www/html` or the socket, or the firewall not allowing the configured port, block external access even though local access works.

**Fix:** Check both explicitly:

```bash
getenforce
sudo firewall-cmd --list-ports
```

Only apply SELinux/firewall fixes if they're actually active — don't assume they're the cause without checking first.

---

### ❌ Mistake 10: Confusing "service is active" with "service is actually working"

**Symptom:** `systemctl status` shows both services as active/running, but the application still doesn't respond correctly.

**Cause:** A service can be "active" while still serving errors (e.g. Nginx running but pointing at the wrong socket, or PHP-FPM running but the pool config wasn't reloaded after an edit).

**Fix:** Always finish with an actual functional test, not just a status check:

```bash
curl -v http://stapp02:8097/index.php
```

The `-v` flag shows the full HTTP exchange, making it easier to see exactly where in the chain a request is failing.

---

## 🧠 Key Takeaways

1. **Nginx serves and routes HTTP requests; PHP-FPM executes PHP.**
2. **Nginx and PHP-FPM can communicate through a Unix socket.**
3. **The socket path must match on both sides.**
4. **The socket's parent directory must exist before PHP-FPM can start.**
5. **Socket ownership and permissions can affect Nginx → PHP-FPM communication.**
6. **`fastcgi_pass` connects Nginx to PHP-FPM.**
7. **`SCRIPT_FILENAME` tells PHP-FPM which PHP script to execute.**
8. **`nginx -t` and `php-fpm -t` are essential validation commands.**
9. **Existing application files should not be changed when the task only requires infrastructure configuration.**
10. **A successful `curl` response from the jump host validates the complete application path.**

---

## ✅ Day 20 Status

**Completed Successfully**

The PHP application was deployed behind Nginx on App Server 2 and successfully accessed through port `8097` using PHP-FPM 8.1 over a Unix socket.
