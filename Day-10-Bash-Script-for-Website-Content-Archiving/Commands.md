# Day 10 — Commands Reference

This file documents every important command used during the Day 10 challenge and explains **what the command does, why it is used, and where it fits into the task**.

---

# 1. SSH Into App Server 2

```bash
ssh <app-server-user>@stapp02
```

## Purpose

Connects to App Server 2.

The script must be created and executed on App Server 2.

## General Syntax

```bash
ssh USER@HOST
```

---

# 2. Install ZIP

```bash
sudo yum install -y zip
```

## Purpose

Installs the `zip` utility required to create the website archive.

## Why is this command outside the script?

The challenge specifically states that the ZIP package must be installed manually before executing the script.

Therefore, this command should **not** appear inside:

```text
/scripts/news_archive.sh
```

---

# 3. Verify ZIP

```bash
zip -v
```

## Purpose

Verifies that ZIP is installed and available.

If version information is displayed, the ZIP utility is available.

---

# 4. Create `/scripts`

```bash
sudo mkdir -p /scripts
```

## Purpose

Creates the directory where the Bash script must be stored.

Required location:

```text
/scripts/news_archive.sh
```

## Why `-p`?

`-p` creates the directory if it does not exist.

If the directory already exists, the command does not produce an error.

---

# 5. Create `/archives`

```bash
sudo mkdir -p /archives
```

## Purpose

Creates the local temporary storage directory.

The archive will be created here:

```text
/archives/xfusioncorp_news.zip
```

---

# 6. Generate SSH Keys

```bash
ssh-keygen -t rsa -b 4096
```

## Purpose

Generates an SSH public/private key pair.

Typical files:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

### Private key

```text
id_rsa
```

Must remain protected.

### Public key

```text
id_rsa.pub
```

Can be installed on the remote server.

---

# 7. Copy Public Key to Storage Server

```bash
ssh-copy-id natasha@ststor01
```

## Purpose

Installs the public SSH key for the `natasha` account on the Storage Server.

The public key is normally added to:

```text
~/.ssh/authorized_keys
```

on the remote server.

## Important

This command may ask for the `natasha` password during the initial setup.

That is normal.

The password is used to install the public key.

After that, SSH/SCP can authenticate using the SSH key.

---

# 8. Test Passwordless SSH

```bash
ssh natasha@ststor01
```

## Purpose

Tests whether SSH key-based authentication is working.

Expected:

```text
Connection succeeds without asking for natasha's password.
```

If successful, exit:

```bash
exit
```

---

# 9. Create the Bash Script

```bash
vi /scripts/news_archive.sh
```

## Purpose

Creates or opens the required Bash script.

The challenge specifically requires:

```text
/scripts/news_archive.sh
```

---

# 10. Complete Script

```bash
#!/bin/bash

SOURCE_DIR="/var/www/html/news"
ARCHIVE_DIR="/archives"
ARCHIVE_NAME="xfusioncorp_news.zip"

REMOTE_USER="natasha"
REMOTE_HOST="ststor01"
REMOTE_DIR="/archives"

mkdir -p "$ARCHIVE_DIR"

zip -r "$ARCHIVE_DIR/$ARCHIVE_NAME" "$SOURCE_DIR"

scp "$ARCHIVE_DIR/$ARCHIVE_NAME" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
```

---

# 11. `#!/bin/bash`

```bash
#!/bin/bash
```

## Purpose

The shebang tells Linux that Bash should be used to interpret the script.

---

# 12. Define Source Directory

```bash
SOURCE_DIR="/var/www/html/news"
```

## Purpose

Stores the path of the website content that needs to be archived.

---

# 13. Define Archive Directory

```bash
ARCHIVE_DIR="/archives"
```

## Purpose

Stores the location where the local ZIP archive will be created.

---

# 14. Define Archive Name

```bash
ARCHIVE_NAME="xfusioncorp_news.zip"
```

## Purpose

Stores the required archive filename.

---

# 15. Define Remote User

```bash
REMOTE_USER="natasha"
```

## Purpose

Defines the account that will be used on the Storage Server.

---

# 16. Define Remote Host

```bash
REMOTE_HOST="ststor01"
```

## Purpose

Defines the hostname of the Nautilus Storage Server.

---

# 17. Define Remote Directory

```bash
REMOTE_DIR="/archives"
```

## Purpose

Defines the destination directory on the Storage Server.

---

# 18. Create Archive Directory

```bash
mkdir -p "$ARCHIVE_DIR"
```

## Purpose

Creates `/archives` if it does not already exist.

Because `-p` is used, the command does not fail simply because the directory already exists.

---

# 19. Create ZIP Archive

```bash
zip -r "$ARCHIVE_DIR/$ARCHIVE_NAME" "$SOURCE_DIR"
```

## Expanded Command

```bash
zip -r /archives/xfusioncorp_news.zip /var/www/html/news
```

## Purpose

Creates a ZIP archive containing the website content.

## Why `-r`?

`-r` means **recursive**.

It ensures that files and subdirectories inside `/var/www/html/news` are included.

---

# 20. Copy Archive Using SCP

```bash
scp "$ARCHIVE_DIR/$ARCHIVE_NAME" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
```

## Expanded Command

```bash
scp /archives/xfusioncorp_news.zip natasha@ststor01:/archives/
```

## Purpose

Transfers the archive from App Server 2 to the Nautilus Storage Server.

---

# 21. SCP Syntax

General syntax:

```bash
scp SOURCE USER@HOST:DESTINATION
```

Example:

```bash
scp /archives/xfusioncorp_news.zip natasha@ststor01:/archives/
```

Meaning:

```text
SOURCE:
    /archives/xfusioncorp_news.zip

USER:
    natasha

HOST:
    ststor01

DESTINATION:
    /archives/
```

---

# 22. Why SCP Does Not Ask for a Password

SCP uses SSH for authentication.

Because SSH key-based authentication was configured using:

```bash
ssh-copy-id natasha@ststor01
```

the SCP command can authenticate using the SSH key.

Therefore:

```bash
scp ...
```

can run without an interactive password prompt.

This is essential for automation.

---

# 23. Make Script Executable

```bash
chmod +x /scripts/news_archive.sh
```

## Purpose

Adds execute permission to the Bash script.

Without execute permission, directly running:

```bash
/scripts/news_archive.sh
```

may result in:

```text
Permission denied
```

---

# 24. Verify Script Permissions

```bash
ls -l /scripts/news_archive.sh
```

Example:

```text
-rwxr-xr-x
```

The important part is the presence of:

```text
x
```

which represents execute permission.

---

# 25. Execute the Script

```bash
/scripts/news_archive.sh
```

## Purpose

Runs the complete archive and transfer process.

The script:

```text
1. Creates /archives if required
2. Reads /var/www/html/news
3. Creates xfusioncorp_news.zip
4. Stores it in /archives/
5. Uses SCP
6. Transfers the archive to ststor01
```

---

# 26. Verify Local Archive

```bash
ls -lh /archives/
```

## Purpose

Checks whether the local archive was successfully created.

Expected:

```text
xfusioncorp_news.zip
```

---

# 27. Verify Archive Contents

```bash
unzip -l /archives/xfusioncorp_news.zip
```

## Purpose

Lists the files inside the ZIP archive without extracting them.

This verifies that the website content was included.

---

# 28. Connect to Storage Server

```bash
ssh natasha@ststor01
```

## Purpose

Connects to the Storage Server so the remote archive can be verified.

---

# 29. Verify Remote Archive

```bash
ls -lh /archives/
```

## Purpose

Checks whether the archive was successfully copied to the Storage Server.

Expected:

```text
xfusioncorp_news.zip
```

---

# 30. Exit Storage Server

```bash
exit
```

## Purpose

Closes the SSH session and returns to App Server 2.

---

# 31. Check SSH Configuration

```bash
ls -la ~/.ssh/
```

## Purpose

Displays SSH keys and configuration files.

Common files include:

```text
id_rsa
id_rsa.pub
authorized_keys
known_hosts
```

---

# 32. Display Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

## Purpose

Displays the public SSH key.

The public key can be installed on the remote server.

## Security Note

Never expose or share the private key:

```text
~/.ssh/id_rsa
```

---

# 33. SSH Debugging

```bash
ssh -v natasha@ststor01
```

## Purpose

Displays detailed information about the SSH connection.

For more detailed debugging:

```bash
ssh -vvv natasha@ststor01
```

This is useful when passwordless authentication is not working.

---

# 34. Test SCP Manually

```bash
scp /archives/xfusioncorp_news.zip natasha@ststor01:/archives/
```

## Purpose

Tests whether the same SCP transfer used by the script works manually.

If the command transfers the file without asking for a password, SSH key authentication is functioning correctly.

---

# 35. Check `/archives` Permissions

```bash
ls -ld /archives
```

## Purpose

Displays the ownership and permissions of the local archive directory.

Useful when troubleshooting:

```text
Permission denied
```

---

# 36. Check Website Directory Permissions

```bash
ls -ld /var/www/html/news
```

## Purpose

Checks the ownership and permissions of the website directory.

The user running the script needs appropriate read access to the website content.

---

# 37. Check Remote Archive Directory

On the Storage Server:

```bash
ls -ld /archives
```

## Purpose

Verifies that the destination directory exists and is accessible to the remote user.

---

# 38. Complete Command Sequence

```bash
ssh <app-server-user>@stapp02

sudo yum install -y zip

zip -v

sudo mkdir -p /scripts
sudo mkdir -p /archives

ssh-keygen -t rsa -b 4096

ssh-copy-id natasha@ststor01

ssh natasha@ststor01
exit

vi /scripts/news_archive.sh

chmod +x /scripts/news_archive.sh

/scripts/news_archive.sh

ls -lh /archives/

unzip -l /archives/xfusioncorp_news.zip

ssh natasha@ststor01

ls -lh /archives/

exit
```

---

# 39. Commands That Must NOT Be Inside the Script

Do not put:

```bash
sudo yum install -y zip
```

inside the script.

The ZIP package must be installed manually.

Also do not use:

```bash
sudo zip ...
```

or:

```bash
sudo scp ...
```

inside the script.

The challenge explicitly states:

```text
Do not use sudo inside the script.
```

---

# 40. Quick Command Cheat Sheet

| Command | Purpose |
|---|---|
| `ssh user@server` | Connect to remote server |
| `sudo yum install -y zip` | Install ZIP manually |
| `zip -v` | Verify ZIP installation |
| `mkdir -p` | Create directory |
| `ssh-keygen` | Generate SSH key pair |
| `ssh-copy-id` | Install public key remotely |
| `zip -r` | Create recursive ZIP archive |
| `scp` | Securely copy files between servers |
| `chmod +x` | Add execute permission |
| `ls -lh` | List files with readable sizes |
| `ls -ld` | Check directory permissions |
| `unzip -l` | List ZIP contents |
| `exit` | Close SSH session |

---
---
