# DevOps Day 22 — Git Repository Cloning

## 📌 Challenge Overview

The Nautilus application development team required a copy of an existing Git repository on the **Storage Server** in the Stratos Datacenter.

The repository already existed at:

```bash
/opt/apps.git
```

The task was to clone it into:

```bash
/usr/src/kodekloudrepos
```

The operation had to be performed using the **natasha** user, without modifying the source repository or changing permissions on any existing directories.

---

## 🎯 Objectives

- Connect to the Storage Server as `natasha`
- Verify the source repository `/opt/apps.git`
- Clone the repository into `/usr/src/kodekloudrepos`
- Avoid unnecessary permission or ownership changes
- Validate that the repository was cloned successfully

---

## 🖥️ Environment

| Component | Value |
|---|---|
| Server | Storage Server |
| Host | `ststor01` |
| User | `natasha` |
| Source Repository | `/opt/apps.git` |
| Destination Parent Directory | `/usr/src/kodekloudrepos` |
| Final Clone Path | `/usr/src/kodekloudrepos/apps` |

---

## 🛠️ Solution

### 1. Connect to the Storage Server

From the jump host:

```bash
ssh natasha@ststor01
```

Confirm the active user:

```bash
whoami
```

Expected:

```text
natasha
```

### 2. Verify the Destination Directory

```bash
ls -ld /usr/src/kodekloudrepos
ls -la /usr/src/kodekloudrepos
```

This confirms that the required parent directory exists before cloning.

### 3. Verify the Source Repository

```bash
ls -ld /opt/apps.git
```

Because the repository is a bare Git repository, it can also be verified with:

```bash
git --git-dir=/opt/apps.git rev-parse --is-bare-repository
```

Expected output:

```text
true
```

### 4. Clone the Repository

Move to the required destination directory:

```bash
cd /usr/src/kodekloudrepos
```

Clone the repository:

```bash
git clone /opt/apps.git
```

Git automatically creates:

```text
/usr/src/kodekloudrepos/apps
```

### 5. Validate the Clone

```bash
cd /usr/src/kodekloudrepos/apps
git status
git remote -v
```

The `origin` remote should point to:

```text
/opt/apps.git
```

---

## ✅ Final State

The repository was successfully cloned from:

```text
/opt/apps.git
```

to:

```text
/usr/src/kodekloudrepos/apps
```

using the `natasha` user, with no unnecessary modifications to the original repository or existing directory permissions.

---

## 💡 Key Learning

This challenge demonstrates an important Git administration concept: a repository can be cloned directly from a local filesystem path just like a remote URL. When cloning `/opt/apps.git`, Git reads the source repository and creates a separate working copy under `/usr/src/kodekloudrepos/apps`.

It also reinforces an important DevOps practice: **make only the changes required by the task**. Commands such as `chmod`, `chown`, `git init`, or manually modifying the source repository were unnecessary and could have introduced configuration drift or caused validation failure.

---

## 🧰 Technologies Used

- Linux
- Git
- SSH
- Bash

---

## 📚 DevOps Concepts Practiced

- Git repository cloning
- Bare repositories
- Local-path Git repositories
- Linux filesystem navigation
- Repository verification
- Remote repository inspection
- Least-change operational practices

---

## 🏁 Result

**Day 22 challenge completed successfully.**
