# Day 43 — Docker Commands

## Environment

Server:

```text
Application Server 2 (stapp02)
```

User:

```text
steve
```

---

## 1. Pull the Required Image

```bash
docker pull nginx:alpine
```

Purpose:

Downloads the Nginx Alpine image from the configured Docker registry.

---

## 2. Create and Start the Container

```bash
docker run -d --name games -p 5000:80 nginx:alpine
```

### Command Breakdown

| Option | Meaning |
|---|---|
| `docker run` | Create and start a container |
| `-d` | Run in detached/background mode |
| `--name games` | Assign the container name `games` |
| `-p 5000:80` | Publish host port 5000 to container port 80 |
| `nginx:alpine` | Use the Nginx Alpine image |

---

## 3. Verify Running Containers

```bash
docker ps
```

Look for:

```text
games
```

and:

```text
0.0.0.0:5000->80/tcp
```

---

## 4. Verify All Containers

```bash
docker ps -a
```

Useful when checking whether a container exited unexpectedly.

---

## 5. Test Nginx

```bash
curl http://localhost:5000
```

Expected result:

Nginx's default HTML response.

---

## 6. Check Container Logs

```bash
docker logs games
```

Useful for troubleshooting Nginx startup and runtime issues.

---

## 7. Inspect the Container

```bash
docker inspect games
```

Useful for examining:

- Network configuration
- Port bindings
- Container state
- Mounts
- Environment
- Image information

---

## 8. Check the Image

```bash
docker images
```

Expected image:

```text
nginx
```

with tag:

```text
alpine
```

---

## 9. Stop the Container

```bash
docker stop games
```

---

## 10. Start the Existing Container Again

```bash
docker start games
```

---

## 11. Remove the Container

Only when no longer required:

```bash
docker rm -f games
```

---

## ⭐ Essential Day 43 Command Sequence

```bash
docker pull nginx:alpine
docker run -d --name games -p 5000:80 nginx:alpine
docker ps
curl http://localhost:5000
```

---

## 🔑 Most Important Syntax

```bash
docker run -d --name <container-name> -p <host-port>:<container-port> <image>:<tag>
```

Example:

```bash
docker run -d --name games -p 5000:80 nginx:alpine
```
