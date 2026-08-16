# 📘 Day 19 — Notes

## Apache Static Website Hosting

### 1. What Was the Objective?

The objective of Day 19 was to prepare **Application Server 3 (`stapp03`)** to host two static websites.

The server needed to:

1. Run Apache HTTP Server.
2. Listen on TCP port `5002`.
3. Host the `blog` website.
4. Host the `cluster` website.
5. Make both websites accessible through their respective URL paths.

---

# 2. What is Apache HTTP Server?

Apache HTTP Server, commonly called **Apache** or **httpd**, is an open-source web server used to serve web content over HTTP and HTTPS.

In this task, Apache acts as the web server responsible for serving static HTML files.

The basic flow is:

```text
Client
   |
   | HTTP Request
   v
Apache (httpd)
   |
   | Reads requested files
   v
/var/www/html/
   |
   +---- blog/
   |
   +---- cluster/
```

---

# 3. What is httpd?

On CentOS/RHEL-based systems, the Apache package and service are commonly referred to as `httpd`.

Install:

```bash
sudo yum install -y httpd
```

Check the version:

```bash
httpd -v
```

Manage the service using:

```bash
sudo systemctl start httpd
sudo systemctl restart httpd
sudo systemctl status httpd
```

---

# 4. Apache Configuration File

The main Apache configuration file used in this task was:

```text
/etc/httpd/conf/httpd.conf
```

The default HTTP port is normally:

```text
Listen 80
```

For this challenge it had to be changed to:

```text
Listen 5002
```

This means Apache accepts HTTP connections on TCP port `5002`.

---

# 5. Why Port 5002?

The challenge specifically required the websites to be accessible through:

```text
http://localhost:5002/blog/
http://localhost:5002/cluster/
```

Therefore Apache could not remain on its default port `80`.

Changing:

```text
Listen 80
```

to:

```text
Listen 5002
```

makes Apache listen on the required port.

---

# 6. Apache Document Root

Apache's default document root on this system was:

```text
/var/www/html
```

The two websites were placed below it:

```text
/var/www/html/blog
/var/www/html/cluster
```

Therefore:

```text
http://localhost:5002/blog/
```

maps to:

```text
/var/www/html/blog/
```

and:

```text
http://localhost:5002/cluster/
```

maps to:

```text
/var/www/html/cluster/
```

This is an important concept:

```text
URL Path
   ↓
Apache Document Root
   ↓
Filesystem Directory
```

---

# 7. Why Were the Website Backups Copied?

The challenge provided the website backups on the jump host:

```text
/home/thor/blog
/home/thor/cluster
```

These files needed to be transferred to `stapp03`.

`scp` was used because it securely copies files over SSH.

Example:

```bash
scp -r /home/thor/blog banner@stapp03:/tmp/
```

The `-r` option means recursive copying, allowing an entire directory and its contents to be transferred.

---

# 8. File Ownership

Apache needs permission to access the website files.

The ownership was configured as:

```bash
sudo chown -R apache:apache /var/www/html/blog
sudo chown -R apache:apache /var/www/html/cluster
```

Here:

- `apache` before `:` = owner
- `apache` after `:` = group
- `-R` = recursively apply to all files/directories

---

# 9. File Permissions

The website directories were configured with:

```bash
sudo chmod -R 755 /var/www/html/blog
sudo chmod -R 755 /var/www/html/cluster
```

`755` means:

```text
Owner  → read + write + execute
Group  → read + execute
Others → read + execute
```

For web content, this allows Apache to read and traverse the website directories.

---

# 10. SELinux

SELinux can restrict what processes are allowed to access.

Because this was a CentOS/RHEL-style environment, the Apache web content needed the appropriate SELinux context.

The context was restored using:

```bash
sudo restorecon -Rv /var/www/html/
```

This applies the expected SELinux labels for files under the Apache web root.

---

# 11. Why Test Apache Configuration?

Before restarting Apache, configuration syntax should be validated.

Command:

```bash
sudo httpd -t
```

Successful output:

```text
Syntax OK
```

This is an important operational habit:

```text
Edit configuration
       ↓
Validate configuration
       ↓
Restart service
```

Instead of blindly restarting a service after modifying its configuration.

---

# 12. systemctl

`systemctl` is used to manage systemd services.

### Enable Apache

```bash
sudo systemctl enable httpd
```

This configures Apache to start automatically during boot.

### Restart Apache

```bash
sudo systemctl restart httpd
```

### Check status

```bash
sudo systemctl status httpd
```

### Quick active check

```bash
sudo systemctl is-active httpd
```

Expected:

```text
active
```

---

# 13. Verifying the Listening Port

The `ss` command can be used to inspect listening sockets.

```bash
sudo ss -lntp | grep 5002
```

Important options:

| Option | Meaning |
|---|---|
| `-l` | Listening sockets |
| `-n` | Numeric addresses/ports |
| `-t` | TCP sockets |
| `-p` | Show process information |

The result should show Apache listening on port `5002`.

---

# 14. Testing with curl

`curl` is extremely useful for testing HTTP services from the command line.

### Blog

```bash
curl http://localhost:5002/blog/
```

### Cluster

```bash
curl http://localhost:5002/cluster/
```

If Apache is correctly configured and the files are available, `curl` returns the website's HTML.

This provides a direct application-level verification.

---

# 15. Difference Between `ss` and `curl`

These two commands test different layers.

### `ss`

```bash
sudo ss -lntp | grep 5002
```

Answers:

> Is a process listening on TCP port 5002?

### `curl`

```bash
curl http://localhost:5002/blog/
```

Answers:

> Can I actually make an HTTP request and receive the website content?

Therefore both are useful:

```text
ss
 ↓
Port is listening

curl
 ↓
HTTP service is responding correctly
```

---

# 16. Final Architecture

The final setup looked like:

```text
                    stapp03
                       |
                       |
                 Apache httpd
                       |
                  TCP :5002
                       |
              /var/www/html/
                 /                         /                        blog          cluster
              |               |
       blog website     cluster website
```

The endpoints were:

```text
http://localhost:5002/blog/
http://localhost:5002/cluster/
```

---

# 17. Important DevOps Lessons

### Configuration management

Always know which configuration file controls the service.

For Apache:

```text
/etc/httpd/conf/httpd.conf
```

### Service validation

Use:

```bash
httpd -t
```

before restarting.

### Service management

Use:

```bash
systemctl
```

to manage services consistently.

### Network verification

Use:

```bash
ss
```

to verify listening ports.

### Application verification

Use:

```bash
curl
```

to verify that the application actually responds.

### Permissions

A web server needs appropriate filesystem permissions and, on SELinux-enabled systems, appropriate security contexts.

---

# 18. Day 19 Completion Summary

I successfully:

- Installed Apache HTTP Server.
- Configured Apache to listen on port `5002`.
- Created separate directories for the two static websites.
- Transferred the website backups from the jump host.
- Deployed the websites under Apache's document root.
- Configured ownership and permissions.
- Restored SELinux contexts.
- Validated the Apache configuration.
- Enabled and restarted the Apache service.
- Verified Apache was active.
- Verified both websites using `curl`.

---

## 🏆 Result

**Day 19 — PASSED ✅**

The final verification confirmed that both static websites were successfully served by Apache on Application Server 3.

```text
Blog:
http://localhost:5002/blog/

Cluster:
http://localhost:5002/cluster/
```

**100 Days of DevOps Progress: 19 / 100 🚀**
