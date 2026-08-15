# Day 18 --- MariaDB Database Server Setup

![DevOps](https://img.shields.io/badge/100%20Days%20of%20DevOps-Day%2018-blue)
![MariaDB](https://img.shields.io/badge/Database-MariaDB-orange)
![Status](https://img.shields.io/badge/Status-Completed-success)

## 📌 Challenge Overview

**Day 18** of the KodeKloud 100 Days of DevOps challenge focused on
provisioning and configuring a MariaDB database server in the **Stratos
Datacenter**.

The objective was to prepare the Nautilus DB Server with MariaDB, create
the required application database and database user, and grant the user
full permissions on that database.

## 🎯 Task Requirements

The following configuration was required:

  Requirement       Configuration
  ----------------- ------------------------------------
  DB Server         `stdb01`
  Database Server   MariaDB
  Database          `kodekloud_db1`
  Database User     `kodekloud_sam`
  Password          `ksH85UJjhb`
  Host              `%`
  Privileges        Full privileges on `kodekloud_db1`

> **Note:** The database user was created as `kodekloud_sam`@`%`. The
> `%` host allows the MariaDB account to authenticate from remote hosts,
> which is important for an application/database-server architecture.

## 🛠️ Implementation

### 1. Connect to the DB Server

``` bash
ssh peter@stdb01
```

Verify the server:

``` bash
hostname
```

Expected:

``` text
stdb01
```

### 2. Install MariaDB

``` bash
sudo yum install -y mariadb-server
```

### 3. Enable and Start MariaDB

``` bash
sudo systemctl enable --now mariadb
```

Verify:

``` bash
sudo systemctl status mariadb
```

The service should be:

``` text
active (running)
```

### 4. Create the Database

Enter MariaDB:

``` bash
sudo mysql
```

Create the required database:

``` sql
CREATE DATABASE kodekloud_db1;
```

### 5. Create the Database User

``` sql
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';
```

### 6. Grant Full Permissions

``` sql
GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';
```

Apply the privilege changes:

``` sql
FLUSH PRIVILEGES;
```

## 🔍 Verification

Verify that the database exists:

``` sql
SHOW DATABASES LIKE 'kodekloud_db1';
```

Verify the user and host:

``` sql
SELECT User, Host FROM mysql.user WHERE User='kodekloud_sam';
```

Expected account:

``` text
kodekloud_sam    %
```

Verify the granted permissions:

``` sql
SHOW GRANTS FOR 'kodekloud_sam'@'%';
```

Expected privilege:

``` text
GRANT ALL PRIVILEGES ON `kodekloud_db1`.* TO `kodekloud_sam`@`%`
```

## 🧪 End-to-End Authentication Test

Exit the MariaDB root session:

``` sql
EXIT;
```

Log in using the newly created database user:

``` bash
mysql -u kodekloud_sam -p
```

Enter the configured password:

``` text
ksH85UJjhb
```

Then verify database access:

``` sql
SHOW DATABASES;
```

Select the application database:

``` sql
USE kodekloud_db1;
```

Successful output:

``` text
Database changed
```

This confirmed that the newly created user could authenticate and access
the required database.

## 🧠 Key DevOps Concepts Learned

### MariaDB Server

MariaDB is a relational database management system used to store and
manage structured application data. In a typical infrastructure setup,
the database server is separated from application servers.

### Database

A database is a logical container for tables, views, procedures, and
other database objects.

In this challenge:

``` text
kodekloud_db1
```

was created for the application.

### Database User

Applications should generally use a dedicated database account instead
of connecting as the database administrator.

The account created was:

``` text
kodekloud_sam
```

### Host Component

A MariaDB account consists of both a username and a host:

``` text
'username'@'host'
```

For this challenge:

``` text
'kodekloud_sam'@'%'
```

The `%` wildcard represents connections from any host.

### Privileges

The following command grants full permissions on every object within the
specified database:

``` sql
GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';
```

The structure is:

``` text
database.*
```

where `*` means all objects inside that database.

## 🏁 Result

**Day 18 was successfully completed and passed by the KodeKloud task
checker.**

### Final Configuration

``` text
Server       : stdb01
DB Engine    : MariaDB
Database     : kodekloud_db1
DB User      : kodekloud_sam
Host         : %
Privileges   : ALL PRIVILEGES
Status       : PASSED ✅
```
------------------------------------------------------------------------

**100 Days of DevOps --- Day 18 Complete 🚀**
