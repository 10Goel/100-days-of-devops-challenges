# Day 39 - Commands

## 1. Connect to Application Server 1

Connect from the jump host using the credentials provided in the lab:

```bash
ssh <user>@stapp01
```

Example:

```bash
ssh tony@stapp01
```

---

## 2. Check Running Containers

Verify that the target container is running:

```bash
docker ps
```

Alternatively, filter the output:

```bash
docker ps --filter "name=ubuntu_latest"
```

---

## 3. Create a Docker Image from the Container

Create the required image from the current state of the container:

```bash
docker commit ubuntu_latest media:devops
```

### Command Structure

```bash
docker commit <container_name_or_id> <image_name>:<tag>
```

---

## 4. Verify the Newly Created Image

List all Docker images:

```bash
docker images
```

Check only the `media` repository:

```bash
docker images media
```

A formatted verification command:

```bash
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}" | grep media
```

---

## Expected Result

```text
REPOSITORY   TAG
media        devops
```

---

## Final Command Sequence

```bash
ssh tony@stapp01
docker ps
docker commit ubuntu_latest media:devops
docker images
```
