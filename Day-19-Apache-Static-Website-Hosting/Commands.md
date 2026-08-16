# 🛠️ Day 19 — Commands Reference

This document contains the important commands used to complete the **Apache Static Website Hosting** challenge.

---

## 1. Connect to Application Server 3

```bash
ssh banner@stapp03
```

Verify:

```bash
whoami
hostname
```

Expected:

```text
banner
stapp03
```

---

## 2. Install Apache

```bash
sudo yum install -y httpd
```

Check version:

```bash
httpd -v
```

---

## 3. Configure Apache Port

Change Apache from port `80` to `5002`:

```bash
sudo sed -i 's/^Listen 80$/Listen 5002/' /etc/httpd/conf/httpd.conf
```

Verify configuration:

```bash
grep -n "^Listen" /etc/httpd/conf/httpd.conf
```

Expected:

```text
Listen 5002
```

---

## 4. Create Website Directories

```bash
sudo mkdir -p /var/www/html/blog
sudo mkdir -p /var/www/html/cluster
```

Verify:

```bash
ls -ld /var/www/html/blog
ls -ld /var/www/html/cluster
```

---

## 5. Copy Website Backups from Jump Host

Run these commands from the jump host:

```bash
scp -r /home/thor/blog banner@stapp03:/tmp/
scp -r /home/thor/cluster banner@stapp03:/tmp/
```

---

## 6. Deploy Website Files

On `stapp03`:

```bash
sudo cp -r /tmp/blog/* /var/www/html/blog
sudo cp -r /tmp/cluster/* /var/www/html/cluster
```

Verify:

```bash
ls -la /var/www/html/blog
ls -la /var/www/html/cluster
```

---

## 7. Set Ownership

```bash
sudo chown -R apache:apache /var/www/html/blog
sudo chown -R apache:apache /var/www/html/cluster
```

---

## 8. Set Permissions

```bash
sudo chmod -R 755 /var/www/html/blog
sudo chmod -R 755 /var/www/html/cluster
```

---

## 9. Restore SELinux Context

```bash
sudo restorecon -Rv /var/www/html/
```

---

## 10. Test Apache Configuration

```bash
sudo httpd -t
```

Expected:

```text
Syntax OK
```

---

## 11. Enable Apache at Boot

```bash
sudo systemctl enable httpd
```

---

## 12. Restart Apache

```bash
sudo systemctl restart httpd
```

---

## 13. Check Apache Status

```bash
sudo systemctl status httpd
```

Or:

```bash
sudo systemctl is-active httpd
```

Expected:

```text
active
```

---

## 14. Verify Listening Port

```bash
sudo ss -lntp | grep 5002
```

Expected to show Apache listening on:

```text
0.0.0.0:5002
```

---

## 15. Test Blog Website

```bash
curl http://localhost:5002/blog/
```

---

## 16. Test Cluster Website

```bash
curl http://localhost:5002/cluster/
```

---

## 17. Complete Verification

```bash
sudo httpd -t
sudo systemctl is-active httpd
sudo ss -lntp | grep 5002
curl http://localhost:5002/blog/
curl http://localhost:5002/cluster/
```

---

## 🔑 Command Categories

| Purpose | Commands |
|---|---|
| Remote access | `ssh` |
| File transfer | `scp` |
| Package installation | `yum` |
| Apache management | `httpd` |
| Service management | `systemctl` |
| Port verification | `ss` |
| Website testing | `curl` |
| File permissions | `chmod` |
| File ownership | `chown` |
| SELinux context | `restorecon` |
| Configuration editing | `sed`, `vi` |

---

## ✅ Final Verification

The following checks confirmed successful completion:

```bash
sudo httpd -t
```

```text
Syntax OK
```

```bash
sudo systemctl is-active httpd
```

```text
active
```

```bash
curl http://localhost:5002/blog/
```

```bash
curl http://localhost:5002/cluster/
```

Both websites returned their expected HTML content.

---

**Day 19 Status: PASSED ✅**
