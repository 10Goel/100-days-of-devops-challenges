# Day 18 Commands --- MariaDB Database Server

## Server

``` text
DB Server: stdb01
```

------------------------------------------------------------------------

## 1. Connect to the DB Server

``` bash
ssh peter@stdb01
```

Verify hostname:

``` bash
hostname
```

Expected:

``` text
stdb01
```

------------------------------------------------------------------------

## 2. Install MariaDB Server

``` bash
sudo yum install -y mariadb-server
```

------------------------------------------------------------------------

## 3. Enable and Start MariaDB

``` bash
sudo systemctl enable --now mariadb
```

Check status:

``` bash
sudo systemctl status mariadb
```

------------------------------------------------------------------------

## 4. Open MariaDB as Administrator

``` bash
sudo mysql
```

------------------------------------------------------------------------

## 5. Create the Database

``` sql
CREATE DATABASE kodekloud_db1;
```

Verify:

``` sql
SHOW DATABASES LIKE 'kodekloud_db1';
```

------------------------------------------------------------------------

## 6. Create the Database User

``` sql
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';
```

Verify:

``` sql
SELECT User, Host
FROM mysql.user
WHERE User='kodekloud_sam';
```

Expected:

``` text
kodekloud_sam    %
```

------------------------------------------------------------------------

## 7. Grant Full Permissions

``` sql
GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';
```

Apply/reload privileges:

``` sql
FLUSH PRIVILEGES;
```

------------------------------------------------------------------------

## 8. Verify Permissions

``` sql
SHOW GRANTS FOR 'kodekloud_sam'@'%';
```

Expected grant:

``` text
GRANT ALL PRIVILEGES ON `kodekloud_db1`.* TO `kodekloud_sam`@`%`
```

------------------------------------------------------------------------

## 9. Exit MariaDB

``` sql
EXIT;
```

------------------------------------------------------------------------

## 10. Test Login as the New User

``` bash
mysql -u kodekloud_sam -p
```

Password:

``` text
ksH85UJjhb
```

------------------------------------------------------------------------

## 11. Verify Database Access

``` sql
SHOW DATABASES;
```

Select the required database:

``` sql
USE kodekloud_db1;
```

Expected:

``` text
Database changed
```

------------------------------------------------------------------------

# Complete Command Sequence

For reference, the essential command sequence is:

``` bash
ssh peter@stdb01
sudo yum install -y mariadb-server
sudo systemctl enable --now mariadb
sudo mysql
```

Then inside MariaDB:

``` sql
CREATE DATABASE kodekloud_db1;

CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';

GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';

FLUSH PRIVILEGES;

SHOW DATABASES LIKE 'kodekloud_db1';

SELECT User, Host
FROM mysql.user
WHERE User='kodekloud_sam';

SHOW GRANTS FOR 'kodekloud_sam'@'%';

EXIT;
```

Then test:

``` bash
mysql -u kodekloud_sam -p
```

Inside MariaDB:

``` sql
SHOW DATABASES;
USE kodekloud_db1;
```

------------------------------------------------------------------------

# Troubleshooting Commands

## Check MariaDB service

``` bash
sudo systemctl status mariadb
```

## Check whether MariaDB is listening

``` bash
sudo ss -lntp | grep 3306
```

## Check installed MariaDB packages

``` bash
rpm -qa | grep -i mariadb
```

## Check database

``` sql
SHOW DATABASES LIKE 'kodekloud_db1';
```

## Check user and host

``` sql
SELECT User, Host
FROM mysql.user
WHERE User='kodekloud_sam';
```

## Check user privileges

``` sql
SHOW GRANTS FOR 'kodekloud_sam'@'%';
```

## Check current database

``` sql
SELECT DATABASE();
```

------------------------------------------------------------------------

# Quick Reference

  -------------------------------------------------------------------------------------------------------
  Purpose                             Command
  ----------------------------------- -------------------------------------------------------------------
  Connect to server                   `ssh peter@stdb01`

  Install MariaDB                     `sudo yum install -y mariadb-server`

  Start + enable                      `sudo systemctl enable --now mariadb`

  Check service                       `sudo systemctl status mariadb`

  Open MariaDB                        `sudo mysql`

  Create DB                           `CREATE DATABASE kodekloud_db1;`

  Create user                         `CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';`

  Grant access                        `GRANT ALL PRIVILEGES ON kodekloud_db1.* TO 'kodekloud_sam'@'%';`

  Reload privileges                   `FLUSH PRIVILEGES;`

  Check user                          `SELECT User, Host FROM mysql.user WHERE User='kodekloud_sam';`

  Check grants                        `SHOW GRANTS FOR 'kodekloud_sam'@'%';`

  Exit MariaDB                        `EXIT;`

  Login as user                       `mysql -u kodekloud_sam -p`

  Select database                     `USE kodekloud_db1;`
  -------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## Day 18 Final Status

``` text
MariaDB          : Configured
Database         : kodekloud_db1
User             : kodekloud_sam@%
Privileges        : ALL PRIVILEGES
Authentication   : Verified
Database Access  : Verified
KodeKloud Check  : PASSED ✅
```
