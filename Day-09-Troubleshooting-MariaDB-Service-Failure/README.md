# Day 9 - Troubleshooting MariaDB Service Failure

## 📌 Challenge Objective

The Nautilus application was unable to connect to its database because the MariaDB service on the Database Server was down.

The objective was to investigate the root cause of the service failure, fix the issue, and restore the MariaDB service.

---

## Infrastructure

| Server | Hostname | Purpose |
|---------|----------|---------|
| Database Server | stdb01 | Hosts MariaDB Database |

---

## Problem Statement

The MariaDB service failed to start.

Instead of blindly restarting the service, the task required identifying the actual reason behind the failure and resolving it.

---

## Root Cause

After examining the service logs, it was discovered that:

- MariaDB server package was installed.
- The MariaDB data directory (`/var/lib/mysql`) did not exist.
- Since the database had never been initialized, MariaDB could not start.

Error from logs:

```
Database MariaDB is not initialized
Make sure the /var/lib/mysql is empty before running mariadb-install-db
```

---

## Solution

Initialized the MariaDB database using:

```bash
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

Started the MariaDB service:

```bash
systemctl start mariadb
```

Verified the service status:

```bash
systemctl status mariadb
```

---

## Result

✔ MariaDB initialized successfully.

✔ Database service started successfully.

✔ Challenge completed successfully.

---

## Concepts Learned

- Linux service troubleshooting
- Reading systemd logs
- Using journalctl
- MariaDB initialization
- Database data directory
- Root cause analysis
- Difference between installation and initialization
- systemctl service management
