# DevOps Day 47 – Commands Reference

This file contains the commands used to complete the Dockerized Python application deployment on **App Server 2**.

---

## 1. Connect to App Server 2

```bash
ssh steve@stapp02
```

Verify the host:

```bash
hostname
```

---

## 2. Inspect the Application Files

```bash
sudo ls -l /python_app/src/
```

Inspect the dependency file:

```bash
sudo cat /python_app/src/requirements.txt
```

Inspect the Python application:

```bash
sudo cat /python_app/src/server.py
```

Optional port check:

```bash
sudo grep -n "8085" /python_app/src/server.py
```

---

## 3. Move to the Docker Build Directory

```bash
cd /python_app
```

Verify the working directory:

```bash
pwd
```

Expected:

```text
/python_app
```

---

## 4. Create the Dockerfile

```bash
sudo vi Dockerfile
```

Dockerfile contents:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY src/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

EXPOSE 8085

CMD ["python", "server.py"]
```

Verify:

```bash
sudo cat Dockerfile
```

---

## 5. Build the Docker Image

```bash
sudo docker build -t nautilus/python-app .
```

Verify the image:

```bash
sudo docker images nautilus/python-app
```

---

## 6. Check for an Existing Container

```bash
sudo docker ps -a --filter "name=pythonapp_nautilus"
```

If an old container exists and must be removed:

```bash
sudo docker rm -f pythonapp_nautilus
```

---

## 7. Create and Run the Container

```bash
sudo docker run -d \
  --name pythonapp_nautilus \
  -p 8095:8085 \
  nautilus/python-app
```

---

## 8. Verify the Container

```bash
sudo docker ps --filter "name=pythonapp_nautilus"
```

Show the configured port mapping:

```bash
sudo docker port pythonapp_nautilus
```

Expected mapping:

```text
8085/tcp -> 0.0.0.0:8095
```

---

## 9. Check Application Logs

```bash
sudo docker logs pythonapp_nautilus
```

Follow logs continuously if troubleshooting:

```bash
sudo docker logs -f pythonapp_nautilus
```

Press `Ctrl+C` to stop following logs.

---

## 10. Test the Application

```bash
curl http://localhost:8095/
```

Verbose test:

```bash
curl -v http://localhost:8095/
```

---

## Useful Docker Troubleshooting Commands

List all containers:

```bash
sudo docker ps -a
```

Inspect the container:

```bash
sudo docker inspect pythonapp_nautilus
```

Stop the container:

```bash
sudo docker stop pythonapp_nautilus
```

Start it again:

```bash
sudo docker start pythonapp_nautilus
```

Restart it:

```bash
sudo docker restart pythonapp_nautilus
```

Enter the running container:

```bash
sudo docker exec -it pythonapp_nautilus /bin/sh
```

Remove the container:

```bash
sudo docker rm -f pythonapp_nautilus
```

Remove the image if rebuilding from scratch:

```bash
sudo docker rmi nautilus/python-app
```

---

## Complete Command Flow

```bash
ssh steve@stapp02
hostname

sudo ls -l /python_app/src/
sudo cat /python_app/src/requirements.txt
sudo cat /python_app/src/server.py

cd /python_app
sudo vi Dockerfile
sudo cat Dockerfile

sudo docker build -t nautilus/python-app .
sudo docker images nautilus/python-app

sudo docker ps -a --filter "name=pythonapp_nautilus"

sudo docker run -d \
  --name pythonapp_nautilus \
  -p 8095:8085 \
  nautilus/python-app

sudo docker ps --filter "name=pythonapp_nautilus"
sudo docker port pythonapp_nautilus
sudo docker logs pythonapp_nautilus
curl http://localhost:8095/
```
