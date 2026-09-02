# Day 36 — Commands

## Objective

Create a running Docker container named `nginx_3` on Application Server 3 using `nginx:alpine`.

## 1. Connect to Application Server 3

```bash
ssh banner@stapp03
```

## 2. Verify Docker

```bash
docker --version
```

## 3. Create and Start the Container

```bash
docker run -d --name nginx_3 nginx:alpine
```

## 4. Verify All Running Containers

```bash
docker ps
```

The required values should include:

```text
IMAGE: nginx:alpine
STATUS: Up ...
NAME: nginx_3
```

## 5. Verify the Specific Container

```bash
docker ps --filter "name=nginx_3"
```

## 6. Optional State Verification

```bash
docker inspect -f '{{.State.Running}}' nginx_3
```

Expected:

```text
true
```

## Complete Command Sequence

```bash
ssh banner@stapp03
docker --version
docker run -d --name nginx_3 nginx:alpine
docker ps
docker ps --filter "name=nginx_3"
```

## Useful Docker Commands

### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### Stop a container

```bash
docker stop nginx_3
```

### Start an existing container

```bash
docker start nginx_3
```

### Restart a container

```bash
docker restart nginx_3
```

### View logs

```bash
docker logs nginx_3
```

### Inspect a container

```bash
docker inspect nginx_3
```

### Execute a command inside a running container

```bash
docker exec nginx_3 ls
```

### Open a shell inside an Alpine-based container

```bash
docker exec -it nginx_3 sh
```

### Remove a stopped container

```bash
docker rm nginx_3
```

### Force remove a running container

```bash
docker rm -f nginx_3
```

## Completion

**KodeKloud Day 36 — Successfully Completed ✅**
