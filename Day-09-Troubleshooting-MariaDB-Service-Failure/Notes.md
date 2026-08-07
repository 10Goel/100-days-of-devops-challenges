# Day 9 Notes - MariaDB Service Troubleshooting

# What is MariaDB?

MariaDB is an open-source relational database management system (RDBMS).

It stores application data in databases and is commonly used with web applications.

Applications communicate with MariaDB to store and retrieve information.

---

# What is systemd?

systemd is Linux's service manager.

It controls system services such as:

- MariaDB
- Nginx
- Apache
- Docker
- SSH

Common commands include:

```bash
systemctl start
systemctl stop
systemctl restart
systemctl status
systemctl enable
systemctl disable
```

---

# What is journalctl?

journalctl reads logs collected by the systemd journal.

It is the primary tool for diagnosing why services fail.

Useful commands:

```bash
journalctl -xeu mariadb.service
journalctl -u nginx
journalctl -b
```

---

# Difference Between Installing and Initializing MariaDB

Installing MariaDB:

- Installs the database software.
- Places binaries, libraries, configuration files, and utilities on the system.

Initialization:

- Creates the data directory.
- Creates system tables.
- Creates InnoDB files.
- Prepares the database for first use.

A server may be installed but still unusable until it has been initialized.

---

# MariaDB Data Directory

Default location:

```
/var/lib/mysql
```

It stores:

- User databases
- System tables
- InnoDB files
- Binary logs
- Aria logs
- Performance Schema
- Metadata

Deleting this directory prevents MariaDB from starting.

---

# Common Reasons MariaDB Fails to Start

1. Database not initialized
2. Missing data directory
3. Incorrect ownership
4. Incorrect permissions
5. Corrupted database files
6. Invalid configuration
7. Full disk
8. Port conflicts
9. SELinux restrictions
10. Missing dependencies

Always identify the root cause before applying a fix.

---

# Recommended Troubleshooting Workflow

Whenever a Linux service fails:

1. Check the service status.
2. Read the logs.
3. Verify configuration files.
4. Inspect required directories and files.
5. Check ownership and permissions.
6. Verify installed packages.
7. Check disk space and memory.
8. Apply the appropriate fix.
9. Verify the service is running.

This structured approach avoids unnecessary changes and helps preserve existing data.

---

# Key Takeaways

- Never assume the root cause of a service failure.
- Always inspect logs before attempting fixes.
- Distinguish between software installation and database initialization.
- Use `mariadb-install-db` only when a database has not been initialized.
- Validate the service status after every corrective action.
- Systematic troubleshooting is a core DevOps skill and minimizes downtime.
