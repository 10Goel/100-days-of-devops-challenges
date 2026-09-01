# Day 35 — Notes

## 📖 Topic

**Installing and configuring Docker CE and Docker Compose on Linux**

---

## 1. Docker CE

Docker Community Edition (Docker CE) is the commonly used Docker distribution for running containers.

Important components installed during the challenge:

- `docker-ce` — Docker Engine
- `docker-ce-cli` — Docker command-line interface
- `containerd.io` — Container runtime used by Docker

Check Docker installation:

```bash
docker --version
```

---

## 2. Docker Repository

Instead of relying only on the default OS repositories, the Docker CE repository was configured.

Install repository utilities:

```bash
yum install -y yum-utils
```

Add the Docker repository:

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Refresh metadata:

```bash
yum makecache
```

---

## 3. Docker Compose V2

Docker Compose is used to define and run multi-container applications.

Modern Docker installations use **Compose V2**, which is invoked as a Docker CLI subcommand:

```bash
docker compose
```

For example:

```bash
docker compose version
```

The older standalone syntax is:

```bash
docker-compose
```

### Important Difference

| Legacy | Modern |
|---|---|
| `docker-compose` | `docker compose` |
| Standalone binary | Docker CLI plugin |
| Older Compose implementation | Compose V2 |

---

## 4. Docker Service Management

Start Docker:

```bash
systemctl start docker
```

Enable Docker at boot:

```bash
systemctl enable docker
```

Check status:

```bash
systemctl status docker
```

Quick check:

```bash
systemctl is-active docker
```

Expected:

```text
active
```

---

## 5. Troubleshooting

### Problem

The legacy package was unavailable:

```bash
yum install -y docker-compose
```

Result:

```text
No match for argument: docker-compose
```

### Resolution

Use the Docker Compose V2 plugin:

```bash
yum install -y docker-compose-plugin
```

Then verify:

```bash
docker compose version
```

---

## 6. Useful Verification Commands

Docker version:

```bash
docker --version
```

Compose version:

```bash
docker compose version
```

Docker service:

```bash
systemctl is-active docker
```

Detailed service status:

```bash
systemctl status docker
```

---

## 💡 Key Takeaways

1. Docker Engine and Docker Compose are related but separate components.
2. Docker CE requires supporting components such as the Docker CLI and container runtime.
3. Docker Compose V2 is normally used through `docker compose`.
4. A missing `docker-compose` command does not necessarily mean Docker is broken.
5. Always check whether the system uses the modern Compose plugin.
6. `systemctl enable` configures a service to start automatically during boot.
7. `systemctl is-active` is useful for a quick service health check.

---

## 🏁 Completion

**Day 35 completed successfully. ✅**
