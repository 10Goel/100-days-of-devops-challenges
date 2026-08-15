# Day 18 Notes --- MariaDB Database Server

## 1. What Was the Day 18 Task?

The Day 18 challenge required setting up a MariaDB database server on
the Nautilus DB Server in the Stratos Datacenter.

The required configuration was:

``` text
Database : kodekloud_db1
User     : kodekloud_sam
Host     : %
Password : ksH85UJjhb
Access   : Full privileges
```

The task was successfully completed and passed by the KodeKloud checker.

------------------------------------------------------------------------

# 2. What is MariaDB?

MariaDB is an open-source relational database management system (RDBMS).

It is a popular MySQL-compatible database platform used by applications
to store structured data.

A simplified architecture looks like:

``` text
Application
     |
     | SQL queries
     v
MariaDB Server
     |
     v
Database
     |
     v
Tables / Records
```

In a DevOps environment, the database is often deployed as a separate
infrastructure component from the application server.

------------------------------------------------------------------------

# 3. MariaDB Server vs MariaDB Client

These are two different concepts.

## MariaDB Server

The server process manages:

-   Databases
-   Tables
-   Users
-   Permissions
-   Connections
-   Queries
-   Data storage

The service is managed using:

``` bash
systemctl
```

For example:

``` bash
sudo systemctl status mariadb
```

## MariaDB Client

The client is the command-line program used to connect to a MariaDB
server.

Example:

``` bash
mysql -u kodekloud_sam -p
```

The client sends SQL commands to the MariaDB server.

------------------------------------------------------------------------

# 4. Installing MariaDB

The MariaDB server package was installed using:

``` bash
sudo yum install -y mariadb-server
```

The `-y` option automatically answers yes to package installation
prompts.

------------------------------------------------------------------------

# 5. Starting and Enabling MariaDB

The service was started and enabled with:

``` bash
sudo systemctl enable --now mariadb
```

This performs two operations:

``` text
enable
   +
start
```

### `start`

Starts MariaDB immediately.

### `enable`

Configures MariaDB to start automatically during system boot.

The service can be checked with:

``` bash
sudo systemctl status mariadb
```

------------------------------------------------------------------------

# 6. Creating a Database

The database was created using:

``` sql
CREATE DATABASE kodekloud_db1;
```

A database provides a logical namespace/container for database objects
such as:

-   Tables
-   Views
-   Stored procedures
-   Functions
-   Triggers

The database created in this challenge was:

``` text
kodekloud_db1
```

------------------------------------------------------------------------

# 7. MariaDB Users

MariaDB uses accounts to authenticate clients.

A MariaDB account is represented as:

``` text
'username'@'host'
```

For example:

``` text
'kodekloud_sam'@'%'
```

This contains two components:

``` text
Username = kodekloud_sam
Host     = %
```

The username identifies the account, while the host determines where the
connection is allowed to originate.

------------------------------------------------------------------------

# 8. Why `%` Matters

The `%` character is a wildcard.

The account:

``` text
'kodekloud_sam'@'%'
```

means the account can authenticate from any host, subject to other
network, firewall, and MariaDB configuration restrictions.

This is especially relevant in a distributed application architecture.

For example:

``` text
Application Server
        |
        | Database connection
        v
   MariaDB Server
```

If the application runs on another server, a local-only account such as:

``` text
'kodekloud_sam'@'localhost'
```

would not match that remote connection.

For this KodeKloud task, the required account was:

``` text
'kodekloud_sam'@'%'
```

------------------------------------------------------------------------

# 9. Creating the User

The database user was created using:

``` sql
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';
```

The command does three important things:

1.  Creates the account.
2.  Defines the allowed host pattern.
3.  Sets the authentication password.

------------------------------------------------------------------------

# 10. Granting Permissions

The user was given full privileges on the required database:

``` sql
GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';
```

Understanding the syntax:

``` text
GRANT ALL PRIVILEGES
      |
      +---- Full database privileges

ON kodekloud_db1.*
      |
      +---- All objects inside kodekloud_db1

TO 'kodekloud_sam'@'%'
      |
      +---- Target database account
```

This grants the user permissions on all objects inside `kodekloud_db1`.

------------------------------------------------------------------------

# 11. What Does ALL PRIVILEGES Mean?

`ALL PRIVILEGES` grants the privileges available for the specified
scope.

Depending on the MariaDB version and context, these can include
operations such as:

-   SELECT
-   INSERT
-   UPDATE
-   DELETE
-   CREATE
-   DROP
-   ALTER
-   INDEX
-   REFERENCES
-   EXECUTE
-   And other applicable privileges

The scope is important.

This:

``` sql
ON kodekloud_db1.*
```

means the permissions apply to objects within `kodekloud_db1`.

It does not mean the user automatically receives unrestricted
administrative control over every database on the server.

------------------------------------------------------------------------

# 12. `FLUSH PRIVILEGES`

The command used was:

``` sql
FLUSH PRIVILEGES;
```

It reloads privilege information.

Modern MariaDB `CREATE USER` and `GRANT` statements manage the privilege
tables appropriately themselves, so `FLUSH PRIVILEGES` is generally not
required after every normal `CREATE USER` or `GRANT`.

It was nevertheless used in the challenge workflow to explicitly reload
privilege information.

------------------------------------------------------------------------

# 13. Verifying the Database

The database can be checked using:

``` sql
SHOW DATABASES LIKE 'kodekloud_db1';
```

Or:

``` sql
SHOW DATABASES;
```

------------------------------------------------------------------------

# 14. Verifying the User

The account can be checked using:

``` sql
SELECT User, Host
FROM mysql.user
WHERE User='kodekloud_sam';
```

Expected:

``` text
kodekloud_sam    %
```

This is important because MariaDB treats different host values as
different accounts.

For example:

``` text
'kodekloud_sam'@'localhost'
'kodekloud_sam'@'%'
```

are separate MariaDB accounts.

------------------------------------------------------------------------

# 15. Verifying Permissions

The user's privileges were checked using:

``` sql
SHOW GRANTS FOR 'kodekloud_sam'@'%';
```

The important result is equivalent to:

``` text
GRANT ALL PRIVILEGES ON `kodekloud_db1`.* TO `kodekloud_sam`@`%`
```

This confirms that the user has full privileges on the required
database.

------------------------------------------------------------------------

# 16. End-to-End Testing

Configuration should not only be written; it should be tested.

The user was tested with:

``` bash
mysql -u kodekloud_sam -p
```

After authentication, the database was selected:

``` sql
USE kodekloud_db1;
```

Successful output:

``` text
Database changed
```

This provided an end-to-end validation:

``` text
Credentials
    |
    v
Authentication
    |
    v
MariaDB
    |
    v
Authorization
    |
    v
kodekloud_db1
```

------------------------------------------------------------------------

# 17. Authentication vs Authorization

This challenge demonstrates an important security distinction.

## Authentication

Authentication answers:

> Who are you?

For example:

``` text
User: kodekloud_sam
Password: ksH85UJjhb
```

## Authorization

Authorization answers:

> What are you allowed to do?

That was configured using:

``` sql
GRANT ALL PRIVILEGES
```

Therefore:

``` text
Authentication = identity verification
Authorization  = permission control
```

------------------------------------------------------------------------

# 18. Why Applications Should Use Dedicated Users

Using the MariaDB root account for an application is poor practice.

A better architecture is:

``` text
Application
     |
     | application-specific credentials
     v
kodekloud_sam
     |
     | authorized access
     v
kodekloud_db1
```

Benefits include:

-   Reduced blast radius
-   Better access control
-   Easier auditing
-   Easier credential rotation
-   Separation of responsibilities
-   Better security practices

------------------------------------------------------------------------

# 19. Important DevOps Lesson

A configuration can look correct but still fail validation.

During this Day 18 task, the MariaDB account's host component was
important.

The correct final account was:

``` text
'kodekloud_sam'@'%'
```

rather than:

``` text
'kodekloud_sam'@'localhost'
```

This demonstrates why DevOps engineers should verify the exact state of
infrastructure instead of assuming that a successful local test means
the configuration completely satisfies the infrastructure requirement.

------------------------------------------------------------------------

# 20. Useful Verification Workflow

A reliable database provisioning workflow is:

``` text
Install
   ↓
Start
   ↓
Enable
   ↓
Create database
   ↓
Create user
   ↓
Grant privileges
   ↓
Verify database
   ↓
Verify user + host
   ↓
Verify grants
   ↓
Test login
   ↓
Test database access
```

This workflow can be reused in many infrastructure and application
deployment scenarios.

------------------------------------------------------------------------

# 21. Final Day 18 State

``` text
Server      : stdb01
DB Engine   : MariaDB
Database    : kodekloud_db1
User        : kodekloud_sam
Host        : %
Password    : ksH85UJjhb
Privileges  : ALL PRIVILEGES
Status      : PASSED
```

## Key Takeaways

-   MariaDB is a relational database server.
-   `systemctl` manages the MariaDB service.
-   Database users are represented as `user@host`.
-   `%` is a wildcard host pattern.
-   `CREATE USER` creates an account.
-   `GRANT` controls authorization.
-   `database.*` means all objects within that database.
-   Authentication and authorization are different concepts.
-   Dedicated application database users are preferable to root access.
-   Always verify infrastructure configuration end-to-end.

------------------------------------------------------------------------

**Day 18 --- MariaDB Database Server Setup: Successfully Completed ✅**
