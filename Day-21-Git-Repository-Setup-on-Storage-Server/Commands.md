# Day 21 – Commands Reference

## 1. Connect to Storage Server

```bash
ssh natasha@ststor01
```

---

## 2. Verify Host

```bash
hostname
```

Expected:

```text
ststor01
```

---

## 3. Install Git

```bash
sudo yum install git -y
```

---

## 4. Verify Git Installation

```bash
git --version
```

---

## 5. Create Bare Git Repository

```bash
sudo git init --bare /opt/ecommerce.git
```

---

## 6. Verify Bare Repository

```bash
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
```

Expected:

```text
true
```

---

## 7. Inspect Repository Contents

```bash
sudo ls -la /opt/ecommerce.git
```

Typical entries include:

```text
HEAD
config
description
hooks/
info/
objects/
refs/
```

---

## 8. Check Repository Type

```bash
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
```

Expected:

```text
true
```

---

## 9. Complete Command Sequence

From the Jump Host:

```bash
ssh natasha@ststor01
```

On the Storage Server:

```bash
sudo yum install git -y
git --version
sudo git init --bare /opt/ecommerce.git
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
sudo ls -la /opt/ecommerce.git
```

---

## 10. Quick Reference

| Purpose | Command |
|---|---|
| SSH to Storage Server | `ssh natasha@ststor01` |
| Install Git | `sudo yum install git -y` |
| Check Git version | `git --version` |
| Create bare repository | `sudo git init --bare /opt/ecommerce.git` |
| Verify bare repository | `sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository` |
| List repository files | `sudo ls -la /opt/ecommerce.git` |

---

## ⚠️ Important

For this challenge, the repository must be:

```text
/opt/ecommerce.git
```

and it must be a **bare** repository.

Correct:

```bash
sudo git init --bare /opt/ecommerce.git
```

Not:

```bash
git init /opt/ecommerce.git
```

---

**Day 21 Status:** ✅ Completed
