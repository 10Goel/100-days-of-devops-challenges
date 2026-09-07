# Day 41 - Building Custom Docker Images with Dockerfiles

## 🚀 100 Days of DevOps Challenge

This repository contains my solution for **Day 41** of the KodeKloud **100 Days of DevOps** challenge.
The objective of this task was to create a custom Docker image using a **Dockerfile**, install and configure Apache HTTP Server, and modify Apache to listen on a custom port.

---

## 📌 Task Overview

The Nautilus application development team required a custom Docker image with the following configuration:

- Use `ubuntu:24.04` as the base image.
- Install `apache2`.
- Configure Apache HTTP Server to listen on port `5001`.
- Do not modify other Apache configuration settings such as the document root.
- Create the Dockerfile at:

```text
/opt/docker/Dockerfile
```

---

## 🏗️ Dockerfile Used

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 && \
    sed -i 's/^Listen 80$/Listen 5001/' /etc/apache2/ports.conf && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 5001

CMD ["apachectl", "-D", "FOREGROUND"]
```

---

## 🔍 Dockerfile Explanation

### `FROM ubuntu:24.04`

Defines the base operating system image used to build the container.

### `RUN`

Executes commands during the image build process.

In this task, the `RUN` instruction:

1. Updates the package repository metadata.
2. Installs Apache HTTP Server.
3. Uses `sed` to change Apache's listening port from `80` to `5001`.
4. Cleans the package cache.
5. Removes unnecessary package list files to reduce image size.

### `DEBIAN_FRONTEND=noninteractive`

Prevents interactive prompts while installing packages inside the Docker build process.

### `sed -i`

Modifies the Apache configuration directly:

```bash
sed -i 's/^Listen 80$/Listen 5001/' /etc/apache2/ports.conf
```

This changes only the required `Listen 80` directive while preserving the remaining Apache configuration.

### `EXPOSE 5001`

Documents that the containerized application is intended to listen on port `5001`.

### `CMD`

Starts Apache in the foreground:

```dockerfile
CMD ["apachectl", "-D", "FOREGROUND"]
```

Running the main process in the foreground is important because a Docker container normally exits when its primary process stops.

---

## 🛠️ Implementation Steps

### 1. Create the Docker directory

```bash
sudo mkdir -p /opt/docker
```

### 2. Create the Dockerfile

```bash
sudo vi /opt/docker/Dockerfile
```

### 3. Build the Docker image

```bash
cd /opt/docker
sudo docker build -t nautilus-apache:latest .
```

### 4. Verify the image

```bash
sudo docker images
```

### 5. Run a test container

```bash
sudo docker run -d --name apache-test -p 5001:5001 nautilus-apache:latest
```

### 6. Verify the running container

```bash
sudo docker ps
```

### 7. Verify Apache configuration

```bash
sudo docker exec apache-test cat /etc/apache2/ports.conf
```

Expected configuration:

```text
Listen 5001
```

---

## 🧠 Key Concepts Practiced

- Docker image creation
- Dockerfile syntax
- Base images
- Docker build layers
- `FROM`, `RUN`, `EXPOSE`, and `CMD`
- Installing software during image builds
- Non-interactive package installation
- Configuration automation using `sed`
- Docker image tagging
- Container port publishing
- Running services in the foreground
- Docker image optimization and cleanup

---
## 🎯 Learning Outcome

By completing this challenge, I gained hands-on experience creating custom Docker images using Dockerfiles.

I learned how to automate application installation and configuration during the image build process, modify service configurations programmatically, expose application ports, and ensure that the container's primary service runs correctly in the foreground.

---

## ✅ Challenge Status

**Day 41: Completed Successfully 🎉**

### Progress
Continuing my journey through the **100 Days of DevOps Challenge** with a stronger understanding of Docker image creation and Dockerfile best practices.

---

### Author

**Sujal Goel**  
DevOps Learner | Cloud Enthusiast | AWS Certified Cloud Practitioner
