# Day 37 — Copy Files into a Docker Container

## 📌 Challenge Overview

Day 37 of the **KodeKloud 100 Days of DevOps Challenge** focused on securely copying a file from a Docker host into a Docker container.

The task required copying an encrypted file from **Application Server 3** into an existing container without modifying the file.

## 🎯 Task Requirements

- **Server:** Application Server 3 (`stapp03`)
- **Container:** `ubuntu_latest`
- **Source:** `/tmp/nautilus.txt.gpg`
- **Destination:** `/usr/src/`
- **Requirement:** File must not be modified

## 🚀 Implementation

### 1. Connect to Application Server 3

```bash
ssh banner@stapp03
```

### 2. Verify the container

```bash
docker ps
```

### 3. Verify the source file

```bash
ls -l /tmp/nautilus.txt.gpg
```

### 4. Copy the encrypted file

```bash
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
```

### 5. Verify the copied file

```bash
docker exec ubuntu_latest ls -l /usr/src/nautilus.txt.gpg
```

## 🔍 Command Breakdown

```bash
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
```

| Component | Purpose |
|---|---|
| `docker cp` | Copies files between host and container |
| `/tmp/nautilus.txt.gpg` | Source file |
| `ubuntu_latest` | Target container |
| `/usr/src/` | Destination directory |

## 🔐 File Integrity

Because the file was encrypted and confidential, it was copied directly without decryption or modification.

Integrity can be verified using SHA-256:

```bash
sha256sum /tmp/nautilus.txt.gpg
docker exec ubuntu_latest sha256sum /usr/src/nautilus.txt.gpg
```

Matching hashes confirm identical file contents.

## 🧠 Key Learnings

- `docker cp` transfers files between Docker hosts and containers.
- `docker exec` executes commands inside running containers.
- Checksums help verify file integrity.
- Container paths use `container_name:/path`.
- Confidential files should not be unnecessarily modified.

## ✅ Result

**Day 37 completed successfully.**

The encrypted file was successfully copied:

```text
/tmp/nautilus.txt.gpg
        ↓
ubuntu_latest:/usr/src/nautilus.txt.gpg
```

**Progress: Day 37 / 100 — Completed Successfully ✅**
