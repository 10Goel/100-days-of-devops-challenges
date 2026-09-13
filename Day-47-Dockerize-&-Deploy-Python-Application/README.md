# DevOps Day 47 – Dockerize and Deploy a Python Application

## Overview

In this challenge, a Python application was containerized using Docker and deployed on **App Server 2**. The application dependencies were installed from `requirements.txt`, the application was exposed on container port **8085**, and the container was published on host port **8095**.

This task demonstrated the complete workflow of building a Docker image from a Python application, creating a container from that image, configuring port mapping, and validating the deployment using `curl`.

---

## Task Requirements

The challenge required the following:

- Work on **App Server 2**.
- Use the existing Python application located under `/python_app/src/`.
- Create a `Dockerfile` under `/python_app`.
- Use any Python image as the base image.
- Install dependencies using `requirements.txt`.
- Expose container port `8085`.
- Run `server.py` using Docker `CMD`.
- Build an image named:

```text
nautilus/python-app
```

- Create a container named:

```text
pythonapp_nautilus
```

- Map host port `8095` to container port `8085`.
- Verify the deployed application using:

```bash
curl http://localhost:8095/
```

---

## Application Structure

The application files were already available in the following structure:

```text
/python_app/
├── Dockerfile
└── src/
    ├── requirements.txt
    └── server.py
```

The Docker build context was `/python_app`, so files inside `src/` were referenced relative to this directory in the Dockerfile.

---

## Dockerfile

The following Dockerfile was created under `/python_app`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY src/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

EXPOSE 8085

CMD ["python", "server.py"]
```

### Dockerfile Explanation

- `FROM python:3.11-slim` uses a lightweight Python base image.
- `WORKDIR /app` defines the working directory inside the container.
- `COPY src/requirements.txt .` copies the dependency file into the image.
- `RUN pip install --no-cache-dir -r requirements.txt` installs the required Python packages.
- `COPY src/ .` copies the Python application code into the container.
- `EXPOSE 8085` documents the application's listening port.
- `CMD ["python", "server.py"]` starts the Python application when the container launches.

---

## Implementation

### 1. Connected to App Server 2

```bash
ssh steve@stapp02
```

The server was verified using:

```bash
hostname
```

---

### 2. Verified the Existing Application Files

```bash
sudo ls -l /python_app/src/
```

The application contained:

```text
requirements.txt
server.py
```

The files were inspected before creating the Docker image:

```bash
sudo cat /python_app/src/requirements.txt
sudo cat /python_app/src/server.py
```

---

### 3. Created the Dockerfile

```bash
cd /python_app
sudo vi Dockerfile
```

After saving the Dockerfile, its contents were verified using:

```bash
sudo cat /python_app/Dockerfile
```

---

### 4. Built the Docker Image

```bash
sudo docker build -t nautilus/python-app .
```

The image was verified using:

```bash
sudo docker images nautilus/python-app
```

---

### 5. Created and Started the Container

```bash
sudo docker run -d \
  --name pythonapp_nautilus \
  -p 8095:8085 \
  nautilus/python-app
```

The mapping configured:

```text
Host Port 8095  --->  Container Port 8085
```

---

### 6. Verified the Running Container

```bash
sudo docker ps --filter "name=pythonapp_nautilus"
```

The expected port mapping appeared similar to:

```text
0.0.0.0:8095->8085/tcp
```

Container logs were also checked:

```bash
sudo docker logs pythonapp_nautilus
```

---

### 7. Tested the Deployment

The application was tested locally from App Server 2:

```bash
curl http://localhost:8095/
```

A successful application response confirmed that the Python application was running correctly inside the Docker container.

---

## Port Mapping Flow

```text
Client Request
    |
    v
localhost:8095
    |
    v
Docker Host Port 8095
    |
    v
Container Port 8085
    |
    v
Python server.py
```

Docker's `-p 8095:8085` option publishes the container's port `8085` through port `8095` on the host machine.

---

## Verification Commands

```bash
sudo docker images nautilus/python-app
sudo docker ps --filter "name=pythonapp_nautilus"
sudo docker port pythonapp_nautilus
sudo docker logs pythonapp_nautilus
curl http://localhost:8095/
```

---

## Key Concepts Practiced

- Dockerfile creation
- Docker build context
- Python application containerization
- Installing application dependencies during image build
- Docker image creation
- Docker container lifecycle
- Port publishing and mapping
- Container log inspection
- Application verification using `curl`
- Difference between host ports and container ports

---

## Final Result

The Python application was successfully:

1. Containerized using Docker.
2. Packaged into the `nautilus/python-app` Docker image.
3. Deployed using the `pythonapp_nautilus` container.
4. Published from container port `8085` to host port `8095`.
5. Verified successfully using `curl`.

**Status: Completed Successfully ✅**
