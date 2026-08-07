# Commands Used - Day 9

---

## 1. SSH into Database Server

```bash
ssh peter@stdb01
```

### Why?

Connects to the Database Server where MariaDB is running.

---

## 2. Become Root User

```bash
sudo su -
```

### Why?

Administrative privileges are required to manage system services and initialize databases.

---

## 3. Check Service Status

```bash
systemctl status mariadb
```

### Why?

Displays:

- Current service state
- Whether service is active
- Whether service failed
- Error summary
- Service startup information

This is always the first command to run when troubleshooting a service.

---

## 4. Attempt to Start MariaDB

```bash
systemctl start mariadb
```

### Why?

Starts the MariaDB database service.

In this challenge it failed, confirming there was an underlying issue.

---

## 5. Enable MariaDB Service

```bash
systemctl enable mariadb
```

### Why?

Configures MariaDB to start automatically after every system boot.

Note:

This command does NOT start the service immediately.

It only enables automatic startup.

---

## 6. View Detailed Logs

```bash
journalctl -xeu mariadb.service
```

### Why?

This is the most important troubleshooting command.

Options:

- x → Shows additional explanations.
- e → Jumps directly to the latest log entries.
- u mariadb.service → Displays logs only for MariaDB.

The logs revealed the actual root cause:

```
Database MariaDB is not initialized
```

Without checking logs, identifying the issue would have been difficult.

---

## 7. Check Data Directory

```bash
ls -la /var/lib/mysql
```

### Why?

Checks whether the MariaDB data directory exists and whether it contains initialized database files.

Expected files normally include:

```
mysql/
performance_schema/
ibdata1
aria_log*
```

In this challenge:

```
No such file or directory
```

which confirmed the database had never been initialized.

---

## 8. Check Directory Size

```bash
du -sh /var/lib/mysql
```

### Why?

Displays the total size of the MariaDB data directory.

Useful to determine whether the directory is empty, contains data, or does not exist.

---

## 9. Verify Installed Packages

```bash
rpm -qa | grep mariadb
```

### Why?

Checks whether MariaDB packages are installed.

This confirmed that:

- mariadb-server
- mariadb
- mariadb-common

were already installed.

Therefore, installation was NOT the problem.

---

## 10. Verify Server Files

```bash
rpm -ql mariadb-server | head -20
```

### Why?

Lists files installed by the mariadb-server package.

Confirmed that the server installation and the `mariadb-install-db` utility were present.

---

## 11. Initialize MariaDB Database

```bash
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

### Why?

Creates the initial MariaDB system database and data directory.

This command:

- creates `/var/lib/mysql`
- creates system tables
- creates InnoDB files
- prepares MariaDB for first startup

This command should ONLY be used when the database has never been initialized.

Do NOT run it on an existing production database.

---

## 12. Start MariaDB Again

```bash
systemctl start mariadb
```

### Why?

Starts MariaDB after successful initialization.

This time the service started successfully.

---

## 13. Verify Service

```bash
systemctl status mariadb
```

or

```bash
systemctl is-active mariadb
```

### Why?

Confirms the service is running correctly.

Expected output:

```
active (running)
```

or

```
active
```

This is the final verification step after fixing the issue.
