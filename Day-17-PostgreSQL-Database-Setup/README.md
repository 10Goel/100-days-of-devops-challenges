# Day 17 — PostgreSQL Database Setup

## 📌 Overview

As part of the KodeKloud 100 Days of DevOps Challenge, Day 17 focused on preparing a PostgreSQL database server for a newly developed application in the Nautilus infrastructure.

The application requires PostgreSQL as its database backend. PostgreSQL was already installed on the Nautilus database server, so the task was to configure the required database user, database, and permissions **without restarting the PostgreSQL service**.

---

## 🎯 Objectives

The task required the following:

- Connect to the Nautilus PostgreSQL database server.
- Create a PostgreSQL user named `kodekloud_pop`.
- Configure the required password for the user.
- Create a database named `kodekloud_db7`.
- Grant full privileges on `kodekloud_db7` to `kodekloud_pop`.
- Verify the configuration.
- Do not restart the PostgreSQL service.

---

## 🖥️ Environment

| Component | Details |
|---|---|
| Infrastructure | Nautilus |
| Database Server | `stdb01` |
| SSH User | `peter` |
| Database | PostgreSQL |
| PostgreSQL Client | `psql` |
| PostgreSQL User | `kodekloud_pop` |
| Database Name | `kodekloud_db7` |
| PostgreSQL Service | Already installed and running |

> **Important:** The correct hostname is `stdb01`, not `stbd01`.

---

## 🔐 Configuration

The following PostgreSQL objects were created:

### PostgreSQL User

```text
kodekloud_pop
```

### Database

```text
kodekloud_db7
```

### Privileges

`kodekloud_pop` was granted full privileges on `kodekloud_db7`.

---

## 🛠️ Implementation

### 1. Connect to the database server

```bash
ssh peter@stdb01
```

### 2. Access PostgreSQL as the PostgreSQL administrator

```bash
sudo -u postgres psql
```

### 3. Create the application database user

```sql
CREATE USER kodekloud_pop WITH PASSWORD '<configured-password>';
```

### 4. Create the application database

```sql
CREATE DATABASE kodekloud_db7;
```

### 5. Grant database privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_pop;
```

---

## 🔍 Verification

### Verify PostgreSQL roles

```sql
\du
```

The output confirmed that `kodekloud_pop` exists.

### Verify databases and access privileges

```sql
\l
```

The output confirmed:

- `kodekloud_db7` exists.
- `kodekloud_pop` has access privileges on the database.

### Verify application-user connectivity

After leaving the PostgreSQL administrator session:

```bash
psql -U kodekloud_pop -d kodekloud_db7 -h localhost -W
```

The connection successfully opened:

```text
kodekloud_db7=>
```

This confirmed that the application user could authenticate and connect to the required database.

---

## ⚠️ Important Constraint

The task explicitly required:

> Do not restart PostgreSQL server service.

No PostgreSQL restart was performed because creating users, databases, and database privileges does not require a service restart.

---

## ✅ Result

Day 17 was completed successfully.

### Final State

```text
Database Server : stdb01
PostgreSQL User : kodekloud_pop
Database        : kodekloud_db7
Privileges      : Full database privileges
Connectivity    : Verified successfully
Service Restart: Not performed
```

---

## 📚 Key DevOps Concepts Learned

### PostgreSQL Roles

PostgreSQL uses roles for authentication and authorization. A role can act as a login user and can be granted different permissions.

### Database Creation

A PostgreSQL database provides an isolated logical environment for storing application data.

### Privilege Management

The `GRANT` command controls what a PostgreSQL role can do with database objects.

### Verification

Configuration should always be verified rather than assuming that a successful command means the entire setup is functional.

### Service Management

Not every configuration change requires restarting a service. Avoiding unnecessary restarts is especially important on production systems.

---

## 🏁 Challenge Status

**Day 17 — Completed Successfully ✅**

This task strengthened practical understanding of PostgreSQL administration, Linux server access, database user management, permissions, authentication, and verification.
