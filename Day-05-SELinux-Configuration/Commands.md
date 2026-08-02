# Day 5 Commands

## Connect to Server

```bash
ssh tony@stapp01
```

Become root

```bash
sudo su -
```

---

## Check Operating System

```bash
cat /etc/os-release
```

---

## Check SELinux Status

```bash
getenforce
```

```bash
sestatus
```

---

## Check Installed Packages

```bash
rpm -qa | grep selinux
```

```bash
rpm -q policycoreutils
```

```bash
rpm -q selinux-policy
```

```bash
rpm -q selinux-policy-targeted
```

---

## Install Required Packages

```bash
dnf install -y selinux-policy selinux-policy-targeted
```

---

## Edit Configuration

```bash
nano /etc/selinux/config
```

or

```bash
vi /etc/selinux/config
```

---

## Verify Configuration

```bash
cat /etc/selinux/config
```

---

## Verify Runtime Status

```bash
getenforce
```

```bash
sestatus
```
