# Day 46 - Docker Compose Notes

## 1. What is Docker Compose?

Docker Compose is a tool used to define and manage **multi-container applications** using a YAML configuration file.

Instead of starting every container manually:

```bash
docker run ...
docker run ...
```

we describe all required services in a Compose file and start the complete stack with:

```bash
docker compose up -d
```

This is especially useful when an application contains multiple components such as:

- Web server
- Database
- Cache
- Message broker
- Backend API

---

## 2. Services in Docker Compose

In this challenge, the Compose file contains two services:

```yaml
services:
  web:
  db:
```

A **service** is a logical definition of how a container should be created and configured.

The service names in this task are:

```text
web
db
```

The actual container names are explicitly set using:

```yaml
container_name:
```

For example:

```yaml
container_name: php_host
```

and:

```yaml
container_name: mysql_host
```

Therefore:

```text
Service name    Container name
------------    --------------
web             php_host
db              mysql_host
```

---

## 3. Web Service

The web service uses:

```yaml
image: php:8.2-apache
```

This image provides both:

- PHP runtime
- Apache HTTP Server

The container is named:

```text
php_host
```

---

## 4. Port Mapping

The web service uses:

```yaml
ports:
  - "6100:80"
```

Docker port mapping syntax is:

```text
HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
6100:80
```

means:

```text
Host stapp02:6100
        |
        v
php_host:80
```

Apache listens on port `80` inside the container, while users connect to port `6100` on the host.

The database mapping:

```yaml
- "3306:3306"
```

maps host TCP port `3306` to MariaDB TCP port `3306` inside `mysql_host`.

---

## 5. Bind Mounts

The web container uses:

```yaml
volumes:
  - /var/www/html:/var/www/html
```

Syntax:

```text
HOST_PATH:CONTAINER_PATH
```

This means the host directory:

```text
/var/www/html
```

is accessible inside the container at:

```text
/var/www/html
```

This allows the web content to remain outside the container lifecycle.

---

## 6. Database Persistence

MariaDB uses:

```yaml
volumes:
  - /var/lib/mysql:/var/lib/mysql
```

MariaDB stores database files inside:

```text
/var/lib/mysql
```

By mounting the host directory into the same location, database data can persist independently of the container.

Conceptually:

```text
Host filesystem
/var/lib/mysql
      |
      v
mysql_host
/var/lib/mysql
```

---

## 7. Environment Variables

The MariaDB container is initialized using:

```yaml
environment:
  MYSQL_DATABASE: database_host
  MYSQL_USER: appuser
  MYSQL_PASSWORD: "..."
  MYSQL_ROOT_PASSWORD: "..."
```

### `MYSQL_DATABASE`

```text
database_host
```

Creates the requested database during initial container initialization.

### `MYSQL_USER`

Creates a custom non-root database user.

Using a non-root application account is better than allowing applications to connect as the MariaDB root user.

### `MYSQL_PASSWORD`

Sets the password for the custom user.

### `MYSQL_ROOT_PASSWORD`

Sets the MariaDB administrative/root password required by the image initialization process.

---

## 8. Docker Compose Default Network

No custom network was required for this challenge.

Docker Compose automatically creates a project network and connects services to it.

Conceptually:

```text
php_host
    |
    | Compose network
    |
mysql_host
```

Containers can communicate with each other using Compose service names.

For example, from the web service, the database service can normally be reached using:

```text
db:3306
```

because `db` is the Compose service name.

---

## 9. `docker compose up -d`

Command:

```bash
sudo docker compose up -d
```

`up` tells Compose to:

- Read the YAML file
- Create required containers
- Create the default network
- Configure ports
- Configure mounts
- Apply environment variables
- Start the services

`-d` means **detached mode**, so the stack runs in the background.

---

## 10. `docker compose config`

Before deploying, it is useful to run:

```bash
sudo docker compose config
```

This validates and normalizes the Compose YAML.

It is useful for detecting:

- Invalid YAML
- Indentation problems
- Incorrect Compose structure
- Unsupported configuration

---

## 11. `docker compose ps`

```bash
sudo docker compose ps
```

shows containers belonging to the Compose application.

For this challenge, the important containers are:

```text
php_host
mysql_host
```

Both should be running before validation.

---

## 12. `docker ps` vs `docker compose ps`

### `docker ps`

Shows all currently running Docker containers on the host.

```bash
sudo docker ps
```

### `docker compose ps`

Shows the containers associated with the current Compose project.

```bash
sudo docker compose ps
```

Both are useful, but `docker compose ps` is more application-stack focused.

---

## 13. Container Logs

For troubleshooting MariaDB:

```bash
sudo docker logs mysql_host
```

or:

```bash
sudo docker compose logs db
```

Logs help identify issues such as:

- Database initialization failure
- Invalid environment variables
- Permission issues
- Port conflicts
- Application startup errors

---

## 14. Testing the Application

The challenge required accessing the application through host port `6100`.

Example:

```bash
curl http://localhost:6100/
```

Request path:

```text
curl
  |
  v
stapp02:6100
  |
  v
Docker port publishing
  |
  v
php_host:80
  |
  v
Apache
```

---

## 15. Inspecting a Container

`docker inspect` provides detailed low-level configuration.

Example:

```bash
sudo docker inspect php_host
```

Useful information includes:

- Network configuration
- Mounts
- Environment variables
- Image
- Container state
- Port bindings

Focused mount inspection:

```bash
sudo docker inspect php_host --format '{{json .Mounts}}'
```

---

## 16. Important Compose Lifecycle Commands

### Deploy

```bash
docker compose up -d
```

### Check status

```bash
docker compose ps
```

### View logs

```bash
docker compose logs
```

### Stop

```bash
docker compose stop
```

### Start

```bash
docker compose start
```

### Restart

```bash
docker compose restart
```

### Remove containers and default network

```bash
docker compose down
```

A major difference is:

```text
docker compose stop
```

stops containers but keeps them.

Whereas:

```text
docker compose down
```

removes the Compose containers and project network.

---

## 17. Why Docker Compose is Better Than Multiple `docker run` Commands

Without Compose:

```text
docker run web...
docker run db...
configure ports...
configure volumes...
configure environment...
configure networking...
```

With Compose:

```text
docker-compose.yml
        |
        +-- web
        |
        +-- db
```

and deployment becomes:

```bash
docker compose up -d
```

Benefits include:

- Repeatability
- Infrastructure defined as code
- Easier multi-container management
- Cleaner configuration
- Easier troubleshooting
- Easier deployment and teardown

---

## 18. Production Security Note

Credentials should not normally be hardcoded directly inside a version-controlled Compose file.

For production environments, prefer mechanisms such as:

- `.env` files excluded from Git
- Docker secrets
- Kubernetes Secrets
- Cloud secret managers
- HashiCorp Vault

The passwords used in this challenge are lab credentials only.

---

## 19. Day 46 Key Takeaways

After completing this challenge, the important concepts to remember are:

```text
Docker Compose
      |
      +-- Services
      +-- Images
      +-- Containers
      +-- Ports
      +-- Bind mounts
      +-- Environment variables
      +-- Default networks
      +-- Logs
      +-- Validation
      +-- Application lifecycle
```

Most importantly, Compose allows an entire multi-container application stack to be described declaratively and managed as a single unit.
