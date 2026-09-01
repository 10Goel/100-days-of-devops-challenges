# Day 35 — Install Docker CE and Docker Compose

## 📌 Challenge Overview

As part of the **KodeKloud 100 Days of DevOps Challenge**, Day 35 focused on preparing **Application Server 3 (`stapp03`)** for containerized application testing.

The task required:

1. Installing **Docker CE** on Application Server 3.
2. Installing **Docker Compose**.
3. Starting the Docker service.

---

## 🖥️ Infrastructure Details

| Item | Details |
|---|---|
| Server | Application Server 3 |
| Hostname | `stapp03` |
| User | `banner` |
| Environment | Nautilus / Stratos Datacenter |
| Docker | Docker CE |
| Compose | Docker Compose V2 plugin |

---

## 🎯 Objectives

- Connect to `stapp03`.
- Install Docker CE and its required components.
- Configure the Docker package repository.
- Install Docker Compose.
- Start and enable the Docker service.
- Verify that Docker and Compose are working correctly.

---

## 🛠️ Implementation

### 1. Connect to Application Server 3

```bash
ssh banner@stapp03
```

Switch to root:

```bash
sudo -i
```

### 2. Install repository utilities

```bash
yum install -y yum-utils
```

### 3. Add the Docker CE repository

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Refresh package metadata:

```bash
yum makecache
```

### 4. Install Docker CE

```bash
yum install -y docker-ce docker-ce-cli containerd.io
```

Verify Docker:

```bash
docker --version
```

### 5. Install Docker Compose

The server uses the modern Docker Compose V2 plugin. Therefore, Compose is invoked with a space rather than a hyphen.

```bash
yum install -y docker-compose-plugin
```

Verify:

```bash
docker compose version
```

> **Note:** `docker-compose` is the legacy standalone command. On this server, the supported command is `docker compose`.

### 6. Start and enable Docker

```bash
systemctl start docker
systemctl enable docker
```

Verify the service:

```bash
systemctl is-active docker
```

Expected:

```text
active
```

---

## 🔍 Verification

The following checks were performed:

```bash
docker --version
docker compose version
systemctl is-active docker
```

Docker was successfully installed and the Docker service was running.

The **KodeKloud Day 35 challenge was successfully completed**.

---

## 🧠 Key Learnings

- Docker CE provides the Docker Engine required to run containers.
- `containerd.io` is a core container runtime dependency used by Docker.
- Docker Compose V2 is integrated with Docker as a CLI plugin.
- Modern Compose uses:

```bash
docker compose
```

instead of the legacy:

```bash
docker-compose
```

- `systemctl enable docker` ensures Docker starts automatically after a system reboot.
- `systemctl is-active docker` provides a quick way to verify whether the service is running.

---

## ⚠️ Troubleshooting Encountered

The legacy package was initially checked with:

```bash
yum install -y docker-compose
```

and the repository returned:

```text
No match for argument: docker-compose
```

The `docker-compose` command was therefore unavailable.

The correct modern approach was to install the Compose V2 plugin:

```bash
yum install -y docker-compose-plugin
```

and verify it using:

```bash
docker compose version
```

This distinction is important when working with newer Docker installations.

---

## ✅ Result

**Status: COMPLETED ✅**

Application Server 3 was successfully prepared with:

- Docker CE
- Docker Compose V2
- Running Docker service

**Day 35 / 100 completed successfully.**

---

## 📚 Skills Practiced

- Linux package management
- Docker installation
- Docker repository configuration
- Docker service management with systemd
- Docker Compose V2
- Linux service verification
- Troubleshooting package availability
- Container infrastructure preparation

---

## 👨‍💻 Challenge Progress

**KodeKloud 100 Days of DevOps**

`Day 35 / 100 — Completed ✅`
