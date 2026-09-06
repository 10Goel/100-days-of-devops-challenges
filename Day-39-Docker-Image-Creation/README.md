# Day 39 - Docker Image Creation from a Container

## 📌 Task Overview

A Nautilus developer was testing new changes inside an existing Docker container and needed to preserve those changes as a reusable Docker image.

The objective was to create a new Docker image named:

```text
media:devops
```

from the running container:

```text
ubuntu_latest
```

on **Application Server 1**.

---

## 🎯 Objective

- Connect to Application Server 1.
- Verify that the `ubuntu_latest` container is available.
- Create a Docker image from the container's current state.
- Name the image `media` with the tag `devops`.
- Verify that the image was created successfully.

---

## 🛠️ Technologies Used

- Linux
- Docker
- SSH
- Docker Containers
- Docker Images

---

## 🚀 Implementation

The existing container was first verified using Docker.

```bash
docker ps
```

The current state of the container was then saved as a new Docker image using:

```bash
docker commit ubuntu_latest media:devops
```

Finally, the image was verified:

```bash
docker images
```

---

## 📋 Verification

The successful output should contain an image similar to:

```text
REPOSITORY   TAG
media        devops
```

The image can also be verified with:

```bash
docker images media
```

---

## 🧠 Key Learning

Docker containers are mutable runtime instances, meaning changes can occur while an application is running. The `docker commit` command can capture the current filesystem state of a container and create a new reusable image from it.

This task demonstrated how existing container changes can be preserved as a Docker image.

---
## ✅ Task Status

**Completed Successfully ✔️**

---

### 100 Days of DevOps

This repository is part of my **100 Days of DevOps** learning journey, where I practice Linux, Git, Docker, Kubernetes, CI/CD, cloud, automation, and other DevOps technologies through hands-on challenges.
