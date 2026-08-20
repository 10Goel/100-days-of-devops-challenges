# Day 20 — Commands Reference

This document contains the commands used during the Day 20 Nginx + PHP-FPM configuration task on App Server 2.

## 1. Log In and Verify Host

```bash
ssh steve@stapp02
```

```bash
hostname
```

```bash
cat /etc/os-release
```

## 2. Install Nginx

```bash
sudo yum install nginx -y
```

Verify:

```bash
nginx -v
```

## 3. Enable PHP 8.1 Module Stream

```bash
sudo yum install epel-release -y

# Add the Remi repo for PHP versions not in the base AppStream repo
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-9.rpm -y

sudo dnf module reset php -y
sudo dnf module enable php:remi-8.1 -y
```

## 4. Install PHP-FPM 8.1

```bash
sudo yum install php-fpm -y
```

Verify version:

```bash
php-fpm -v
```

## 5. Verify Application Files

```bash
ls -l /var/www/html/
```

Expected task-provided files:

```text
index.php
info.php
```

These files were **not modified**.

## 6. Create PHP-FPM Socket Parent Directory

```bash
sudo mkdir -p /var/run/php-fpm
```

## 7. Backup PHP-FPM Configuration

```bash
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.bak
```

## 8. Configure PHP-FPM with sed

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

## 9. Check for Conflicting ACL Settings (if present)

If the config contains:

```text
listen.acl_users = apache,nginx
```

it overrides `listen.owner`/`listen.group`. Disable it with:

```bash
sudo sed -i 's|^listen\.acl_users|;listen.acl_users|' /etc/php-fpm.d/www.conf
```

## 10. Verify PHP-FPM Settings

```bash
grep -E '^(user|group|listen|listen.owner|listen.group|listen.mode)' /etc/php-fpm.d/www.conf
```

## 11. Test PHP-FPM Configuration

```bash
sudo php-fpm -t
```

Expected:

```text
configuration file /etc/php-fpm.conf test is successful
```

## 12. Start and Enable PHP-FPM

```bash
sudo systemctl enable --now php-fpm
```

Check status:

```bash
sudo systemctl status php-fpm --no-pager
```

## 13. Verify PHP-FPM Socket

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

## 14. Create Nginx Application Configuration

```bash
sudo tee /etc/nginx/conf.d/php-app.conf > /dev/null <<'EOF'
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
EOF
```

## 15. Review Nginx Configuration

```bash
cat /etc/nginx/conf.d/php-app.conf
```

## 16. Test Nginx Configuration

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

## 17. Start and Enable Nginx

```bash
sudo systemctl enable --now nginx
```

Check status:

```bash
sudo systemctl status nginx --no-pager
```

## 18. Verify Nginx Port

```bash
sudo ss -lntp | grep 8097
```

## 19. Fix Ownership/Permissions (if needed)

```bash
sudo chown -R nginx:nginx /var/www/html
```

## 20. Check SELinux and Firewall (if applicable)

```bash
getenforce
```

If `Enforcing`:

```bash
sudo setsebool -P httpd_can_network_connect 1
sudo chcon -R -t httpd_sys_content_t /var/www/html
sudo chcon -t httpd_sys_rw_content_t /var/run/php-fpm/default.sock
```

Check firewall:

```bash
sudo firewall-cmd --list-ports
sudo firewall-cmd --permanent --add-port=8097/tcp
sudo firewall-cmd --reload
```

## 21. Test PHP Locally

```bash
curl http://localhost:8097/index.php
```

Test the second application file:

```bash
curl http://localhost:8097/info.php
```

## 22. Test Using Server Hostname

```bash
curl http://stapp02:8097/index.php
```

## 23. Final Verification from Jump Host

Exit App Server 2:

```bash
exit
```

From the jump host:

```bash
curl http://stapp02:8097/index.php
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
curl http://stapp02:8097/index.php
```
