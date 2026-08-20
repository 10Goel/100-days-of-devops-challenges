# Day 21 – Git Repository Setup on Storage Server

## 📌 Challenge Overview

As part of the Nautilus development project, the DevOps team was tasked with establishing a Git repository on the Storage Server in the Stratos Data Center.

The objective was to install Git on the Storage Server and create a **bare Git repository** at the exact path `/opt/ecommerce.git`.

## 🎯 Objectives

- Access the Storage Server through the Jump Host.
- Install the Git package using `yum`.
- Create a bare Git repository named `ecommerce.git`.
- Verify that the repository was initialized correctly.

## 🖥️ Infrastructure

| Server | Hostname | User | Purpose |
|---|---|---|---|
| Jump Host | `jump-host` | `thor` | Secure access to infrastructure |
| Storage Server | `ststor01` | `natasha` | Stores Nautilus server data |

## 🛠️ Implementation

### 1. Connect to the Storage Server

From the Jump Host:

```bash
ssh natasha@ststor01
```

### 2. Install Git

```bash
sudo yum install git -y
```

Verify:

```bash
git --version
```

### 3. Create the Bare Repository

```bash
sudo git init --bare /opt/ecommerce.git
```

### 4. Verify the Repository

```bash
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
```

Expected output:

```text
true
```

The repository contents can also be inspected with:

```bash
sudo ls -la /opt/ecommerce.git
```

## 🧠 Key Learning

A **bare Git repository** contains Git's version-control data but does not contain a working tree. It is commonly used as a centralized repository on a server where developers push and pull code.

The important distinction is:

```bash
git init /path/to/repository
```

creates a normal repository, while:

```bash
git init --bare /path/to/repository.git
```

creates a bare repository.

## ✅ Result

The Day 21 challenge was completed successfully. Git was installed on the Storage Server and the required bare repository was created at:

```text
/opt/ecommerce.git
```

## 🔑 Skills Practiced

- Linux SSH
- Package installation with `yum`
- Git installation and verification
- Bare Git repositories
- Server-side repository setup
- Linux filesystem verification
- Basic Git administration

## 📚 Commands Summary

```bash
ssh natasha@ststor01
sudo yum install git -y
git --version
sudo git init --bare /opt/ecommerce.git
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
sudo ls -la /opt/ecommerce.git
```

---

**Challenge:** DevOps Day 21  
**Topic:** Git Repository Setup  
**Status:** ✅ Completed
