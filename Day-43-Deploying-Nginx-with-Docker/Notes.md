# Day 43 — Docker & Nginx Notes

## 1. Docker Images vs Containers

A **Docker image** is a read-only template containing the files, libraries, configuration, and metadata required to run an application.

A **container** is a running or stopped instance created from an image.

```text
Docker Image
     ↓
docker run
     ↓
Container
     ↓
Application Process
```

For this task:

```text
Image     → nginx:alpine
Container → games
```

---

# 2. Docker Image Tags

Docker images commonly use tags to identify variants or versions.

Example:

```text
nginx:alpine
```

Breakdown:

```text
nginx  → Image repository
alpine → Image tag
```

The `alpine` variant is designed to be relatively lightweight.

### Best Practice

For reproducible production deployments, consider using a specific version tag or image digest rather than relying indefinitely on a moving tag.

---

# 3. `docker pull`

The command:

```bash
docker pull nginx:alpine
```

downloads the specified image if it is not already available locally.

General syntax:

```bash
docker pull <image>:<tag>
```

Examples:

```bash
docker pull nginx:alpine
docker pull nginx:1.29-alpine
```

---

# 4. `docker run`

`docker run` creates a new container from an image and starts it.

General syntax:

```bash
docker run [OPTIONS] IMAGE
```

Example:

```bash
docker run -d --name games -p 5000:80 nginx:alpine
```

---

# 5. Detached Mode

The `-d` option means:

```text
detached mode
```

The container runs in the background and the terminal is returned to the shell.

Without `-d`, the container's foreground process can occupy the terminal.

---

# 6. Container Naming

The option:

```bash
--name games
```

assigns a predictable name to the container.

Instead of referring to an automatically generated container name, we can use:

```bash
docker ps
docker logs games
docker inspect games
docker stop games
```

---

# 7. Docker Port Publishing

One of the most important concepts from this task is:

```bash
-p 5000:80
```

The format is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
Host port       Container port
    5000   →         80
```

A request to:

```text
http://localhost:5000
```

is forwarded to port `80` inside the container.

---

# 8. Why Nginx Uses Port 80

Nginx is commonly configured to listen for HTTP traffic on:

```text
TCP/80
```

Inside the container, Nginx serves its default HTTP site through port 80.

The host does not have to expose the same port.

For example:

```text
Host 5000 → Container 80
Host 8080 → Container 80
Host 80   → Container 80
```

All are possible as long as the host port is available.

---

# 9. Port Mapping Is Not the Same as Exposing a Port

Docker terminology can be confusing.

### `EXPOSE`

An image may contain metadata such as:

```dockerfile
EXPOSE 80
```

This documents the port the application expects to use.

### `-p`

Publishing with:

```bash
-p 5000:80
```

actually creates a host-to-container port mapping.

So:

```text
EXPOSE 80
```

does not by itself make the service accessible on the host.

---

# 10. Container Networking Basics

When Docker creates a container, it normally connects it to a Docker network.

A port publishing rule allows traffic from outside the container network to reach the application.

Conceptually:

```text
External Client
      |
      v
Host :5000
      |
      v
Docker Port Mapping
      |
      v
Container :80
      |
      v
Nginx
```

---

# 11. Verify Container State

Use:

```bash
docker ps
```

This displays running containers.

For stopped containers too:

```bash
docker ps -a
```

Important information includes:

- Container ID
- Image
- Command
- Creation time
- Status
- Published ports
- Container name

---

# 12. Testing the Application

The command:

```bash
curl http://localhost:5000
```

is a simple way to verify HTTP connectivity.

If Nginx is functioning correctly, the command returns the default Nginx HTML response.

This is useful because it tests more than just whether the container exists—it checks whether the service is reachable through the published port.

---

# 13. Container Lifecycle

A basic Docker container lifecycle is:

```text
Image
  ↓
docker run
  ↓
Created + Started
  ↓
Running
  ↓
docker stop
  ↓
Stopped
  ↓
docker start
  ↓
Running again
  ↓
docker rm
  ↓
Removed
```

Useful commands:

```bash
docker ps
docker stop games
docker start games
docker restart games
docker rm games
```

---

# 14. Container vs Process

A container is not equivalent to a full virtual machine.

A container generally isolates processes using Linux kernel features while sharing the host kernel.

For the Nginx container:

```text
Container
   |
   └── Nginx master/worker processes
```

If the main container process exits, Docker considers the container stopped.

---

# 15. Why the Nginx Container Keeps Running

The container was started in detached mode:

```bash
docker run -d ...
```

But detached mode alone does **not** keep a container alive.

The container remains running because Nginx provides an active foreground process inside the container.

General principle:

```text
Main container process running
        ↓
Container stays running

Main container process exits
        ↓
Container stops
```

---

# 16. Useful Troubleshooting Commands

### Check running containers

```bash
docker ps
```

### Check all containers

```bash
docker ps -a
```

### View logs

```bash
docker logs games
```

### Follow logs

```bash
docker logs -f games
```

### Inspect configuration

```bash
docker inspect games
```

### Check port mapping

```bash
docker port games
```

Expected:

```text
80/tcp -> 0.0.0.0:5000
```

### Test HTTP

```bash
curl http://localhost:5000
```

---

# 17. Common Mistakes

## Mistake 1 — Reversing the port mapping

Incorrect:

```bash
-p 80:5000
```

For this task, the correct mapping is:

```bash
-p 5000:80
```

Remember:

```text
HOST:CONTAINER
```

---

## Mistake 2 — Forgetting detached mode

Without:

```bash
-d
```

the container may remain attached to the terminal.

---

## Mistake 3 — Using the wrong image

Required:

```text
nginx:alpine
```

Not merely:

```text
nginx
```

when the task specifically requests the Alpine variant.

---

## Mistake 4 — Wrong container name

Required:

```text
games
```

Verify using:

```bash
docker ps
```

---

# 18. DevOps Takeaways

This challenge demonstrates a basic but important container deployment workflow:

```text
Pull Image
    ↓
Create Container
    ↓
Configure Port Mapping
    ↓
Start Container
    ↓
Verify Container
    ↓
Test Application
```

This workflow forms the foundation for more advanced topics such as:

- Dockerfiles
- Docker networks
- Docker Compose
- Reverse proxies
- Containerized microservices
- Kubernetes Deployments
- Kubernetes Services
- Ingress
- CI/CD container deployments

---

# 19. Quick Revision

### Pull image

```bash
docker pull nginx:alpine
```

### Run container

```bash
docker run -d --name games -p 5000:80 nginx:alpine
```

### Verify

```bash
docker ps
```

### Test

```bash
curl http://localhost:5000
```

### Port syntax

```text
HOST_PORT:CONTAINER_PORT
```

### This task

```text
5000:80
```

---

## ✅ Day 43 Summary

**Primary concepts:**

- Docker images
- Image tags
- Nginx Alpine
- Docker containers
- `docker pull`
- `docker run`
- Detached mode
- Container naming
- Port publishing
- Host-to-container networking
- Container lifecycle
- Service verification
- Basic troubleshooting

**Status: ✅ Day 43 Completed Successfully**
