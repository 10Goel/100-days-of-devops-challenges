# Day 44 — Docker Compose Commands

## Environment

**Server:** Application Server 2 (`stapp02`)  
**User:** `steve`

## 1. Create Docker Directory

```bash
sudo mkdir -p /opt/docker
```

## 2. Verify Existing Data

```bash
sudo ls -la /opt/dba
```

Do not delete, replace, or otherwise modify the existing data.

## 3. Create Compose File

```bash
sudo vi /opt/docker/docker-compose.yml
```

Configuration:

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

## 4. Navigate to Compose Directory

```bash
cd /opt/docker
```

## 5. Validate Configuration

```bash
sudo docker compose config
```

## 6. Start the Service

```bash
sudo docker compose up -d
```

## 7. Verify Running Containers

```bash
sudo docker ps
```

Expected:

```text
httpd
```

with:

```text
0.0.0.0:6300->80/tcp
```

## 8. Verify Compose Services

```bash
sudo docker compose ps
```

## 9. Test Apache

```bash
curl http://localhost:6300
```

## 10. View Logs

```bash
sudo docker logs httpd
```

or:

```bash
sudo docker compose logs httpd
```

Follow logs:

```bash
sudo docker compose logs -f httpd
```

## 11. Check Port Mapping

```bash
sudo docker port httpd
```

Expected:

```text
80/tcp -> 0.0.0.0:6300
```

## 12. Inspect Container

```bash
sudo docker inspect httpd
```

## 13. Verify Mount

```bash
sudo docker inspect httpd --format '{{json .Mounts}}'
```

Confirm:

```text
/opt/dba
```

maps to:

```text
/usr/local/apache2/htdocs
```

## 14. Stop and Remove Compose Resources

```bash
sudo docker compose down
```

## 15. Start Again

```bash
sudo docker compose up -d
```

## Essential Command Sequence

```bash
sudo mkdir -p /opt/docker
sudo ls -la /opt/dba
sudo vi /opt/docker/docker-compose.yml
cd /opt/docker
sudo docker compose config
sudo docker compose up -d
sudo docker ps
curl http://localhost:6300
```

## Important Syntax

### Port mapping

```yaml
ports:
  - "HOST_PORT:CONTAINER_PORT"
```

Day 44:

```yaml
ports:
  - "6300:80"
```

### Bind mount

```yaml
volumes:
  - HOST_PATH:CONTAINER_PATH
```

Day 44:

```yaml
volumes:
  - /opt/dba:/usr/local/apache2/htdocs
```
