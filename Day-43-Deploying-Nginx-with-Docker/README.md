# Day 43 — Deploying Nginx with Docker

## 📌 KodeKloud 100 Days of DevOps

**Day:** 43  
**Focus:** Running an Nginx Alpine container with Docker port mapping  
**Environment:** Stratos Datacenter — Application Server 2 (`stapp02`)

---

## 🎯 Objective

The Nautilus DevOps team required an Nginx-based application container on **Application Server 2**.

The task was to:

1. Pull the `nginx:alpine` Docker image.
2. Create a Docker container named `games`.
3. Map host port `5000` to container port `80`.
4. Keep the container running.

---

## 🛠️ Implementation

### 1. Pull the Nginx Alpine image

```bash
docker pull nginx:alpine
```

### 2. Create and start the container

```bash
docker run -d --name games -p 5000:80 nginx:alpine
```

The command runs the container in detached mode and publishes:

```text
Application Server 2 host:5000
        ↓
Docker container:80
        ↓
Nginx
```

### 3. Verify the running container

```bash
docker ps
```

Expected port mapping:

```text
0.0.0.0:5000->80/tcp
```

### 4. Test the Nginx service

```bash
curl http://localhost:5000
```

The Nginx welcome page confirms that the web server is reachable through the published host port.

---

## 🔍 Key Docker Concepts

### Docker Image

`nginx:alpine` is a lightweight Nginx image based on Alpine Linux.

### Container

The image was instantiated as a container named:

```text
games
```

### Detached Mode

The `-d` option starts the container in the background.

### Port Publishing

The option:

```bash
-p 5000:80
```

means:

```text
Host Port 5000 → Container Port 80
```

This makes Nginx accessible through port `5000` on the Docker host.

### Container Lifecycle

The container continues running because Nginx runs as the container's foreground process.

---

## ✅ Verification Checklist

- [x] `nginx:alpine` image pulled
- [x] Container created with name `games`
- [x] Host port `5000` mapped to container port `80`
- [x] Container running in detached mode
- [x] Nginx response verified with `curl`
- [x] Day 43 challenge completed successfully

---

## 🧠 What I Learned

This task strengthened my understanding of:

- Docker image pulling
- Container creation and execution
- Detached containers
- Container naming
- Docker port publishing
- Nginx deployment using containers
- Basic container verification
- Host-to-container networking

---

## 🚀 Practical DevOps Relevance

Port publishing is fundamental when exposing containerized services.

In real-world environments, a Dockerized application commonly follows a flow such as:

```text
Client
  ↓
Host / Load Balancer
  ↓
Published Host Port
  ↓
Docker Container
  ↓
Application Service
```

Understanding this mechanism is essential before moving toward Docker Compose, reverse proxies, Kubernetes Services, and container orchestration.

---

**Status: ✅ Completed**
