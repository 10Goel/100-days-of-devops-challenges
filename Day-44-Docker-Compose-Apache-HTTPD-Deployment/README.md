# Day 44 — Docker Compose: Apache HTTPD Deployment

## KodeKloud 100 Days of DevOps

**Day:** 44  
**Focus:** Docker Compose, Apache HTTPD, port publishing, and bind mounts  
**Environment:** Stratos Datacenter — Application Server 2 (`stapp02`)

## Objective

Deploy an Apache HTTPD web server using Docker Compose.

Requirements:

- Create `/opt/docker/docker-compose.yml`
- Use `httpd:latest`
- Create a container named `httpd`
- Map host port `6300` to container port `80`
- Mount `/opt/dba` to `/usr/local/apache2/htdocs`
- Preserve the existing data in `/opt/dba`

## Final Compose Configuration

```yaml
services:
  httpd:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6300:80"
    volumes:
      - /opt/dba:/usr/local/apache2/htdocs
```

## Configuration Breakdown

| Setting | Purpose |
|---|---|
| `services` | Defines application services managed by Compose |
| `httpd` | Compose service name |
| `image: httpd:latest` | Apache HTTPD image |
| `container_name: httpd` | Explicit container name |
| `6300:80` | Host port 6300 → container port 80 |
| `/opt/dba:/usr/local/apache2/htdocs` | Bind mount for existing website content |

## Deployment

### 1. Create the Docker directory

```bash
sudo mkdir -p /opt/docker
```

### 2. Verify existing data

```bash
sudo ls -la /opt/dba
```

The existing content was preserved and not modified.

### 3. Create the Compose file

```bash
sudo vi /opt/docker/docker-compose.yml
```

Paste:

```yaml
services:
  httpd:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6300:80"
    volumes:
      - /opt/dba:/usr/local/apache2/htdocs
```

### 4. Start the service

```bash
cd /opt/docker
sudo docker compose up -d
```

### 5. Verify the container

```bash
sudo docker ps
```

Expected port mapping:

```text
0.0.0.0:6300->80/tcp
```

### 6. Test Apache

```bash
curl http://localhost:6300
```

The response verifies that Apache is serving the existing content through the published port.

## Architecture

```text
                 Application Server 2
                       stapp02
                          |
                    Host :6300
                          |
                          v
                 +----------------+
                 | Docker Engine  |
                 +----------------+
                          |
                          v
                 +----------------+
                 | httpd Container|
                 |     :80        |
                 +----------------+
                          |
                          v
             /usr/local/apache2/htdocs
                          ^
                          |
                     Bind Mount
                          |
                       /opt/dba
```

## Key Concepts

### Docker Compose

Compose defines container configuration declaratively in YAML, making deployments repeatable and version-controllable.

### Port Publishing

```yaml
ports:
  - "6300:80"
```

means:

```text
Host port 6300 → Container port 80
```

### Bind Mount

```yaml
volumes:
  - /opt/dba:/usr/local/apache2/htdocs
```

maps a host directory directly into the container.

### Apache Document Root

For the official HTTPD image, Apache serves website content from:

```text
/usr/local/apache2/htdocs
```

## Verification Checklist

- [x] `/opt/docker/docker-compose.yml` created
- [x] `httpd:latest` configured
- [x] Container named `httpd`
- [x] Port `6300` mapped to container port `80`
- [x] `/opt/dba` mounted to Apache document root
- [x] Existing `/opt/dba` data preserved
- [x] Container started successfully
- [x] Apache response verified
- [x] Day 44 completed successfully

## DevOps Relevance

This challenge bridges individual Docker containers and multi-service container orchestration:

```text
Docker
  ↓
Docker Compose
  ↓
Multi-container applications
  ↓
CI/CD
  ↓
Kubernetes
```

The concepts practiced here—declarative configuration, networking, volumes, service lifecycle, and verification—are foundational for modern containerized infrastructure.

**Status: Completed Successfully**
