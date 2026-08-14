# Day 17 — Notes: PostgreSQL Database Setup

## 1. What was the task?

The Nautilus application development team needed a PostgreSQL database environment for a newly developed application.

PostgreSQL was already installed on the Nautilus database server. The task was therefore not to install PostgreSQL, but to configure the database environment required by the application.

The required objects were:

- PostgreSQL user: `kodekloud_pop`
- Database: `kodekloud_db7`
- Full database privileges for `kodekloud_pop`

The PostgreSQL service was **not supposed to be restarted**.

---

## 2. Server Details

The database server hostname used for this task was:

```text
stdb01
```

The SSH connection was:

```bash
ssh peter@stdb01
```

### Important correction

The hostname is:

```text
stdb01
```

It is **not**:

```text
stbd01
```

This distinction matters because SSH uses the exact hostname configured in the infrastructure.

---

## 3. PostgreSQL Architecture Used in the Task

The configuration involved three main components:

```text
Linux Server
    │
    └── PostgreSQL
          │
          ├── Role/User
          │     └── kodekloud_pop
          │
          └── Database
                └── kodekloud_db7
```

The application user authenticates to PostgreSQL and connects to the application database.

---

## 4. Creating a PostgreSQL User

The following command creates a PostgreSQL login role:

```sql
CREATE USER kodekloud_pop WITH PASSWORD '<configured-password>';
```

The `CREATE USER` command creates a role with login capability.

PostgreSQL internally treats users as roles. The `\du` command can be used to list them.

---

## 5. Creating the Database

The database was created with:

```sql
CREATE DATABASE kodekloud_db7;
```

A database is a logical container for application data and database objects.

---

## 6. Granting Privileges

The application user needed full privileges on the database.

The command used was:

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_pop;
```

This gives the specified role all privileges available at the database level.

### Important distinction

Database-level privileges and object-level privileges are related but not identical.

For example, permissions on tables can be managed separately:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON TABLE example_table
TO kodekloud_pop;
```

For this KodeKloud task, the required operation was the database-level grant.

---

## 7. Why Was a PostgreSQL Restart Not Required?

A common beginner misconception is that every database configuration change requires a service restart.

That is not true here.

Creating:

- users
- databases
- grants

is performed dynamically through PostgreSQL SQL commands.

Therefore, the following was **not required**:

```bash
sudo systemctl restart postgresql
```

In fact, the task explicitly prohibited restarting PostgreSQL.

### DevOps lesson

Avoid unnecessary service restarts, especially on production systems, because they can cause:

- temporary service interruption
- unnecessary downtime
- disruption to applications
- avoidable operational risk

---

## 8. Verification

After creating the role:

```sql
\du
```

This displayed:

```text
kodekloud_pop
```

After creating the database and granting permissions:

```sql
\l
```

The output showed:

```text
kodekloud_db7
```

and included access privileges for:

```text
kodekloud_pop
```

---

## 9. Testing Authentication

The most useful final verification was to test an actual connection using the newly created application user:

```bash
psql -U kodekloud_pop -d kodekloud_db7 -h localhost -W
```

A successful connection resulted in:

```text
kodekloud_db7=>
```

This is stronger verification than simply checking that the user and database exist.

It confirms that:

1. The PostgreSQL server is reachable.
2. The user exists.
3. The password is correct.
4. Authentication works.
5. The user can connect to the required database.

---

## 10. Important PostgreSQL Commands Learned

### Open PostgreSQL as administrator

```bash
sudo -u postgres psql
```

### List roles

```sql
\du
```

### List databases

```sql
\l
```

### Create a user

```sql
CREATE USER username WITH PASSWORD 'password';
```

### Create a database

```sql
CREATE DATABASE database_name;
```

### Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE database_name TO username;
```

### Exit PostgreSQL

```sql
\q
```

### Connect using a specific user and database

```bash
psql -U username -d database_name -h localhost -W
```

---

## 11. Authentication vs Authorization

This task is a good example of the difference between authentication and authorization.

### Authentication

Authentication answers:

> Who are you?

In this task:

```text
kodekloud_pop
```

authenticated using its PostgreSQL password.

### Authorization

Authorization answers:

> What are you allowed to access?

The command:

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_pop;
```

provided the required authorization.

So the workflow is:

```text
User
  │
  │ Authentication
  ▼
PostgreSQL
  │
  │ Authorization / Privileges
  ▼
kodekloud_db7
```

---

## 12. Final Result

The PostgreSQL environment was configured successfully.

```text
Server              : stdb01
PostgreSQL User     : kodekloud_pop
Database            : kodekloud_db7
Database Privileges : Granted
Authentication      : Verified
PostgreSQL Restart  : Not performed
Task                : Completed
```

---

## 13. DevOps Takeaway

This task demonstrates an important real-world DevOps pattern:

**Prepare infrastructure according to application requirements while making the minimum necessary changes.**

The PostgreSQL service was already available, so the work focused on:

1. Accessing the correct server.
2. Creating the required identity.
3. Creating the required database.
4. Applying least-scoped required permissions.
5. Testing connectivity.
6. Avoiding unnecessary service disruption.

**Day 17 completed successfully. ✅**
