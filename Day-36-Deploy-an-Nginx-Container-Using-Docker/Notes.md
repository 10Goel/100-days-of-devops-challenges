# Day 36 — Docker Container Fundamentals

# 1. What Is Docker?

Docker is a platform used to package and run applications inside lightweight, isolated environments called **containers**.

Containers package an application together with its required dependencies, helping the application run consistently across development, testing, and production environments.

---

# 2. What Is a Docker Image?

A Docker image is a **read-only blueprint or template** used to create containers.

An image can contain:

- Application code
- Runtime
- Libraries
- Dependencies
- Configuration defaults
- Operating system components

Example:

```text
nginx:alpine
```

Images are commonly stored in container registries such as Docker Hub.

Useful commands:

```bash
docker pull nginx:alpine
docker images
```

---

# 3. What Is a Docker Container?

A Docker container is a **running instance of a Docker image**.

The relationship is:

```text
Docker Image
     |
     | docker run
     v
Docker Container
```

For Day 36:

```text
Image: nginx:alpine
Container: nginx_3
```

Multiple containers can be created from the same image.

---

# 4. Docker Image vs Container

| Image | Container |
|---|---|
| Blueprint/template | Running instance |
| Used to create containers | Created from an image |
| Immutable/read-only layers | Includes a writable container layer |
| Stored locally or in registries | Runs on a Docker host |
| Example: `nginx:alpine` | Example: `nginx_3` |

---

# 5. Docker Container Lifecycle

A container can move through several states:

```text
Image
  |
  v
Created
  |
  v
Running
  |
  +-----> Paused
  |          |
  |          v
  +----------+
  |
  v
Exited / Stopped
  |
  v
Removed
```

Common states:

- **Created** — container exists but is not running.
- **Running** — the main process is executing.
- **Paused** — processes are temporarily suspended.
- **Exited** — the main process has stopped.
- **Removed** — the container has been deleted.

---

# 6. The `docker run` Command

General syntax:

```bash
docker run [OPTIONS] IMAGE
```

Example:

```bash
docker run nginx:alpine
```

Docker generally:

1. Checks whether the image exists locally.
2. Pulls the image if it is missing.
3. Creates a container.
4. Configures the container.
5. Starts the container.

---

# 7. Detached Mode (`-d`)

The `-d` option runs a container in the background.

```bash
docker run -d nginx:alpine
```

Detached mode is commonly used for services such as:

- Web servers
- APIs
- Databases
- Application servers

Without `-d`, the terminal remains attached to the container process.

---

# 8. Naming Containers

Docker can automatically generate random names, but custom names make management easier.

```bash
docker run -d --name nginx_3 nginx:alpine
```

Benefits:

- Easier identification
- More readable commands
- Simpler automation
- No need to repeatedly use long container IDs

Example:

```bash
docker stop nginx_3
docker start nginx_3
docker logs nginx_3
```

---

# 9. Docker Image Tags

Images usually follow this format:

```text
repository:tag
```

Example:

```text
nginx:alpine
```

Where:

```text
nginx   = image repository
alpine  = image tag
```

Tags can identify:

- Versions
- Operating system variants
- Application releases
- Specific builds

Examples:

```text
nginx:latest
nginx:alpine
nginx:1.27
```

Using explicit tags makes deployments more predictable.

---

# 10. What Is Alpine?

Alpine Linux is a lightweight Linux distribution.

Alpine-based images are popular because they often provide:

- Smaller image sizes
- Faster downloads
- Reduced storage requirements
- Smaller dependency footprints

Day 36 used:

```text
nginx:alpine
```

---

# 11. Listing Containers

## Running containers

```bash
docker ps
```

Important output fields include:

- Container ID
- Image
- Command
- Created
- Status
- Ports
- Names

## All containers

```bash
docker ps -a
```

This includes stopped and exited containers.

---

# 12. Starting, Stopping, and Restarting

Stop:

```bash
docker stop nginx_3
```

Start:

```bash
docker start nginx_3
```

Restart:

```bash
docker restart nginx_3
```

A stopped container can usually be started again without creating a new container.

---

# 13. Container Logs

Containers commonly write application output to standard output and standard error.

View logs:

```bash
docker logs nginx_3
```

Follow logs in real time:

```bash
docker logs -f nginx_3
```

Logs are important for troubleshooting startup and runtime failures.

---

# 14. Inspecting Containers

Detailed metadata can be viewed with:

```bash
docker inspect nginx_3
```

This can include:

- Container state
- Image details
- Network configuration
- Mounts
- Environment variables
- IP addresses

Check only the running state:

```bash
docker inspect -f '{{.State.Running}}' nginx_3
```

Expected:

```text
true
```

---

# 15. Executing Commands Inside Containers

Use `docker exec` to run commands inside a running container.

Example:

```bash
docker exec nginx_3 ls
```

Interactive shell:

```bash
docker exec -it nginx_3 sh
```

Minimal Alpine containers commonly provide `sh` rather than `bash`.

Exit the container shell:

```bash
exit
```

---

# 16. Port Publishing

Containers have their own networking environment.

To expose a container service through the Docker host:

```bash
docker run -d -p 8080:80 --name nginx_3 nginx:alpine
```

Format:

```text
-p HOST_PORT:CONTAINER_PORT
```

Example:

```text
Host:8080
   |
   v
Container:80
```

---

# 17. Containers Are Ephemeral

Containers are designed to be disposable.

If a container is removed, its writable container layer is removed too.

```bash
docker rm nginx_3
```

Important data should therefore be stored using persistent storage such as:

- Docker volumes
- Bind mounts
- Databases
- External storage services

---

# 18. Docker Volumes

Volumes provide persistent storage managed by Docker.

Create a volume:

```bash
docker volume create app_data
```

Use it:

```bash
docker run -d -v app_data:/data nginx:alpine
```

The volume can survive even after the container is removed.

---

# 19. Bind Mounts

Bind mounts connect a host directory to a directory inside a container.

Example:

```bash
docker run -d -v /host/path:/container/path nginx:alpine
```

They are useful when containers need access to files on the host.

---

# 20. Basic Container Networking

Docker provides networking so containers can communicate.

Common network drivers include:

- Bridge
- Host
- None

List networks:

```bash
docker network ls
```

Standalone containers commonly use Docker's bridge networking.

---

# 21. Essential Docker Container Workflow

```text
1. Choose an Image
        |
        v
2. Pull Image
        |
        v
3. Create Container
        |
        v
4. Start Container
        |
        v
5. Verify Status
        |
        v
6. View Logs / Inspect
        |
        v
7. Stop or Restart
        |
        v
8. Remove When No Longer Needed
```

---

# 22. Essential Commands Summary

Pull an image:

```bash
docker pull nginx:alpine
```

List images:

```bash
docker images
```

Run a container:

```bash
docker run -d --name nginx_3 nginx:alpine
```

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Stop:

```bash
docker stop nginx_3
```

Start:

```bash
docker start nginx_3
```

Restart:

```bash
docker restart nginx_3
```

Logs:

```bash
docker logs nginx_3
```

Inspect:

```bash
docker inspect nginx_3
```

Execute a command:

```bash
docker exec nginx_3 ls
```

Remove:

```bash
docker rm nginx_3
```

---

# Key Takeaways

1. A Docker image is a blueprint; a container is its running instance.
2. `docker run` creates and starts containers.
3. Docker automatically pulls missing images.
4. `-d` runs containers in the background.
5. `--name` provides a human-readable identifier.
6. `docker ps` shows currently running containers.
7. `docker ps -a` shows all container states.
8. Image tags identify versions or variants.
9. Containers are designed to be disposable.
10. Persistent data should be stored outside the container layer.
11. `docker logs` is essential for troubleshooting.
12. `docker inspect` provides detailed metadata.
13. `docker exec` runs commands inside containers.
14. Port publishing connects host ports to container ports.
15. Understanding the container lifecycle is a core DevOps skill.

---

## Day 36 Summary

An Nginx container named **`nginx_3`** was successfully deployed from the **`nginx:alpine`** image and verified to be in the **running** state.

**Day 36 completed successfully. ✅**
