# Day 16 --- Commands Reference

This file contains the commands used during the Day 16 Nginx Load
Balancer challenge, organized in the order in which they are useful
during the task.

------------------------------------------------------------------------

## 1. Connect to Application Server 1

``` bash
ssh tony@stapp01
```

Check listening ports:

``` bash
sudo ss -lntp
```

Exit:

``` bash
exit
```

### Important finding

Apache/httpd was listening on:

``` text
*:5003
```

------------------------------------------------------------------------

## 2. Connect to Application Server 2

``` bash
ssh steve@stapp02
```

Check listening ports:

``` bash
sudo ss -lntp
```

Exit:

``` bash
exit
```

### Important finding

Apache/httpd was listening on:

``` text
*:5003
```

------------------------------------------------------------------------

## 3. Connect to Application Server 3

``` bash
ssh banner@stapp03
```

Check listening ports:

``` bash
sudo ss -lntp
```

Exit:

``` bash
exit
```

The existing Apache backend port must be preserved.

------------------------------------------------------------------------

# 4. Connect to the Load Balancer

``` bash
ssh loki@stlb01
```

------------------------------------------------------------------------

# 5. Check Nginx Installation

``` bash
nginx -v
```

Observed version:

``` text
nginx version: nginx/1.20.1
```

If Nginx is not installed in another similar lab environment, the
installation command may be:

``` bash
sudo yum install nginx -y
```

------------------------------------------------------------------------

# 6. Back Up the Nginx Configuration

Before changing the configuration:

``` bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
```

This creates a backup that can be used for rollback.

------------------------------------------------------------------------

# 7. Edit the Main Nginx Configuration

The task required modification of:

``` text
/etc/nginx/nginx.conf
```

Open it with:

``` bash
sudo vi /etc/nginx/nginx.conf
```

Configuration used:

``` nginx
events {
}

http {

    upstream nautlius_backend {
        server stapp01:5003;
        server stapp02:5003;
        server stapp03:5003;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://nautlius_backend;
        }
    }
}
```

------------------------------------------------------------------------

# 8. Validate Nginx Configuration

Always test the configuration before restarting:

``` bash
sudo nginx -t
```

Expected successful result:

``` text
syntax is ok
test is successful
```

------------------------------------------------------------------------

# 9. Restart Nginx

After a successful configuration test:

``` bash
sudo systemctl restart nginx
```

------------------------------------------------------------------------

# 10. Check Nginx Status

``` bash
sudo systemctl status nginx
```

Expected:

``` text
Active: active (running)
```

------------------------------------------------------------------------

# 11. Enable Nginx at Boot

The service was also enabled:

``` bash
sudo systemctl enable nginx
```

This creates the appropriate systemd symlink so Nginx can start
automatically during boot.

------------------------------------------------------------------------

# 12. Test Through Localhost

The first successful HTTP test was:

``` bash
curl http://localhost:80
```

Expected application response:

``` text
Welcome to xFusionCorp Industries!
```

------------------------------------------------------------------------

# 13. Final Load Balancer Test

The task specifically required testing the load balancer hostname:

``` bash
curl http://stlb01:80
```

Successful response:

``` text
Welcome to xFusionCorp Industries!
```

This confirmed that the application was accessible through the Nginx
load balancer.

------------------------------------------------------------------------

# 14. Useful Diagnostic Commands

### Check listening ports

``` bash
sudo ss -lntp
```

### Check Nginx version

``` bash
nginx -v
```

### Validate Nginx configuration

``` bash
sudo nginx -t
```

### Check Nginx service

``` bash
sudo systemctl status nginx
```

### Restart Nginx

``` bash
sudo systemctl restart nginx
```

### Start Nginx

``` bash
sudo systemctl start nginx
```

### Stop Nginx

``` bash
sudo systemctl stop nginx
```

### Reload Nginx configuration

``` bash
sudo systemctl reload nginx
```

A reload is often preferable in production when the configuration can be
reloaded without interrupting existing connections.

### Enable Nginx at boot

``` bash
sudo systemctl enable nginx
```

### Check HTTP response

``` bash
curl http://stlb01:80
```

------------------------------------------------------------------------

# 15. Rollback Command

If the new configuration ever needs to be reverted:

``` bash
sudo cp /etc/nginx/nginx.conf.backup /etc/nginx/nginx.conf
```

Then validate:

``` bash
sudo nginx -t
```

If the validation succeeds:

``` bash
sudo systemctl restart nginx
```

------------------------------------------------------------------------

# 16. Complete Command Sequence

For reference, the essential sequence was:

``` bash
ssh tony@stapp01
sudo ss -lntp
exit

ssh steve@stapp02
sudo ss -lntp
exit

ssh banner@stapp03
sudo ss -lntp
exit

ssh loki@stlb01
nginx -v

sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup

sudo vi /etc/nginx/nginx.conf

sudo nginx -t

sudo systemctl restart nginx

sudo systemctl status nginx

sudo systemctl enable nginx

curl http://localhost:80

curl http://stlb01:80
```

------------------------------------------------------------------------

# 17. Expected Final Output

The final command:

``` bash
curl http://stlb01:80
```

returned:

``` text
Welcome to xFusionCorp Industries!
```

This was the final verification that the Nginx load balancer was
successfully forwarding HTTP traffic to the Nautilus application
backend.

------------------------------------------------------------------------

## ⚠️ Important Precautions

### Do not change the Apache port

The existing Apache backend was:

``` text
5003
```

Do not change it simply because Nginx uses port `80`.

### Do not confuse `8080` with Apache

During inspection, `8080` was associated with `ttyd`, while Apache/httpd
was listening on `5003`.

### Validate before restarting

Always use:

``` bash
sudo nginx -t
```

before:

``` bash
sudo systemctl restart nginx
```

### Preserve the original configuration

A backup was created using:

``` bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
```

------------------------------------------------------------------------

# Day 16 Command Summary

  --------------------------------------------------------------------------------------------------
  Purpose                             Command
  ----------------------------------- --------------------------------------------------------------
  SSH to App 1                        `ssh tony@stapp01`

  SSH to App 2                        `ssh steve@stapp02`

  SSH to App 3                        `ssh banner@stapp03`

  SSH to Load Balancer                `ssh loki@stlb01`

  Check ports                         `sudo ss -lntp`

  Check Nginx version                 `nginx -v`

  Back up config                      `sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup`

  Edit config                         `sudo vi /etc/nginx/nginx.conf`

  Validate config                     `sudo nginx -t`

  Restart Nginx                       `sudo systemctl restart nginx`

  Check service                       `sudo systemctl status nginx`

  Enable at boot                      `sudo systemctl enable nginx`

  Test locally                        `curl http://localhost:80`

  Final test                          `curl http://stlb01:80`
  --------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## ✅ Day 16 Status

**Nginx HTTP Load Balancer --- Successfully Configured and Verified**
