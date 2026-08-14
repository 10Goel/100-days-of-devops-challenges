# Day 17 — PostgreSQL Commands

This file contains the commands used during the Day 17 KodeKloud DevOps task.

---

## 1. Connect to the PostgreSQL Server

From the jump host:

```bash
ssh peter@stdb01
```

> Correct hostname: `stdb01`

---

## 2. Check PostgreSQL Version

```bash
psql --version
```

---

## 3. Check PostgreSQL Service

```bash
sudo systemctl status postgresql
```

> Do not restart the PostgreSQL service for this task.

---

## 4. Open PostgreSQL as the Administrator

```bash
sudo -u postgres psql
```

---

## 5. Create the PostgreSQL User

```sql
CREATE USER kodekloud_pop WITH PASSWORD '<configured-password>';
```

Expected result:

```text
CREATE ROLE
```

---

## 6. Create the Database

```sql
CREATE DATABASE kodekloud_db7;
```

Expected result:

```text
CREATE DATABASE
```

---

## 7. Grant Full Database Privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_pop;
```

Expected result:

```text
GRANT
```

---

## 8. Verify PostgreSQL Roles

```sql
\du
```

Expected role:

```text
kodekloud_pop
```

---

## 9. Verify Databases and Privileges

```sql
\l
```

Expected database:

```text
kodekloud_db7
```

The access privileges should include `kodekloud_pop`.

---

## 10. Exit PostgreSQL

```sql
\q
```

---

## 11. Test Login Using the Application User

```bash
psql -U kodekloud_pop -d kodekloud_db7 -h localhost -W
```

Enter the configured password when prompted.

A successful connection should produce a PostgreSQL prompt similar to:

```text
kodekloud_db7=>
```

---

## 12. Exit the Test Connection

```sql
\q
```

---

## 🔎 Quick Command Sequence

```bash
ssh peter@stdb01
sudo -u postgres psql
```

Then:

```sql
CREATE USER kodekloud_pop WITH PASSWORD '<configured-password>';
CREATE DATABASE kodekloud_db7;
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_pop;
\du
\l
\q
```

Then test:

```bash
psql -U kodekloud_pop -d kodekloud_db7 -h localhost -W
```

Finally:

```sql
\q
```

---

## ⚠️ Commands NOT to Run

The task explicitly says not to restart PostgreSQL.

Do **not** run:

```bash
sudo systemctl restart postgresql
```

or:

```bash
sudo systemctl restart postgresql-*
```

---

## 📌 Useful PostgreSQL Reference

| Purpose | Command |
|---|---|
| Check version | `psql --version` |
| Check service | `sudo systemctl status postgresql` |
| Open PostgreSQL as admin | `sudo -u postgres psql` |
| Create user | `CREATE USER ...` |
| Create database | `CREATE DATABASE ...` |
| Grant privileges | `GRANT ALL PRIVILEGES ...` |
| List roles | `\du` |
| List databases | `\l` |
| Exit psql | `\q` |
| Test user connection | `psql -U ... -d ... -h localhost -W` |

---

## ✅ Day 17 Status

**PostgreSQL database setup completed successfully.**
