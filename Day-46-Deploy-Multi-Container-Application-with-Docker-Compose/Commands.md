# Day 46 - Commands Reference

## 1. Connect to App Server 2

```bash
ssh steve@stapp02
```

---

## 2. Check Docker

```bash
sudo docker --version
```

Check Docker Compose:

```bash
sudo docker compose version
```

If the environment uses the legacy command:

```bash
sudo docker-compose --version
```

---

## 3. Inspect Existing Containers and Ports

```bash
sudo docker ps -a
```

```bash
sudo ss -lntp | grep -E ':6100|:3306'
```

---

## 4. Create the Required Directory

```bash
sudo mkdir -p /opt/finance
```

```bash
cd /opt/finance
```

```bash
pwd
```

Expected:

```text
/opt/finance
```

---

## 5. Create the Docker Compose File

```bash
sudo vi /opt/finance/docker-compose.yml
```

Compose configuration used:

```yaml
services:
  web:
    image: php:8.2-apache
    container_name: php_host
    ports:
      - "6100:80"
    volumes:
      - /var/www/html:/var/www/html

  db:
    image: mariadb:latest
    container_name: mysql_host
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: appuser
      MYSQL_PASSWORD: "Str0ngDB_Pass#2026"
      MYSQL_ROOT_PASSWORD: "R00tDB_Pass#2026"
```

---

## 6. Verify the File

```bash
sudo cat /opt/finance/docker-compose.yml
```

---

## 7. Validate the Compose Configuration

```bash
sudo docker compose -f /opt/finance/docker-compose.yml config
```

Legacy alternative:

```bash
sudo docker-compose -f /opt/finance/docker-compose.yml config
```

---

## 8. Inspect the Web Root

```bash
sudo ls -la /var/www/html
```

---

## 9. Start the Application Stack

```bash
cd /opt/finance
```

```bash
sudo docker compose up -d
```

Legacy alternative:

```bash
sudo docker-compose up -d
```

---

## 10. Check Container Status

```bash
sudo docker ps
```

```bash
sudo docker compose -f /opt/finance/docker-compose.yml ps
```

---

## 11. Check Database Logs

```bash
sudo docker logs mysql_host --tail 30
```

Check only the database container:

```bash
sudo docker ps --filter name=mysql_host
```

---

## 12. Test the Web Application

```bash
curl http://localhost:6100/
```

```bash
curl http://stapp02:6100/
```

For verbose troubleshooting:

```bash
curl -v http://localhost:6100/
```

---

## 13. Verify Port Mappings

```bash
sudo docker port php_host
```

```bash
sudo docker port mysql_host
```

---

## 14. Verify Bind Mounts

Web container:

```bash
sudo docker inspect php_host --format '{{json .Mounts}}'
```

Database container:

```bash
sudo docker inspect mysql_host --format '{{json .Mounts}}'
```

---

## 15. Verify Database Environment Variables

```bash
sudo docker inspect mysql_host --format '{{range .Config.Env}}{{println .}}{{end}}'
```

---

## 16. Verify Database Initialization

```bash
sudo docker exec mysql_host mariadb   -uappuser   -p'Str0ngDB_Pass#2026'   -e "SHOW DATABASES;"
```

Expected database:

```text
database_host
```

---

## Useful Compose Commands

### Start

```bash
sudo docker compose up -d
```

### Status

```bash
sudo docker compose ps
```

### Logs

```bash
sudo docker compose logs
```

### Follow logs

```bash
sudo docker compose logs -f
```

### Stop

```bash
sudo docker compose stop
```

### Start stopped services

```bash
sudo docker compose start
```

### Restart

```bash
sudo docker compose restart
```

### Remove the stack

```bash
sudo docker compose down
```

### View generated Compose configuration

```bash
sudo docker compose config
```

---

## Final Verification Commands

```bash
sudo docker compose -f /opt/finance/docker-compose.yml config
sudo docker ps
curl http://localhost:6100/
sudo docker port php_host
sudo docker port mysql_host
```
