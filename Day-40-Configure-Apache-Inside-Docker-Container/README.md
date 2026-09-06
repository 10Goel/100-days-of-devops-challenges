# Day 40 - Configure Apache Inside a Running Docker Container

## 📌 Challenge Overview

A Nautilus DevOps team member had started configuring services inside an existing Docker container but could not complete the work.

The objective of this challenge was to configure Apache inside the running `kkloud` container on **App Server 3**, change Apache's listening port, ensure the service was running, and keep the container running at the end.

## 🎯 Task Objectives

- Access **App Server 3**.
- Work with the running Docker container named `kkloud`.
- Install `apache2` inside the container using `apt`.
- Configure Apache to listen on port **5002** instead of port 80.
- Avoid binding Apache to a specific IP address or hostname.
- Ensure Apache is running inside the container.
- Ensure the Docker container remains running after configuration.

## 🏗️ Environment

| Component | Details |
|---|---|
| Target Server | App Server 3 |
| Docker Container | `kkloud` |
| Package Manager | `apt` |
| Web Server | Apache2 |
| Required Port | `5002` |
| Default Apache Port | `80` |

## 🚀 Implementation Summary

### 1. Verified the Running Container

```bash
docker ps
```

### 2. Accessed the Container

```bash
docker exec -it kkloud bash
```

### 3. Installed Apache

```bash
apt update
apt install -y apache2
```

### 4. Configured Apache

Apache was configured to listen on port `5002` by updating:

```text
/etc/apache2/ports.conf
```

The default virtual host configuration was also updated to use port `5002`.

### 5. Validated and Started Apache

```bash
apache2ctl configtest
service apache2 start
```

### 6. Verified the Service

```bash
ss -lntp | grep 5002
curl -I http://localhost:5002
```

### 7. Verified Container State

After exiting the container, the Docker host was checked to ensure that `kkloud` remained running.

## 🧠 Key Learning Outcomes

- Managing services inside existing Docker containers.
- Using `docker exec` for interactive container administration.
- Installing software dynamically inside containers.
- Understanding isolated process environments.
- Configuring application services inside a container.
- Changing service ports and validating network listeners.
- Understanding container processes and lifecycle.
- Verifying that a container remains operational after service configuration.

## ⚠️ Production Perspective

Although installing and configuring software interactively inside a running container is useful for troubleshooting and lab environments, it is generally not the preferred production workflow.

Production-ready containers should normally use:

- Dockerfiles
- Declarative configuration
- Version-controlled configuration
- Reproducible image builds
- CI/CD pipelines

## ✅ Task Status

**Day 40 Completed Successfully ✔️**

---

### 🚀 100 Days of DevOps Journey

This repository is part of my **100 Days of DevOps** learning journey, where I build practical experience with Linux, Git, Docker, Kubernetes, CI/CD, cloud infrastructure, automation, and modern DevOps practices.
