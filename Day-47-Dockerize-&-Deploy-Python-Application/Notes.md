# DevOps Day 47 – Docker Notes

## Objective

The objective of Day 47 was to Dockerize an existing Python application and deploy it on **App Server 2**.

The challenge covered the full containerization workflow:

```text
Application Source
      ↓
Dockerfile
      ↓
Docker Image
      ↓
Docker Container
      ↓
Port Mapping
      ↓
Application Access
```

---

## 1. What is a Dockerfile?

A `Dockerfile` is a text file containing instructions that Docker follows to build an image.

Example:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
EXPOSE 8085
CMD ["python", "server.py"]
```

Each instruction creates or configures part of the final image.

---

## 2. Docker Image vs Docker Container

A Docker image is a reusable, read-only template containing the application and its dependencies.

A Docker container is a running instance created from that image.

For this task:

```text
Image:
nautilus/python-app

Container:
pythonapp_nautilus
```

Conceptually:

```text
Dockerfile
    ↓ docker build
Docker Image
    ↓ docker run
Docker Container
```

---

## 3. Understanding the Docker Build Context

The image was built using:

```bash
sudo docker build -t nautilus/python-app .
```

The final `.` is the **build context**.

Because the command was executed from:

```text
/python_app
```

Docker was able to access files underneath that directory:

```text
/python_app/
└── src/
    ├── requirements.txt
    └── server.py
```

Therefore these Dockerfile instructions were valid:

```dockerfile
COPY src/requirements.txt .
COPY src/ .
```

Docker cannot normally copy files located outside the selected build context.

---

## 4. Understanding `FROM`

```dockerfile
FROM python:3.11-slim
```

`FROM` defines the base image used to build the new image.

The Python image already provides:

- Python runtime
- Python standard libraries
- `pip`

The `slim` variant is smaller than the regular Python image and is useful when the application does not require a large set of system packages.

---

## 5. Understanding `WORKDIR`

```dockerfile
WORKDIR /app
```

`WORKDIR` sets the default working directory inside the container.

Subsequent commands such as:

```dockerfile
COPY
RUN
CMD
```

operate relative to `/app` where applicable.

After the build, the application files existed similar to:

```text
/app/
├── requirements.txt
└── server.py
```

---

## 6. Installing Python Dependencies

The Dockerfile used:

```dockerfile
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

The first instruction copies the application's dependency list into the image.

The second installs all packages listed inside it.

The option:

```text
--no-cache-dir
```

prevents `pip` from retaining its download cache, which helps reduce unnecessary image size.

---

## 7. Why `requirements.txt` Was Copied Before the Application

A useful Docker optimization is to copy the dependency file before copying the application source code:

```dockerfile
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
```

Docker builds images using layers and can reuse unchanged layers from its build cache.

If application code changes but `requirements.txt` remains unchanged, Docker may reuse the dependency installation layer instead of reinstalling every Python package.

This can significantly improve rebuild speed.

---

## 8. Understanding `COPY`

The instruction:

```dockerfile
COPY src/ .
```

copies the contents of:

```text
/python_app/src/
```

from the Docker build context into:

```text
/app/
```

inside the Docker image.

The source path is relative to the build context, while the destination is relative to the active `WORKDIR` when a relative destination is used.

---

## 9. Understanding `EXPOSE`

```dockerfile
EXPOSE 8085
```

`EXPOSE` documents that the application is intended to listen on port `8085` inside the container.

An important distinction is:

> `EXPOSE` does not automatically publish the port to the Docker host.

Port publishing is performed when the container starts using `-p`.

---

## 10. Understanding `CMD`

```dockerfile
CMD ["python", "server.py"]
```

`CMD` defines the default process that runs when the container starts.

Because the working directory is `/app`, this effectively runs:

```text
python /app/server.py
```

The JSON-array form is known as the **exec form** and is generally preferred because Docker launches the process directly rather than running it through a shell.

---

## 11. Building the Image

The image was built using:

```bash
sudo docker build -t nautilus/python-app .
```

Explanation:

```text
docker build
    -t nautilus/python-app
    .
```

Where:

- `docker build` starts an image build.
- `-t` assigns a name/tag to the resulting image.
- `nautilus/python-app` is the required image name.
- `.` specifies the current directory as the build context.

---

## 12. Creating the Container

The container was created with:

```bash
sudo docker run -d \
  --name pythonapp_nautilus \
  -p 8095:8085 \
  nautilus/python-app
```

Explanation:

```text
-d
```

runs the container in detached/background mode.

```text
--name pythonapp_nautilus
```

assigns the required container name.

```text
-p 8095:8085
```

maps a host port to a container port.

```text
nautilus/python-app
```

specifies the image used to create the container.

---

## 13. Host Port vs Container Port

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

For this task:

```text
-p 8095:8085
```

means:

```text
Host Machine                Docker Container

Port 8095  -------------->  Port 8085
```

Therefore:

```bash
curl http://localhost:8095/
```

reaches the application listening on port `8085` inside the container.

---

## 14. Why the Python App Must Listen on the Correct Interface

Inside a container, network applications should normally listen on:

```text
0.0.0.0
```

rather than only:

```text
127.0.0.1
```

`127.0.0.1` means the process accepts connections only from the container's loopback interface.

`0.0.0.0` allows the process to accept traffic arriving through the container's network interface, including traffic forwarded by Docker's published port.

For example, a Flask application commonly uses:

```python
app.run(host="0.0.0.0", port=8085)
```

---

## 15. `docker ps` vs `docker ps -a`

To show only running containers:

```bash
sudo docker ps
```

To show all containers, including stopped or failed containers:

```bash
sudo docker ps -a
```

This is especially useful when a container starts and then exits immediately.

---

## 16. Using Docker Logs for Troubleshooting

If the application cannot be reached, one of the first commands to run is:

```bash
sudo docker logs pythonapp_nautilus
```

Possible issues visible in logs include:

- missing Python modules,
- application exceptions,
- invalid configuration,
- incorrect ports,
- file-not-found errors,
- application startup failures.

To follow logs live:

```bash
sudo docker logs -f pythonapp_nautilus
```

---

## 17. Useful Verification Workflow

A good Docker verification sequence is:

```bash
sudo docker images nautilus/python-app
sudo docker ps -a --filter "name=pythonapp_nautilus"
sudo docker port pythonapp_nautilus
sudo docker logs pythonapp_nautilus
curl http://localhost:8095/
```

This verifies:

```text
Image exists
      ↓
Container exists
      ↓
Container is running
      ↓
Correct port mapping exists
      ↓
Application started successfully
      ↓
Application responds to HTTP requests
```

---

## 18. Common Problems and Troubleshooting

### Problem: Container Exits Immediately

Check:

```bash
sudo docker ps -a
sudo docker logs pythonapp_nautilus
```

Possible reasons include:

- application crash,
- incorrect `CMD`,
- missing packages,
- missing source files.

---

### Problem: `curl` Returns Connection Refused

Verify:

```bash
sudo docker ps
sudo docker port pythonapp_nautilus
sudo docker logs pythonapp_nautilus
```

Also confirm that the Python application listens on port `8085` and preferably on `0.0.0.0`.

---

### Problem: Port Is Already Allocated

If Docker reports that port `8095` is already in use, inspect listeners or containers:

```bash
sudo docker ps
sudo ss -lntp | grep 8095
```

A different process or container may already be bound to that port.

---

### Problem: Container Name Already Exists

Docker container names must be unique.

Check:

```bash
sudo docker ps -a --filter "name=pythonapp_nautilus"
```

If the old container is no longer needed:

```bash
sudo docker rm -f pythonapp_nautilus
```

Then recreate it.

---

### Problem: Docker Build Cannot Find a File

Example:

```text
COPY failed: file not found
```

Check the build context and file paths.

If building from `/python_app`:

```bash
cd /python_app
sudo docker build -t nautilus/python-app .
```

then Dockerfile source paths should be relative to `/python_app`, such as:

```dockerfile
COPY src/requirements.txt .
COPY src/ .
```

---

## 19. Important Docker Concepts Learned

### Image

A packaged application environment containing code, runtime, libraries, and configuration.

### Container

A running process created from a Docker image.

### Build Context

The set of files available to Docker during `docker build`.

### Dockerfile

Instructions defining how the image is built.

### Port Publishing

Allows a host port to forward traffic to a port inside a container.

### Container Logs

Application output captured by Docker and accessible through `docker logs`.

---

## 20. Final Architecture

```text
App Server 2

    curl localhost:8095
             |
             v
    Host Port 8095
             |
             | Docker Port Publishing
             v
    +--------------------------+
    | pythonapp_nautilus       |
    |                          |
    | Container Port 8085      |
    |          |               |
    |          v               |
    |      server.py           |
    |                          |
    | Image:                   |
    | nautilus/python-app      |
    +--------------------------+
```

---

## Key Takeaway

Day 47 demonstrated how Docker packages an application and its dependencies into a reproducible image and then runs that image as an isolated container.

The most important relationships from this task are:

```text
Dockerfile → Image → Container
```

and:

```text
Host Port 8095 → Container Port 8085 → Python Application
```

Successfully receiving a response from:

```bash
curl http://localhost:8095/
```

confirmed that the image, container, application process, and Docker networking configuration were all functioning correctly.
