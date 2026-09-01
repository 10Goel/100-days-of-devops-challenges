# Day 35 — Commands

## 🎯 Objective

Install Docker CE and Docker Compose on **Application Server 3 (`stapp03`)**, then start the Docker service.

---

## 1. Connect to Application Server 3

From the jump host:

```bash
ssh banner@stapp03
```

---

## 2. Switch to Root

```bash
sudo -i
```

Verify:

```bash
whoami
```

Expected:

```text
root
```

---

## 3. Install Repository Utilities

```bash
yum install -y yum-utils
```

---

## 4. Add Docker CE Repository

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Refresh package metadata:

```bash
yum makecache
```

---

## 5. Install Docker CE

```bash
yum install -y docker-ce docker-ce-cli containerd.io
```

Verify:

```bash
docker --version
```

---

## 6. Install Docker Compose V2

```bash
yum install -y docker-compose-plugin
```

Verify:

```bash
docker compose version
```

> Use `docker compose`, not `docker-compose`, for Compose V2.

---

## 7. Start Docker

```bash
systemctl start docker
```

---

## 8. Enable Docker at Boot

```bash
systemctl enable docker
```

---

## 9. Verify Docker Service

Quick verification:

```bash
systemctl is-active docker
```

Expected:

```text
active
```

Detailed verification:

```bash
systemctl status docker
```

Press `q` to exit the status screen.

---

## 10. Final Verification

```bash
docker --version
```

```bash
docker compose version
```

```bash
systemctl is-active docker
```

Expected result:

```text
Docker version 29.7.2, build ...
Docker Compose version v2.x.x
active
```

---

## ⚠️ Troubleshooting Command

The legacy Compose package was unavailable:

```bash
yum install -y docker-compose
```

If this produces:

```text
No match for argument: docker-compose
```

use:

```bash
yum install -y docker-compose-plugin
```

Then:

```bash
docker compose version
```

---

## 📋 Complete Command Sequence

```bash
ssh banner@stapp03
sudo -i
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache
yum install -y docker-ce docker-ce-cli containerd.io
yum install -y docker-compose-plugin
systemctl start docker
systemctl enable docker
systemctl is-active docker
docker --version
docker compose version
```

---

## ✅ Completion

**KodeKloud Day 35 — Successfully Completed.**
