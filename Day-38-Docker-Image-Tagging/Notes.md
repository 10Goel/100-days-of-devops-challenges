# Docker Container Fundamentals - Day 38 Notes

## 1. What Is Docker?

Docker is a containerization platform used to package an application and its dependencies into a lightweight, portable unit called a **container**.

A Docker container can run consistently across different environments because the application dependencies are packaged together.

---

## 2. What Is a Container?

A container is an isolated process running on the host operating system.

Unlike a traditional virtual machine, a container does not require a complete guest operating system.

### Containers provide:

- Process isolation
- Dependency isolation
- Fast startup
- Portability
- Efficient resource utilization

---

## 3. What Is a Docker Image?

A Docker image is a read-only template used to create containers.

An image can contain:

- Application code
- Runtime environment
- Libraries
- Dependencies
- Configuration
- Operating system filesystem components

A container is created from an image.

### Relationship

```text
Docker Image
     |
     v
docker run
     |
     v
Docker Container
```

---

## 4. Docker Image Names and Tags

Docker images are commonly referenced using:

```text
repository:tag
```

Example:

```text
busybox:musl
```

Where:

- `busybox` is the repository name.
- `musl` is the image tag.

Tags are commonly used to identify:

- Versions
- Environments
- Releases
- Build types

Examples:

```text
nginx:latest
nginx:1.27
busybox:musl
busybox:blog
```

---

## 5. Pulling a Docker Image

The `docker pull` command downloads an image from a container registry.

Syntax:

```bash
docker pull IMAGE:TAG
```

Example:

```bash
docker pull busybox:musl
```

Docker downloads the required image layers and stores them locally.

To view local images:

```bash
docker images
```

or:

```bash
docker image ls
```

---

## 6. Docker Image Tagging

Docker allows multiple tags to reference the same image.

Syntax:

```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

Example:

```bash
docker tag busybox:musl busybox:blog
```

This creates a new reference named:

```text
busybox:blog
```

pointing to the same image as:

```text
busybox:musl
```

### Important

`docker tag` does **not** create a duplicate copy of the image.

Both tags can point to the same image ID:

```text
busybox:musl  ---> IMAGE_ID
busybox:blog  ---> IMAGE_ID
```

---

## 7. Why Is Docker Tagging Important?

Image tagging is important in real DevOps workflows because teams need meaningful ways to identify images.

For example:

```text
myapp:dev
myapp:test
myapp:staging
myapp:production
myapp:v1.0.0
```

Tags help teams:

- Track releases
- Manage versions
- Promote builds between environments
- Identify stable images
- Prepare images for deployment
- Push images to container registries

---

## 8. Useful Docker Image Commands

### List Images

```bash
docker images
```

### Pull an Image

```bash
docker pull IMAGE:TAG
```

### Tag an Image

```bash
docker tag SOURCE_IMAGE TARGET_IMAGE
```

### Inspect an Image

```bash
docker image inspect IMAGE:TAG
```

### Remove an Image Tag

```bash
docker rmi IMAGE:TAG
```

### Remove an Image

```bash
docker image rm IMAGE:TAG
```

---

## 9. Container Lifecycle Basics

A common Docker container lifecycle is:

```text
Image
  |
  v
docker run
  |
  v
Created Container
  |
  v
Running Container
  |
  v
Stopped Container
  |
  v
Removed Container
```

Useful commands:

```bash
docker run IMAGE
docker ps
docker ps -a
docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker rm CONTAINER
```

---

## 10. Key Takeaway from Day 38

The main objective of this task was to understand that a Docker image can have multiple tags.

The command:

```bash
docker tag busybox:musl busybox:blog
```

creates another tag for the same local image without creating another physical copy of the image layers.

This concept is essential for Docker image versioning and DevOps deployment workflows.
