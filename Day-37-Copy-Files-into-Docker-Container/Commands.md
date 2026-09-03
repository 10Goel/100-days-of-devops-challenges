# Day 37 — Commands

## 🎯 Objective

Copy:

```text
/tmp/nautilus.txt.gpg
```

into the running Docker container:

```text
ubuntu_latest
```

at:

```text
/usr/src/
```

without modifying the file.

---

## 1. Connect to Application Server 3

```bash
ssh banner@stapp03
```

## 2. Verify the container

```bash
docker ps
```

## 3. Verify the source file

```bash
ls -l /tmp/nautilus.txt.gpg
```

## 4. Copy the file

```bash
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
```

## 5. Verify inside the container

```bash
docker exec ubuntu_latest ls -l /usr/src/nautilus.txt.gpg
```

## 6. Verify file integrity

Host:

```bash
sha256sum /tmp/nautilus.txt.gpg
```

Container:

```bash
docker exec ubuntu_latest sha256sum /usr/src/nautilus.txt.gpg
```

The hashes should match.

---

## 📋 Complete Command Sequence

```bash
ssh banner@stapp03
docker ps
ls -l /tmp/nautilus.txt.gpg
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
docker exec ubuntu_latest ls -l /usr/src/nautilus.txt.gpg
sha256sum /tmp/nautilus.txt.gpg
docker exec ubuntu_latest sha256sum /usr/src/nautilus.txt.gpg
```

---

## 🔧 Useful Docker File Commands

Copy host → container:

```bash
docker cp source_file container:/destination/
```

Copy container → host:

```bash
docker cp container:/source/file /destination/
```

Execute a command inside a container:

```bash
docker exec container command
```

Open a shell:

```bash
docker exec -it container bash
```

If Bash is unavailable:

```bash
docker exec -it container sh
```

## ✅ Completion

**KodeKloud Day 37 — Successfully Completed.**
