# Day 20 — Commands Reference

This document contains the commands used during the Day 20 Nginx + PHP-FPM configuration task.

## 1. Verify Host

```bash
hostname
```

```bash
cat /etc/os-release
```

## 2. Verify Nginx

```bash
nginx -v
```

Expected:

```text
nginx version: nginx/1.20.1
```

## 3. Check PHP Modules

```bash
sudo dnf module list php
```

## 4. Enable PHP 8.2

```bash
sudo dnf module reset php -y
sudo dnf module enable php:8.2 -y
```

## 5. Install PHP-FPM

```bash
sudo dnf install -y php-fpm php-cli
```

Verify:

```bash
php -v
```

## 6. Verify Application Files

```bash
ls -l /var/www/html/
```

The task-provided files were:

```text
index.php
info.php
```

These files were not modified.

## 7. Create PHP-FPM Socket Directory

```bash
sudo mkdir -p /var/run/php-fpm
```

## 8. Backup PHP-FPM Configuration

```bash
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.bak
```

## 9. Configure PHP-FPM with sed

Set the PHP-FPM process user:

```bash
sudo sed -i 's|^user = .*|user = nginx|' /etc/php-fpm.d/www.conf
```

Set the PHP-FPM process group:

```bash
sudo sed -i 's|^group = .*|group = nginx|' /etc/php-fpm.d/www.conf
```

Set the required Unix socket:

```bash
sudo sed -i 's|^listen = .*|listen = /var/run/php-fpm/default.sock|' /etc/php-fpm.d/www.conf
```

Configure socket ownership:

```bash
sudo sed -i 's|^[;#]*listen.owner[[:space:]]*=.*|listen.owner = nginx|' /etc/php-fpm.d/www.conf
```

```bash
sudo sed -i 's|^[;#]*listen.group[[:space:]]*=.*|listen.group = nginx|' /etc/php-fpm.d/www.conf
```

```bash
sudo sed -i 's|^[;#]*listen.mode[[:space:]]*=.*|listen.mode = 0660|' /etc/php-fpm.d/www.conf
```

## 10. Disable Conflicting PHP-FPM ACL

The configuration contained:

```text
listen.acl_users = apache,nginx
```

This caused PHP-FPM to ignore `listen.owner` and `listen.group`.

It was disabled with:

```bash
sudo sed -i 's|^listen\.acl_users|;listen.acl_users|' /etc/php-fpm.d/www.conf
```

## 11. Verify PHP-FPM Settings

```bash
grep -E '^(user|group|listen|listen.owner|listen.group|listen.mode)' /etc/php-fpm.d/www.conf
```

## 12. Test PHP-FPM Configuration

```bash
sudo php-fpm -t
```

Expected:

```text
configuration file /etc/php-fpm.conf test is successful
```

## 13. Start and Enable PHP-FPM

```bash
sudo systemctl enable --now php-fpm
```

Check:

```bash
sudo systemctl status php-fpm --no-pager
```

## 14. Verify PHP-FPM Socket

```bash
sudo ls -l /var/run/php-fpm/default.sock
```

Optional socket check:

```bash
sudo ss -lx | grep default.sock
```

If `ss` is unavailable:

```bash
sudo dnf install -y iproute
```

## 15. Create Nginx Application Configuration

```bash
sudo tee /etc/nginx/conf.d/php-app.conf > /dev/null <<'EOF'
server {
    listen 8092;
    server_name stapp03;

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
EOF
```

## 16. Review Nginx Configuration

```bash
cat /etc/nginx/conf.d/php-app.conf
```

## 17. Test Nginx Configuration

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

## 18. Start and Enable Nginx

```bash
sudo systemctl enable --now nginx
```

Check:

```bash
sudo systemctl status nginx --no-pager
```

## 19. Verify Nginx Port

```bash
sudo ss -lntp | grep 8092
```

## 20. Test PHP Locally

```bash
curl http://localhost:8092/index.php
```

Test the second application file:

```bash
curl http://localhost:8092/info.php
```

## 21. Test Using Server Hostname

```bash
curl http://stapp03:8092/index.php
```

## 22. Final Verification from Jump Host

Exit App Server 3:

```bash
exit
```

From the jump host:

```bash
curl http://stapp03:8092/index.php
```

Successful result:

```text
Welcome to xFusionCorp Industries!
```

## 🔑 Most Important Commands

### PHP-FPM configuration test

```bash
sudo php-fpm -t
```

### Nginx configuration test

```bash
sudo nginx -t
```

### Final application test

```bash
curl http://stapp03:8092/index.php
```
