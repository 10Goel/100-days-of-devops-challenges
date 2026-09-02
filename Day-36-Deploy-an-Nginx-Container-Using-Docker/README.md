# Day 36 — Deploy an Nginx Container Using Docker

## Challenge Overview

Day 36 of the **KodeKloud 100 Days of DevOps Challenge** focused on deploying and verifying a Docker container.

### Task Requirements
On **Application Server 3 (`stapp03`)**:

- Create a container named `nginx_3`
- Use the image `nginx:alpine`
- Ensure the container remains in the **running** state

## Environment

| Item | Details |
|---|---|
| Server | Application Server 3 |
| Hostname | `stapp03` |
| User | `banner` |
| Container | `nginx_3` |
| Image | `nginx:alpine` |
| Required State | Running |

## Implementation

### Connect to the server

```bash
ssh banner@stapp03
```

### Verify Docker

```bash
docker --version
```

### Create and start the container

```bash
docker run -d --name nginx_3 nginx:alpine
```

Docker automatically pulled the image because it was not initially available locally.

### Verify the container

```bash
docker ps
```

An additional filtered check was performed:

```bash
docker ps --filter "name=nginx_3"
```

The verification confirmed:

```text
Container Name: nginx_3
Image: nginx:alpine
Status: Up / Running
```

## Command Breakdown

```bash
docker run -d --name nginx_3 nginx:alpine
```

| Option | Purpose |
|---|---|
| `docker run` | Creates and starts a container |
| `-d` | Runs the container in detached mode |
| `--name nginx_3` | Assigns a custom container name |
| `nginx:alpine` | Specifies the image and Alpine tag |

## Key Learnings

- A container is a running instance of an image.
- `docker run` can pull, create, and start a container.
- `-d` is commonly used for long-running services.
- Container names simplify management.
- `docker ps` displays running containers.
- Image tags identify a specific version or variant.
- Alpine-based images are lightweight and commonly used for container deployments.

## Result

**Day 36 completed successfully. ✅**

The required Nginx container was successfully deployed on Application Server 3 using the `nginx:alpine` image and verified to be running.

**Progress: Day 36 / 100**
