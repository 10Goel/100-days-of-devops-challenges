# Day 46 - Deploy a Multi-Container Application with Docker Compose

## Overview

This challenge focused on deploying a small two-tier application stack on **App Server 2 (`stapp02`)** using **Docker Compose**.

The deployment consisted of:

- A **PHP + Apache web container**
- A **MariaDB database container**
- Host-to-container port mappings
- Persistent bind mounts for web and database data
- MariaDB environment variables for database initialization

The complete stack was defined in a single Compose file located at:

```text
/opt/finance/docker-compose.yml
```

---

## Task Requirements

### Web Service

| Requirement | Value |
|---|---|
| Container name | `php_host` |
| Image | `php` with an Apache tag |
| Host port | `6100` |
| Container port | `80` |
| Host volume | `/var/www/html` |
| Container volume | `/var/www/html` |

### Database Service

| Requirement | Value |
|---|---|
| Container name | `mysql_host` |
| Image | `mariadb:latest` |
| Host port | `3306` |
| Container port | `3306` |
| Host volume | `/var/lib/mysql` |
| Container volume | `/var/lib/mysql` |
| Database | `database_host` |
| User | Custom non-root user |
| Password | Strong custom password |

---

## Architecture

```text
                        App Server 2
                          stapp02

                            :6100
                              |
                              v
                    +-------------------+
                    |     php_host      |
                    |   PHP + Apache    |
                    |      Port 80      |
                    +-------------------+
                              |
                     Docker Compose Network
                              |
                              v
                    +-------------------+
                    |    mysql_host     |
                    |      MariaDB      |
                    |     Port 3306     |
                    +-------------------+

Bind Mounts:

/var/www/html   <---->   php_host:/var/www/html
/var/lib/mysql  <---->   mysql_host:/var/lib/mysql
```

---

## Docker Compose Configuration

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

> The passwords shown above are lab-only example credentials and should not be reused in production.

---

## Implementation Summary

1. Connected to **App Server 2 (`stapp02`)**.
2. Created the required directory `/opt/finance`.
3. Created `/opt/finance/docker-compose.yml`.
4. Defined `web` and `db` services.
5. Configured the PHP/Apache container as `php_host`.
6. Published host port `6100` to container port `80`.
7. Mounted `/var/www/html` into the web container.
8. Configured the MariaDB container as `mysql_host`.
9. Published port `3306`.
10. Mounted `/var/lib/mysql` for database persistence.
11. Configured the MariaDB database and non-root user.
12. Validated the Compose configuration.
13. Started the stack in detached mode.
14. Verified both containers were running.
15. Tested the web service using `curl`.
16. Verified the MariaDB container and database initialization.

---

## Validation

### Verify Containers

```bash
sudo docker ps
```

Expected container names:

```text
php_host
mysql_host
```

### Verify Compose Stack

```bash
sudo docker compose -f /opt/finance/docker-compose.yml ps
```

### Test Web Service

```bash
curl http://localhost:6100/
```

or

```bash
curl http://stapp02:6100/
```

### Verify Port Publishing

```bash
sudo docker port php_host
sudo docker port mysql_host
```

Expected mappings:

```text
80/tcp   -> 0.0.0.0:6100
3306/tcp -> 0.0.0.0:3306
```

---

## Key Concepts Practiced

- Docker Compose
- Multi-container applications
- Service definitions
- Container naming
- Port publishing
- Bind mounts
- Environment variables
- MariaDB initialization
- Container logs
- Compose validation
- Detached deployments
- Application connectivity testing

---

## Useful Docker Compose Lifecycle Commands

Start the stack:

```bash
sudo docker compose up -d
```

Show status:

```bash
sudo docker compose ps
```

View logs:

```bash
sudo docker compose logs
```

Stop containers without deleting them:

```bash
sudo docker compose stop
```

Start stopped containers:

```bash
sudo docker compose start
```

Remove the Compose stack:

```bash
sudo docker compose down
```

> `docker compose down` should only be used when removing the stack is intended.

---

## Result

The Day 46 challenge was completed successfully.

A complete PHP/Apache + MariaDB application stack was deployed using Docker Compose with the required container names, ports, volumes, database configuration, and validation checks.

---

## Learning Outcome

This challenge demonstrated why Docker Compose is useful when an application requires multiple related containers. Instead of managing the web and database containers independently using separate `docker run` commands, the entire deployment can be declared in one YAML file and managed as a single application stack.

```bash
sudo docker compose up -d
```

This makes multi-container deployments more repeatable, readable, and maintainable.
