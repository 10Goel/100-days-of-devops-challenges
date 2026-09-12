# Day 44 — Docker Compose & Container Deployment Notes

## 1. What Is Docker Compose?

Docker Compose is used to define and manage containerized applications through a YAML configuration file.

Instead of maintaining long `docker run` commands, configuration can be declared in a Compose file.

Compose can define:

- Services
- Images
- Containers
- Ports
- Volumes
- Networks
- Environment variables
- Dependencies
- Restart policies

---

## 2. Docker Run vs Docker Compose

The Day 44 deployment could be represented by a long Docker command:

```bash
docker run -d \
  --name httpd \
  -p 6300:80 \
  -v /opt/dba:/usr/local/apache2/htdocs \
  httpd:latest
```

Compose expresses the same configuration declaratively:

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

Benefits include:

- Readability
- Repeatability
- Version control
- Easier maintenance
- Easier scaling to multiple services

---

## 3. Compose File Structure

Basic structure:

```yaml
services:
  service-name:
    image: image-name
```

Day 44:

```yaml
services:
  httpd:
    image: httpd:latest
```

Here:

```text
services → top-level service collection
httpd    → service name
image    → container image
```

---

## 4. Service Name vs Container Name

```yaml
services:
  httpd:
    container_name: httpd
```

### Service name

```text
httpd
```

Identifies the service within the Compose project.

### Container name

```text
httpd
```

Explicitly names the Docker container.

This allows commands such as:

```bash
docker logs httpd
docker inspect httpd
docker stop httpd
```

---

## 5. YAML and Indentation

YAML is indentation-sensitive.

Correct:

```yaml
services:
  httpd:
    image: httpd:latest
    ports:
      - "6300:80"
```

Best practices:

- Use spaces, not tabs
- Keep indentation consistent
- Quote port mappings
- Group related configuration logically

---

## 6. The `image` Property

```yaml
image: httpd:latest
```

means:

```text
Repository/Image → httpd
Tag              → latest
```

The image contains Apache HTTPD and its required runtime environment.

For production environments, using a specific version or immutable digest can provide more reproducible deployments than a moving `latest` tag.

---

## 7. Port Publishing

Day 44 uses:

```yaml
ports:
  - "6300:80"
```

The syntax is:

```text
HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
Host :6300
   ↓
Docker port publishing
   ↓
Container :80
   ↓
Apache HTTPD
```

A request to:

```text
http://localhost:6300
```

reaches Apache on port 80 inside the container.

---

## 8. Why Apache Uses Port 80

HTTP commonly uses:

```text
TCP port 80
```

Apache HTTPD listens on port 80 in the container.

The host can publish it through a different port:

```text
Host 6300 → Container 80
```

This is useful when the host's port 80 is already occupied or when several services need different host ports.

---

## 9. Bind Mounts

Day 44 uses:

```yaml
volumes:
  - /opt/dba:/usr/local/apache2/htdocs
```

This is a Docker **bind mount**.

Syntax:

```text
HOST_PATH:CONTAINER_PATH
```

Therefore:

```text
Host:
 /opt/dba

        ↓

Container:
 /usr/local/apache2/htdocs
```

---

## 10. Apache Document Root

The official HTTPD image uses:

```text
/usr/local/apache2/htdocs
```

as its standard document root.

Mounting:

```text
/opt/dba
```

there means Apache can serve the existing website content stored on the host.

---

## 11. Bind Mount vs Named Volume

### Bind mount

```yaml
- /opt/dba:/usr/local/apache2/htdocs
```

The source is an explicit host filesystem path.

### Named volume

Example:

```yaml
- website_data:/usr/local/apache2/htdocs
```

The storage is managed by Docker.

Key difference:

```text
Bind mount  → Host path controlled by operator
Named volume → Storage managed by Docker
```

Day 44 specifically requires a bind mount because the website data already exists at `/opt/dba`.

---

## 12. Existing Data Preservation

The challenge explicitly provides data in:

```text
/opt/dba
```

The correct approach is to mount that directory.

Do not:

```bash
rm -rf /opt/dba/*
```

Do not overwrite its contents.

The container should consume the existing data through the mount.

---

## 13. `docker compose up`

Command:

```bash
docker compose up -d
```

Purpose:

- Reads the Compose configuration
- Pulls required images if necessary
- Creates required containers
- Configures ports
- Configures volumes
- Starts services

The `-d` flag starts services in detached mode.

---

## 14. `docker compose down`

Command:

```bash
docker compose down
```

Typically:

- Stops Compose-managed containers
- Removes those containers
- Removes the Compose-created network when applicable

It does not mean all Docker images are deleted.

---

## 15. Configuration Validation

Use:

```bash
docker compose config
```

before deployment when practical.

It helps identify:

- YAML errors
- Incorrect indentation
- Invalid Compose configuration
- Incorrectly resolved values

Validation becomes increasingly useful as Compose files grow.

---

## 16. Container Verification

Check running containers:

```bash
docker ps
```

Check all containers:

```bash
docker ps -a
```

Check Compose services:

```bash
docker compose ps
```

For Day 44, the important container is:

```text
httpd
```

with:

```text
6300->80
```

---

## 17. Application-Level Verification

A running container alone is not enough to prove the application works.

Use:

```bash
curl http://localhost:6300
```

This verifies the complete path:

```text
HTTP Client
    ↓
Host :6300
    ↓
Docker Port Publishing
    ↓
Container :80
    ↓
Apache HTTPD
    ↓
Document Root
    ↓
Mounted /opt/dba content
```

---

## 18. Troubleshooting Workflow

If the service does not respond, troubleshoot layer by layer.

### Container status

```bash
docker ps -a
```

### Compose status

```bash
docker compose ps
```

### Logs

```bash
docker logs httpd
```

### Port mapping

```bash
docker port httpd
```

### Mount configuration

```bash
docker inspect httpd
```

### Host content

```bash
ls -la /opt/dba
```

### HTTP test

```bash
curl http://localhost:6300
```

This avoids guessing and isolates the failing layer.

---

## 19. Common Mistakes

### Reversing port mapping

Incorrect:

```yaml
- "80:6300"
```

Correct:

```yaml
- "6300:80"
```

Remember:

```text
HOST:CONTAINER
```

### Reversing volume mapping

Incorrect:

```yaml
- /usr/local/apache2/htdocs:/opt/dba
```

Correct:

```yaml
- /opt/dba:/usr/local/apache2/htdocs
```

Remember:

```text
HOST_PATH:CONTAINER_PATH
```

### Using the wrong image

Required:

```yaml
image: httpd:latest
```

### Modifying `/opt/dba`

The existing content must remain intact.

### Running from the wrong directory

When relying on the default Compose filename, use:

```bash
cd /opt/docker
```

---

## 20. Compose and Version Control

A Compose file is well suited to Git because it describes infrastructure configuration as code.

Example:

```text
Day-44/
├── README.md
├── commands.md
├── notes.md
└── docker-compose.yml
```

Benefits:

- Change tracking
- Peer review
- Reproducibility
- Easier environment recreation
- Infrastructure documentation

This is an important foundation for Infrastructure as Code practices.

---

## 21. Docker Compose to Kubernetes

The concepts learned through Compose transfer well into Kubernetes.

| Docker Compose | Kubernetes concept |
|---|---|
| Image | Container image |
| Service definition | Workload configuration |
| Ports | Container/Service ports |
| Volumes | Volumes / Persistent Volumes |
| Environment variables | ConfigMaps / Secrets |
| Multiple services | Multiple workloads |
| Compose configuration | Kubernetes manifests |

The tools differ, but the underlying container concepts remain important.

---

## 22. DevOps Takeaways

Day 44 combines:

```text
Declarative Configuration
        ↓
Docker Compose
        ↓
Apache HTTPD
        ↓
Port Publishing
        ↓
Bind Mount
        ↓
Service Verification
```

The major lesson is that container deployments should be:

- Explicit
- Repeatable
- Versionable
- Testable
- Easy to troubleshoot

---

## 23. Quick Revision

### Compose file

```text
/opt/docker/docker-compose.yml
```

### Image

```text
httpd:latest
```

### Container

```text
httpd
```

### Host port

```text
6300
```

### Container port

```text
80
```

### Bind mount

```text
/opt/dba
        ↓
/usr/local/apache2/htdocs
```

### Start

```bash
docker compose up -d
```

### Verify

```bash
docker compose ps
docker ps
curl http://localhost:6300
```

### Stop/remove Compose resources

```bash
docker compose down
```

---

## Day 44 Summary

**Primary concepts:**

- Docker Compose
- YAML configuration
- Apache HTTPD
- Service definitions
- Container naming
- Port publishing
- Bind mounts
- Apache document root
- Existing host data
- Compose lifecycle
- Configuration validation
- Service verification
- Container troubleshooting
- Infrastructure-as-Code fundamentals

**Status: Completed Successfully**
