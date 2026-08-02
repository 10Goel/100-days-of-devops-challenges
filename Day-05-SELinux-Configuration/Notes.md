# 📘 Day 5 Notes - SELinux

## What is SELinux?

SELinux (Security Enhanced Linux) is a Linux security framework developed by the NSA.

It provides Mandatory Access Control (MAC) to improve system security.

Unlike traditional Linux permissions, SELinux controls what processes can access.

---

# Linux Permission vs SELinux

Traditional Linux

Owner → Group → Others

Uses:

- rwx permissions

Example:

-rwxr-xr-x

---

SELinux

Controls:

- Files
- Processes
- Ports
- Services

Even if a file has 777 permissions, SELinux can still deny access.

---

# SELinux Modes

## Enforcing

Security policies are actively enforced.

Commands violating policy are blocked.

```text
getenforce

Enforcing
```

---

## Permissive

Policies are not enforced.

Violations are logged only.

Useful for testing.

```text
getenforce

Permissive
```

---

## Disabled

SELinux is completely disabled.

```text
getenforce

Disabled
```

---

# Useful Commands

Check current mode

```bash
getenforce
```

Detailed status

```bash
sestatus
```

View configuration

```bash
cat /etc/selinux/config
```

---

# Important Configuration File

```
/etc/selinux/config
```

Example

```text
SELINUX=disabled
SELINUXTYPE=targeted
```

---

# Installing SELinux Packages

CentOS / RHEL

```bash
dnf install -y selinux-policy selinux-policy-targeted
```

---

# Runtime vs Permanent Changes

Runtime

Changes take effect immediately.

Example

```bash
setenforce 0
```

Permanent

Modify

```
/etc/selinux/config
```

Changes apply after reboot.

---

# Important Interview Questions

### What is SELinux?

A Linux kernel security module providing Mandatory Access Control (MAC).

---

### Difference between DAC and MAC?

DAC

Owner controls permissions.

MAC

System security policy controls permissions.

---

### Difference between chmod and SELinux?

chmod

Controls Linux file permissions.

SELinux

Controls access based on security policy.

---

### Difference between setenforce and editing config?

setenforce

Temporary until reboot.

Editing `/etc/selinux/config`

Permanent after reboot.

---

### Common Commands

```bash
getenforce
```

```bash
sestatus
```

```bash
setenforce 0
```

```bash
setenforce 1
```

```bash
restorecon
```

```bash
chcon
```

---

# What I Learned

- Understanding SELinux fundamentals.
- Difference between Linux permissions and SELinux.
- Checking SELinux status.
- Installing SELinux policy packages.
- Editing the SELinux configuration file.
- Difference between temporary and permanent configuration changes.
- Verifying SELinux settings using Linux commands.
