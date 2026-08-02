# 🛡️ Day 5 - SELinux Configuration

## 🎯 Objective

Configure SELinux on App Server 1 by:

- Installing the required SELinux packages.
- Permanently disabling SELinux in the configuration file.
- Completing the task without rebooting the server.

---

## 📖 Problem Statement

The security team wanted SELinux configured on App Server 1.

Requirements:

- Install required SELinux packages.
- Permanently disable SELinux.
- Do not reboot the server.
- The final configuration after reboot should remain disabled.

---

## 🖥️ Server Details

| Server | Hostname |
|---------|----------|
| App Server 1 | stapp01 |

---

## 🔧 Solution

### 1. Connect to App Server

```bash
ssh tony@stapp01
sudo su -
```

### 2. Verify Current SELinux Status

```bash
getenforce
```

or

```bash
sestatus
```

---

### 3. Install Required Packages

```bash
dnf install -y selinux-policy selinux-policy-targeted
```

---

### 4. Edit SELinux Configuration

```bash
nano /etc/selinux/config
```

Change

```text
SELINUX=enforcing
```

to

```text
SELINUX=disabled
```

Save the file.

---

### 5. Verify Configuration

```bash
cat /etc/selinux/config
```

Expected output:

```text
SELINUX=disabled
```

---

## ✅ Result

Successfully configured SELinux according to the task requirements.
