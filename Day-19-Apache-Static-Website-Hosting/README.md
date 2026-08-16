# 🚀 Day 19 — Apache Static Website Hosting

## 📌 Challenge Overview

**100 Days of DevOps — Day 19**

In this challenge, I prepared **Application Server 3 (`stapp03`)** in the Stratos Datacenter to host two static websites using **Apache HTTP Server (`httpd`)**.

The objective was to install and configure Apache, change its listening port to **5002**, deploy the provided website backups, and verify that both websites were accessible locally through Apache.

---

## 🎯 Objectives

The task required me to:

- Install the `httpd` package and its dependencies on Application Server 3.
- Configure Apache to listen on **TCP port 5002**.
- Deploy the `blog` website backup.
- Deploy the `cluster` website backup.
- Configure both websites under Apache's document root.
- Verify both websites using `curl`.
- Confirm that Apache was running successfully.

---

## 🏗️ Infrastructure

| Component | Details |
|---|---|
| Datacenter | Stratos Datacenter |
| Server | Application Server 3 |
| Hostname | `stapp03` |
| SSH User | `banner` |
| Web Server | Apache HTTP Server (`httpd`) |
| Apache Port | `5002` |
| Blog URL | `http://localhost:5002/blog/` |
| Cluster URL | `http://localhost:5002/cluster/` |

### Source Website Backups

The website backups were available on the jump host:

```text
/home/thor/blog
/home/thor/cluster
```

### Apache Deployment Paths

```text
/var/www/html/blog
/var/www/html/cluster
```

---

## 🔧 Implementation

### 1. Connected to Application Server 3

```bash
ssh banner@stapp03
```

Verified the server identity:

```bash
whoami
hostname
```

Expected:

```text
banner
stapp03
```

### 2. Installed Apache

```bash
sudo yum install -y httpd
```

### 3. Changed Apache Port

Apache was configured to listen on port `5002` instead of the default HTTP port `80`.

```bash
sudo sed -i 's/^Listen 80$/Listen 5002/' /etc/httpd/conf/httpd.conf
```

Verified:

```bash
grep -n "^Listen" /etc/httpd/conf/httpd.conf
```

### 4. Created Website Directories

```bash
sudo mkdir -p /var/www/html/blog
sudo mkdir -p /var/www/html/cluster
```

### 5. Transferred Website Backups

From the jump host:

```bash
scp -r /home/thor/blog banner@stapp03:/tmp/
scp -r /home/thor/cluster banner@stapp03:/tmp/
```

### 6. Deployed Website Files

On `stapp03`:

```bash
sudo cp -r /tmp/blog/* /var/www/html/blog
sudo cp -r /tmp/cluster/* /var/www/html/cluster
```

### 7. Configured Permissions

```bash
sudo chown -R apache:apache /var/www/html/blog
sudo chown -R apache:apache /var/www/html/cluster

sudo chmod -R 755 /var/www/html/blog
sudo chmod -R 755 /var/www/html/cluster
```

### 8. Restored SELinux Contexts

```bash
sudo restorecon -Rv /var/www/html/
```

### 9. Validated Apache Configuration

```bash
sudo httpd -t
```

Expected:

```text
Syntax OK
```

### 10. Enabled and Restarted Apache

```bash
sudo systemctl enable httpd
sudo systemctl restart httpd
```

Verified the service:

```bash
sudo systemctl is-active httpd
```

Expected:

```text
active
```

---

## 🧪 Verification

### Blog Website

```bash
curl http://localhost:5002/blog/
```

The response confirmed that the blog website was being served successfully.

### Cluster Website

```bash
curl http://localhost:5002/cluster/
```

The response confirmed that the cluster website was being served successfully.

### Apache Service

```bash
sudo systemctl is-active httpd
```

Result:

```text
active
```

### Apache Version

```bash
sudo httpd -v
```

The server was running Apache HTTP Server on CentOS Stream.

---

## 🧠 Key Learning Outcomes

This challenge strengthened my understanding of:

- Installing and managing Apache on Linux.
- Apache configuration files.
- Changing the HTTP listening port.
- Linux web-server document roots.
- Deploying static website files.
- Linux ownership and permissions.
- SELinux file contexts.
- `systemctl` service management.
- Validating Apache configuration before restarting.
- Testing web applications using `curl`.
- Verifying listening ports using `ss`.

---

## 📂 Final Directory Structure

```text
/var/www/html/
├── blog/
│   └── website files
│
└── cluster/
    └── website files
```

Apache served these directories through:

```text
http://localhost:5002/blog/
http://localhost:5002/cluster/
```

---

## ✅ Final Status

**Day 19 — Successfully Completed 🎉**

The Apache server on `stapp03` was configured to listen on port `5002`, both static websites were deployed, and both endpoints were successfully verified using `curl`.

> **Challenge Status: PASSED ✅**

---

## 📚 Technologies Used

- Linux
- Apache HTTP Server (`httpd`)
- Bash
- SSH
- SCP
- systemd
- SELinux
- curl
- ss

---

## 👨‍💻 DevOps Journey

**100 Days of DevOps — Day 19 / 100**

> Learn → Configure → Verify → Document → Repeat 🚀
