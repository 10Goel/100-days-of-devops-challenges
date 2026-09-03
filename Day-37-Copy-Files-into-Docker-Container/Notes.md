# Day 37 — Docker File Management Fundamentals

# 1. Docker Container Filesystems

Docker containers have isolated filesystems. A file on the Docker host does not automatically exist inside a container.

Example:

```text
Docker Host:       /tmp/nautilus.txt.gpg
Docker Container:  /usr/src/
```

Docker provides multiple methods for managing files and data:

- `docker cp`
- Docker volumes
- Bind mounts

---

# 2. The `docker cp` Command

`docker cp` copies files and directories between a Docker host and a container.

## Host → Container

```bash
docker cp SOURCE CONTAINER:DESTINATION
```

Example:

```bash
docker cp file.txt my_container:/tmp/
```

## Container → Host

```bash
docker cp CONTAINER:SOURCE DESTINATION
```

Example:

```bash
docker cp my_container:/tmp/file.txt .
```

---

# 3. Container Path Syntax

Container paths use a colon:

```text
container_name:/path/inside/container
```

Example:

```text
ubuntu_latest:/usr/src/
```

The colon separates the container name from its internal filesystem path.

---

# 4. Verifying Files Inside Containers

Use `docker exec` to execute commands inside a running container.

```bash
docker exec ubuntu_latest ls -l /usr/src/
```

Verify a specific file:

```bash
docker exec ubuntu_latest ls -l /usr/src/nautilus.txt.gpg
```

---

# 5. Understanding `docker exec`

General syntax:

```bash
docker exec [OPTIONS] CONTAINER COMMAND
```

Example:

```bash
docker exec ubuntu_latest ls
```

Interactive shell:

```bash
docker exec -it ubuntu_latest bash
```

If Bash is unavailable:

```bash
docker exec -it ubuntu_latest sh
```

---

# 6. File Integrity and Checksums

File integrity means ensuring that data remains unchanged during transfer or storage.

SHA-256 can be used for verification:

```bash
sha256sum /tmp/nautilus.txt.gpg
```

Inside the container:

```bash
docker exec ubuntu_latest sha256sum /usr/src/nautilus.txt.gpg
```

If both hashes match, the file contents are identical.

Checksums help detect:

- Corruption
- Accidental modification
- Incomplete transfers
- Unexpected changes

---

# 7. `docker cp` vs Docker Volumes

## `docker cp`

- One-time manual transfer
- Useful for copying individual files
- Does not automatically synchronize changes

Example:

```bash
docker cp file.txt container:/data/
```

## Docker Volumes

Volumes provide persistent Docker-managed storage.

```bash
docker volume create app_data
```

Use a volume:

```bash
docker run -v app_data:/data nginx
```

Volumes are generally better for persistent application data.

---

# 8. Bind Mounts

A bind mount maps a host directory directly into a container.

```bash
docker run -v /host/data:/container/data image
```

Unlike `docker cp`, changes can be immediately visible through the shared mapping.

| `docker cp` | Bind Mount |
|---|---|
| One-time copy | Shared filesystem mapping |
| Manual operation | Continuous access |
| Useful for transfer | Useful for shared files |

---

# 9. Container Data Is Often Ephemeral

A container has image layers plus a writable container layer.

```text
Image Layers
     +
Writable Layer
     =
Container Filesystem
```

When a container is removed, its writable layer can be lost.

Persistent data should therefore usually use:

- Docker volumes
- Bind mounts
- Databases
- External storage

---

# 10. Security Considerations

When handling confidential or encrypted files:

- Do not modify the original unnecessarily.
- Do not decrypt files unless required.
- Verify the destination path carefully.
- Use checksums for important transfers.
- Avoid exposing sensitive contents in logs.

For Day 37, the encrypted `.gpg` file was copied directly without modification.

---

# 11. Essential Commands

Copy host → container:

```bash
docker cp file.txt container:/tmp/
```

Copy container → host:

```bash
docker cp container:/tmp/file.txt .
```

List files inside a container:

```bash
docker exec container ls -l
```

Inspect file metadata:

```bash
docker exec container stat /path/to/file
```

Calculate SHA-256:

```bash
sha256sum file.txt
```

Calculate SHA-256 inside a container:

```bash
docker exec container sha256sum /path/to/file
```

---

# 🧠 Key Takeaways

1. Containers have isolated filesystems.
2. `docker cp` transfers files between host and containers.
3. `docker exec` runs commands inside running containers.
4. SHA-256 verifies file integrity.
5. `docker cp` is useful for one-time transfers.
6. Volumes are better for persistent Docker data.
7. Bind mounts provide direct host-container file access.
8. Sensitive files should be handled without unnecessary modification.

## 🏁 Day 37 Summary

The encrypted file `/tmp/nautilus.txt.gpg` was successfully copied into:

```text
ubuntu_latest:/usr/src/nautilus.txt.gpg
```

**Day 37 completed successfully. ✅**
