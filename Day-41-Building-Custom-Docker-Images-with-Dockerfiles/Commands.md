# Day 41 - Commands Used

## 📌 Dockerfile Creation

Create the required directory:

```bash
sudo mkdir -p /opt/docker
```

Create the Dockerfile:

```bash
sudo vi /opt/docker/Dockerfile
```

---

## 🐳 Dockerfile Content

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

## 🔎 Verify the Dockerfile

```bash
sudo cat /opt/docker/Dockerfile
```

---

## 🏗️ Build the Docker Image

Navigate to the Docker build context:

```bash
cd /opt/docker
```

Build the image:

```bash
sudo docker build -t nautilus-apache:latest .
```

The `.` represents the current directory and is used as the Docker build context.

---

## 📦 List Docker Images

```bash
sudo docker images
```

Alternative command:

```bash
sudo docker image ls
```

---

## ▶️ Run the Container

```bash
sudo docker run -d --name apache-test -p 5001:5001 nautilus-apache:latest
```

Command explanation:

- `-d` → Run container in detached mode.
- `--name apache-test` → Assign a name to the container.
- `-p 5001:5001` → Map host port `5001` to container port `5001`.
- `nautilus-apache:latest` → Docker image used to create the container.

---

## 🔍 Check Running Containers

```bash
sudo docker ps
```

View all containers:

```bash
sudo docker ps -a
```

---

## ⚙️ Verify Apache Port Configuration

```bash
sudo docker exec apache-test cat /etc/apache2/ports.conf
```

Expected output:

```text
Listen 5001
```

---

## 📋 View Container Logs

```bash
sudo docker logs apache-test
```

Follow logs continuously:

```bash
sudo docker logs -f apache-test
```

---

## 🧪 Access a Shell Inside the Container

```bash
sudo docker exec -it apache-test bash
```

Exit the container shell:

```bash
exit
```

---

## 🛑 Stop the Container

```bash
sudo docker stop apache-test
```

---

## 🗑️ Remove the Container

```bash
sudo docker rm apache-test
```

Force stop and remove:

```bash
sudo docker rm -f apache-test
```

---

## 🗑️ Remove the Docker Image

```bash
sudo docker rmi nautilus-apache:latest
```

---

## 🔧 Useful Dockerfile Inspection Commands

View image history:

```bash
sudo docker history nautilus-apache:latest
```

Inspect image metadata:

```bash
sudo docker inspect nautilus-apache:latest
```

Inspect container metadata:

```bash
sudo docker inspect apache-test
```

---

## 💡 Useful Build Command Variations

Build without using cache:

```bash
sudo docker build --no-cache -t nautilus-apache:latest .
```

Build with a specific Dockerfile:

```bash
sudo docker build -f /opt/docker/Dockerfile -t nautilus-apache:latest /opt/docker
```
