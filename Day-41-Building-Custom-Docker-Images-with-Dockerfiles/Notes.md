# Dockerfile Notes - Day 41

## 🐳 What is a Dockerfile?

A **Dockerfile** is a text file containing instructions used by Docker to automatically build a container image.

Instead of manually creating and configuring a container every time, a Dockerfile allows the entire process to be defined as **Infrastructure as Code**.

Example workflow:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container
```

---

# 1. Dockerfile vs Docker Image vs Docker Container

## Dockerfile

A set of instructions describing how an image should be built.

Example:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
```

## Docker Image

A packaged, read-only template created from a Dockerfile.

Examples:

```text
ubuntu:24.04
nginx:latest
nautilus-apache:latest
```

## Docker Container

A running instance of a Docker image.

Relationship:

```text
Dockerfile → Image → Container
```

---

# 2. Important Dockerfile Instructions

## `FROM`

Defines the base image.

```dockerfile
FROM ubuntu:24.04
```

Every Dockerfile normally starts with a `FROM` instruction.

The base image provides the operating system environment and initial filesystem.

---

## `RUN`

Executes commands while building the image.

```dockerfile
RUN apt-get update
```

Multiple commands can be combined:

```dockerfile
RUN apt-get update && \
    apt-get install -y apache2
```

Each `RUN` instruction creates a new image layer.

---

## `CMD`

Defines the default command executed when a container starts.

```dockerfile
CMD ["apachectl", "-D", "FOREGROUND"]
```

A Dockerfile should generally have one effective `CMD`.

If multiple `CMD` instructions are present, only the final one is used.

### Exec form

```dockerfile
CMD ["command", "argument"]
```

Recommended because it does not invoke an additional shell.

### Shell form

```dockerfile
CMD command argument
```

Exec form is generally preferred for predictable signal handling.

---

## `ENTRYPOINT`

Defines the primary executable for a container.

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

When combined:

```text
ENTRYPOINT + CMD
```

The `ENTRYPOINT` defines the executable while `CMD` can provide default arguments.

---

## `EXPOSE`

Documents the port on which an application is expected to listen.

```dockerfile
EXPOSE 5001
```

Important: `EXPOSE` does **not** publish the port automatically.

Port publishing happens during container execution:

```bash
docker run -p 5001:5001 image-name
```

---

## `COPY`

Copies files from the Docker build context into the image.

```dockerfile
COPY index.html /var/www/html/
```

---

## `ADD`

Similar to `COPY` but has additional features such as archive extraction and remote URL handling.

In most situations, `COPY` is preferred because it is simpler and more predictable.

---

## `WORKDIR`

Sets the working directory for subsequent instructions.

```dockerfile
WORKDIR /app
```

Example:

```dockerfile
WORKDIR /app
COPY . .
RUN npm install
```

---

## `ENV`

Defines environment variables.

```dockerfile
ENV APP_ENV=production
```

---

## `ARG`

Defines build-time variables.

```dockerfile
ARG VERSION=1.0
```

Example:

```bash
docker build --build-arg VERSION=2.0 .
```

Unlike `ENV`, `ARG` is primarily available during the image build process.

---

## `USER`

Specifies which user should execute subsequent instructions or run the container.

```dockerfile
USER appuser
```

Running applications as a non-root user is generally considered a Docker security best practice.

---

## `VOLUME`

Declares a mount point intended for persistent data.

```dockerfile
VOLUME ["/var/lib/data"]
```

---

# 3. Docker Build Context

The final argument of the build command is the **build context**.

```bash
docker build -t my-image .
```

Here:

```text
.
```

means the current directory.

Docker sends the build context to the Docker daemon.

Therefore, unnecessary files should be excluded using `.dockerignore`.

---

# 4. Docker Image Layers

Docker images are built in layers.

Example:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get install -y apache2
COPY app/ /app/
```

Conceptually:

```text
Base Image
   ↓
Layer 1: apt-get update
   ↓
Layer 2: Install Apache
   ↓
Layer 3: Copy application files
```

Docker can reuse cached layers during future builds.

---

# 5. Docker Build Cache

Docker caches layers to improve build performance.

If an instruction and its dependencies have not changed, Docker may reuse the previously built layer.

Example:

```bash
docker build -t my-image .
```

Disable cache:

```bash
docker build --no-cache -t my-image .
```

## Best Practice

Place instructions that change less frequently earlier in the Dockerfile.

This improves cache reuse and reduces build time.

---

# 6. Combining RUN Commands

Instead of:

```dockerfile
RUN apt-get update
RUN apt-get install -y apache2
RUN apt-get clean
```

Use:

```dockerfile
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean
```

Benefits:

- Fewer layers.
- Smaller image size.
- Cleaner image build process.

---

# 7. Package Installation Best Practices

A common pattern for Debian or Ubuntu images is:

```dockerfile
RUN apt-get update && \
    apt-get install -y package-name && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

Why?

- `apt-get update` refreshes package metadata.
- `apt-get install` installs the required package.
- `apt-get clean` removes cached package files.
- Removing `/var/lib/apt/lists/*` reduces unnecessary image data.

---

# 8. Why Use Non-Interactive Installation?

During Docker builds, package installations must not wait for user input.

Example:

```dockerfile
DEBIAN_FRONTEND=noninteractive apt-get install -y apache2
```

This ensures the build can run automatically without interactive prompts.

---

# 9. Foreground Processes in Containers

A Docker container normally remains running while its primary process is running.

For Apache:

```dockerfile
CMD ["apachectl", "-D", "FOREGROUND"]
```

If Apache starts in the background and the main process exits, the container will also exit.

Therefore, long-running container applications should usually run in the foreground.

---

# 10. Configuration Changes During Image Build

Configuration files can be modified automatically during image creation.

Example:

```dockerfile
RUN sed -i 's/^Listen 80$/Listen 5001/' /etc/apache2/ports.conf
```

Advantages:

- Repeatable configuration.
- No manual post-build changes.
- Consistent environments.
- Easier automation.

---

# 11. Dockerfile Best Practices

## Use specific image versions

Prefer:

```dockerfile
FROM ubuntu:24.04
```

instead of:

```dockerfile
FROM ubuntu:latest
```

Specific versions improve reproducibility.

---

## Keep images small

Only install required packages.

Remove temporary files and caches when possible.

---

## Use `.dockerignore`

Example:

```text
.git
node_modules
*.log
.env
```

This prevents unnecessary files from entering the build context.

---

## Use multi-stage builds when appropriate

Example concept:

```text
Build Stage
    ↓
Compile Application
    ↓
Copy Only Final Artifact
    ↓
Runtime Stage
```

This helps create smaller production images.

---

## Avoid running applications as root

Create a dedicated user when possible:

```dockerfile
RUN useradd -m appuser
USER appuser
```

---

## Use clear image tags

Examples:

```text
myapp:1.0
myapp:1.1
myapp:production
```

Avoid relying exclusively on:

```text
latest
```

for production deployments.

---

# 12. Dockerfile Instruction Summary

| Instruction | Purpose |
|---|---|
| `FROM` | Defines the base image |
| `RUN` | Executes commands during image build |
| `CMD` | Defines the default container command |
| `ENTRYPOINT` | Defines the primary executable |
| `COPY` | Copies files into the image |
| `ADD` | Copies files with additional functionality |
| `EXPOSE` | Documents the application port |
| `WORKDIR` | Sets the working directory |
| `ENV` | Sets environment variables |
| `ARG` | Defines build-time variables |
| `USER` | Sets the user for commands/container |
| `VOLUME` | Declares persistent storage locations |

---

# 13. Day 41 Key Takeaways

Through this challenge, I practiced:

- Creating Dockerfiles.
- Building custom Docker images.
- Using Ubuntu as a base image.
- Installing packages during image creation.
- Configuring Apache automatically.
- Modifying configuration files with `sed`.
- Using `EXPOSE`.
- Using `CMD`.
- Understanding Docker build contexts.
- Understanding image layers and caching.
- Running and testing custom containers.
- Applying Dockerfile optimization practices.

---

## 🎯 Final Concept

The core idea behind a Dockerfile is:

> **Define the environment once, build it consistently anywhere.**

Dockerfiles make infrastructure and application environments repeatable, portable, version-controlled, and easier to automate.
