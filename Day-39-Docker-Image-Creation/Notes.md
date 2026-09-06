# Day 39 - Docker Container and Image Fundamentals

## 1. What Is a Docker Image?

A Docker image is a **read-only template** used to create Docker containers.

An image can contain:

- Application code
- Operating system libraries
- Runtime dependencies
- Configuration files
- Environment settings

Examples:

```text
ubuntu:latest
nginx:latest
media:devops
```

Docker images are used as the blueprint for containers.

---

## 2. What Is a Docker Container?

A Docker container is a **running instance of a Docker image**.

For example:

```text
Docker Image
ubuntu:latest
      ↓
Docker Container
ubuntu_latest
```

Multiple containers can be created from the same image.

---

## 3. Docker Image vs Docker Container

| Docker Image | Docker Container |
|---|---|
| Blueprint or template | Running instance of an image |
| Read-only layers | Has a writable container layer |
| Used to create containers | Created from an image |
| Can be stored and shared | Usually temporary runtime environment |

---

## 4. Why Create an Image from a Container?

During development or troubleshooting, changes may be made directly inside a running container.

Examples include:

- Installing packages
- Changing configuration files
- Adding application files
- Updating dependencies
- Testing software changes

If the container is deleted, these changes may be lost.

The `docker commit` command allows the current container state to be saved as a new Docker image.

---

## 5. The `docker commit` Command

Syntax:

```bash
docker commit <container_name_or_id> <repository>:<tag>
```

Example:

```bash
docker commit ubuntu_latest media:devops
```

This creates:

- Repository: `media`
- Tag: `devops`

The resulting image contains the container's committed filesystem state.

---

## 6. Important Docker Commands

### List Running Containers

```bash
docker ps
```

### List All Containers

```bash
docker ps -a
```

### List Docker Images

```bash
docker images
```

### Create an Image from a Container

```bash
docker commit <container> <image>:<tag>
```

### Inspect an Image

```bash
docker image inspect <image>:<tag>
```

Example:

```bash
docker image inspect media:devops
```

### Run a Container from an Image

```bash
docker run -it <image>:<tag>
```

Example:

```bash
docker run -it media:devops
```

---

## 7. Docker Image Naming Convention

A Docker image generally follows this format:

```text
<repository>:<tag>
```

Example:

```text
media:devops
```

Where:

- `media` is the repository or image name.
- `devops` is the tag used to identify a particular version or variant.

If no tag is specified, Docker generally uses `latest` by default.

---

## 8. Docker Commit vs Dockerfile

### Docker Commit

```bash
docker commit ubuntu_latest media:devops
```

Useful for:

- Quick backups
- Capturing experimental changes
- Temporary snapshots
- Troubleshooting environments

### Dockerfile

A Dockerfile defines the image creation process as code.

Useful for:

- Repeatable builds
- Version-controlled infrastructure
- CI/CD pipelines
- Production environments

### Best Practice

For production systems, a **Dockerfile is generally preferred** because it provides a reproducible and documented image build process.

`docker commit` is useful for quickly preserving the current state of a container.

---

## 9. Key Takeaway

The workflow demonstrated in this task is:

```text
Docker Image
     ↓
Create Container
     ↓
Modify/Test Container
     ↓
docker commit
     ↓
New Docker Image
```

This process is useful when temporary changes made inside a container need to be preserved and reused.

---

## 🧠 Day 39 Learning Summary

- Understood the difference between Docker images and containers.
- Learned how to inspect running containers.
- Used `docker commit` to preserve container changes.
- Created the image `media:devops`.
- Verified the created image using Docker image commands.
- Learned when `docker commit` is useful and why Dockerfiles are preferred for reproducible production builds.

**Day 39 Completed Successfully ✔️**
