# Day 10 — Detailed Notes

## 1. Core Concept

Day 10 combines multiple Linux and DevOps concepts into one practical automation task:

```text
Bash Scripting
      +
ZIP Archiving
      +
SSH
      +
SSH Key Authentication
      +
SCP
      +
Linux Permissions
      +
Automation
```

The overall workflow is:

```text
Website Content
      ↓
Create ZIP Archive
      ↓
Store Locally
      ↓
Transfer to Storage Server
      ↓
Verify Backup
```

---

# 2. Why This Task Matters

In a production environment, website content may need to be backed up regularly.

Doing this manually would require:

```text
1. Log into the server
2. Find the website files
3. Create an archive
4. Copy the archive
5. Enter a password
6. Verify the backup
```

This is repetitive and error-prone.

A Bash script converts this process into an automated operation:

```bash
/scripts/news_archive.sh
```

This is the basic idea behind infrastructure automation.

---

# 3. Bash Scripting

A Bash script is a file containing Linux commands that can be executed as a single program.

Example:

```bash
#!/bin/bash

echo "Starting backup"

zip -r backup.zip /var/www/html/news

echo "Backup completed"
```

Instead of manually executing every command, we execute the script.

---

# 4. Shebang

The first line is:

```bash
#!/bin/bash
```

This is called the **shebang**.

It tells Linux which interpreter should execute the script.

Here:

```text
Interpreter = Bash
```

---

# 5. Bash Variables

The script uses variables:

```bash
SOURCE_DIR="/var/www/html/news"
ARCHIVE_DIR="/archives"
ARCHIVE_NAME="xfusioncorp_news.zip"
```

and:

```bash
REMOTE_USER="natasha"
REMOTE_HOST="ststor01"
REMOTE_DIR="/archives"
```

Variables make scripts easier to read, maintain, and modify.

For example:

```bash
zip -r "$ARCHIVE_DIR/$ARCHIVE_NAME" "$SOURCE_DIR"
```

is much easier to maintain than repeatedly writing long paths.

---

# 6. Why Double Quotes Are Used Around Variables

The script uses:

```bash
"$SOURCE_DIR"
```

instead of:

```bash
$SOURCE_DIR
```

Double quotes protect values containing spaces or special characters.

For example:

```text
/var/www/html/company news
```

Without quotes, Bash could interpret the path as multiple arguments.

Therefore, quoting variables is a good Bash scripting practice.

---

# 7. ZIP Archiving

ZIP is an archive/compression format.

The basic command is:

```bash
zip -r archive.zip directory
```

For this challenge:

```bash
zip -r /archives/xfusioncorp_news.zip /var/www/html/news
```

This creates:

```text
/archives/xfusioncorp_news.zip
```

containing the website content.

---

# 8. Meaning of `-r`

The `-r` option means:

```text
recursive
```

Suppose the website contains:

```text
news/
├── index.html
├── article.html
├── css/
│   └── style.css
└── images/
    ├── logo.png
    └── banner.png
```

Using:

```bash
zip -r
```

includes the complete directory structure and its contents.

---

# 9. Why ZIP Is Installed Outside the Script

The challenge specifically states that the `zip` package must be installed manually.

Therefore:

```bash
sudo yum install -y zip
```

is executed before running the script.

It is deliberately **not** included inside `news_archive.sh`.

This separates:

```text
Environment Preparation
        ↓
Install required software

from

Task Automation
        ↓
Run the actual backup
```

---

# 10. SSH

SSH stands for:

```text
Secure Shell
```

SSH provides secure remote access to Linux systems.

Example:

```bash
ssh natasha@ststor01
```

means:

```text
Username = natasha
Server   = ststor01
```

---

# 11. Why SSH Is Needed

The archive is created on:

```text
App Server 2
```

but must also be stored on:

```text
Nautilus Storage Server
```

SSH provides secure communication between the servers.

---

# 12. Password Authentication

Normally, SSH can authenticate using a password:

```text
ssh natasha@ststor01
          ↓
Password prompt
          ↓
Authentication
```

This is acceptable for an interactive human session.

However, it is unsuitable for automated scripts.

Imagine the script being executed by cron at 2:00 AM.

Nobody may be available to enter the password.

Therefore, we need non-interactive authentication.

---

# 13. SSH Key-Based Authentication

SSH supports authentication using a cryptographic key pair.

There are two keys:

```text
Private Key
Public Key
```

The private key remains on the source machine.

The public key is installed on the destination account.

Conceptually:

```text
App Server 2
     |
     | Private Key
     |
     ↓
SSH Authentication
     |
     ↓
Storage Server
     |
     | Public Key
     ↓
Authorized
```

---

# 14. Private Key

A typical private key is:

```text
~/.ssh/id_rsa
```

The private key must be protected.

It should never be unnecessarily shared with other users.

---

# 15. Public Key

A typical public key is:

```text
~/.ssh/id_rsa.pub
```

The public key can safely be installed on the remote server.

---

# 16. `ssh-keygen`

The command:

```bash
ssh-keygen -t rsa -b 4096
```

generates an SSH key pair.

Typical files:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

The first is the private key.

The second is the public key.

---

# 17. `ssh-copy-id`

The command:

```bash
ssh-copy-id natasha@ststor01
```

installs the public key for the remote account.

It normally adds the key to:

```text
~/.ssh/authorized_keys
```

on the Storage Server.

---

# 18. Why `ssh-copy-id` Can Ask for a Password

During initial setup, password authentication may be required.

The process is:

```text
No public key installed
        ↓
Authenticate using password
        ↓
Install public key
        ↓
Future authentication uses SSH key
```

Therefore, entering the password during `ssh-copy-id` is normal.

The requirement is that the **script itself** must not ask for a password.

---

# 19. Passwordless Authentication

Passwordless authentication does not mean:

```text
No authentication
```

It means:

```text
No interactive password authentication
```

Authentication still occurs using the SSH key.

Therefore:

```text
Passwordless SSH
=
SSH key-based authentication
```

---

# 20. Why Passwordless Authentication Is Required

The script contains:

```bash
scp ...
```

If SCP asks:

```text
Password:
```

the script becomes interactive.

That defeats the purpose of automation.

The desired workflow is:

```text
Script
  ↓
SCP
  ↓
SSH Key
  ↓
Authentication
  ↓
File Transfer
```

No human interaction is required.

---

# 21. SCP

SCP means:

```text
Secure Copy Protocol
```

It is used to securely transfer files between systems.

Example:

```bash
scp file.txt natasha@ststor01:/archives/
```

This means:

```text
Source:
file.txt

Remote User:
natasha

Remote Host:
ststor01

Destination:
 /archives/
```

---

# 22. `cp` vs `scp`

### `cp`

Used for copying files on the same machine:

```bash
cp file.txt /backup/
```

### `scp`

Used for copying files between machines:

```bash
scp file.txt natasha@ststor01:/archives/
```

Therefore:

```text
cp
↓
Local copy

scp
↓
Remote copy
```

---

# 23. Why SCP Uses SSH

SCP uses SSH-based communication.

Therefore, when we configure SSH key-based authentication:

```bash
ssh-copy-id natasha@ststor01
```

the same authentication mechanism can be used by:

```bash
scp
```

This is why the archive can be copied without a password prompt.

---

# 24. Why We Don't Store Passwords in the Script

A bad approach would be to put a password directly into a script.

This creates security risks because anyone who can read the script may obtain the credentials.

Instead:

```text
SSH Key
   ↓
Authentication
   ↓
SCP
   ↓
Remote Transfer
```

is used.

This is much more appropriate for automation.

---

# 25. Linux Execute Permission

The script needs execute permission.

We use:

```bash
chmod +x /scripts/news_archive.sh
```

Here:

```text
chmod
=
change file permissions

+x
=
add execute permission
```

---

# 26. Linux Permissions

Linux has three basic permissions:

```text
r = read
w = write
x = execute
```

Example:

```text
-rwxr-xr-x
```

contains execute permissions.

---

# 27. Why the Script Needs Execute Permission

Without execute permission:

```bash
/scripts/news_archive.sh
```

may result in:

```text
Permission denied
```

After:

```bash
chmod +x /scripts/news_archive.sh
```

the script can be executed directly.

---

# 28. Why `sudo` Is Not Used Inside the Script

The challenge explicitly says:

```text
Do not use sudo inside the script.
```

Therefore, the script contains no `sudo`.

Required administrative setup is performed manually beforehand.

The automation itself runs under the appropriate server user's permissions.

---

# 29. Why `sudo` Can Be a Problem in Automation

Consider:

```bash
sudo some-command
```

If sudo requires a password, the script can stop and ask:

```text
[sudo] password for user:
```

This makes the automation interactive.

Good automation should ideally be:

```text
Repeatable
+
Predictable
+
Non-interactive
```

---

# 30. Local Backup

The local archive is:

```text
/archives/xfusioncorp_news.zip
```

It is created from:

```text
/var/www/html/news
```

The process is:

```text
/var/www/html/news
        ↓
      zip -r
        ↓
/archives/xfusioncorp_news.zip
```

---

# 31. Remote Backup

The same archive is transferred to:

```text
ststor01:/archives/
```

Final remote path:

```text
/archives/xfusioncorp_news.zip
```

---

# 32. Why Store a Second Copy?

The challenge states that `/archives/` on App Server 2 is temporary storage and may be cleaned periodically.

Therefore, a second copy is maintained on the Storage Server.

Conceptually:

```text
App Server 2
Temporary Archive
        +
Storage Server
Persistent/Validation Copy
```

This is a basic backup strategy.

---

# 33. Verification

A good DevOps practice is:

```text
Execute
   ↓
Verify
```

Do not simply assume that the operation succeeded.

### Local verification

```bash
ls -lh /archives/
```

### Check archive contents

```bash
unzip -l /archives/xfusioncorp_news.zip
```

### Remote verification

```bash
ssh natasha@ststor01
ls -lh /archives/
```

---

# 34. Common Error — `zip: command not found`

If the script shows:

```text
zip: command not found
```

install ZIP:

```bash
sudo yum install -y zip
```

Then verify:

```bash
zip -v
```

---

# 35. Common Error — SCP Asks for Password

If SCP asks for a password, first test:

```bash
ssh natasha@ststor01
```

If SSH also asks for a password, inspect the SSH key configuration.

Check:

```bash
ls -la ~/.ssh/
```

Then ensure the public key is installed for the remote user.

---

# 36. Common Error — Permission Denied

If the script cannot access `/var/www/html/news`:

```bash
ls -ld /var/www/html/news
```

If the script cannot write to `/archives`:

```bash
ls -ld /archives
```

The executing user must have the appropriate permissions.

---

# 37. Common Error — Script Cannot Execute

Check:

```bash
ls -l /scripts/news_archive.sh
```

If execute permission is missing:

```bash
chmod +x /scripts/news_archive.sh
```

---

# 38. Common Error — Remote Directory Missing

Connect:

```bash
ssh natasha@ststor01
```

Check:

```bash
ls -ld /archives
```

The destination directory must exist and be accessible to the remote user.

---

# 39. Why This Is a DevOps Task

This challenge combines:

```text
Linux
+
Bash
+
File Permissions
+
SSH
+
SSH Keys
+
SCP
+
Remote Servers
+
Backup
+
Automation
```

These are foundational DevOps skills.

---

# 40. Real-World Applications

The same concepts can be used for:

### Website Backups

```text
Website
   ↓
Archive
   ↓
Remote Backup
```

### Log Backups

```text
Application Logs
   ↓
Compress
   ↓
Transfer
```

### Configuration Backups

```text
Configuration Files
   ↓
Archive
   ↓
Central Storage
```

### Scheduled Automation

The script could later be executed automatically through cron:

```text
Cron
  ↓
Bash Script
  ↓
Archive
  ↓
Remote Backup
```

---

# 41. Relationship With Previous Challenges

This task builds on earlier Linux and DevOps concepts:

```text
Linux Users
Linux Permissions
SSH
Cron
Bash
Services
Ansible
```

Day 10 brings several of these concepts together into a practical automation workflow.

---

# 42. Security Lessons

### Protect private keys

Never expose:

```text
~/.ssh/id_rsa
```

### Public keys are different

It is normal to install:

```text
~/.ssh/id_rsa.pub
```

on the remote server.

### Avoid hard-coded passwords

Do not store credentials directly inside scripts.

### Avoid unnecessary sudo

Use only the privileges actually required.

### Prefer key-based authentication

SSH keys are well suited to non-interactive automation.

---

# 43. Final Mental Model

Remember Day 10 as:

```text
SOURCE
/var/www/html/news
        |
        | zip -r
        ↓
LOCAL ARCHIVE
/archives/xfusioncorp_news.zip
        |
        | scp
        | SSH Key Authentication
        ↓
REMOTE ARCHIVE
ststor01:/archives/xfusioncorp_news.zip
```

---

# 44. Most Important Takeaways

### `zip -r`

Creates a recursive ZIP archive.

```bash
zip -r archive.zip directory
```

### `ssh`

Connects to a remote server.

```bash
ssh user@server
```

### `ssh-keygen`

Generates SSH keys.

```bash
ssh-keygen
```

### `ssh-copy-id`

Installs a public key on a remote server.

```bash
ssh-copy-id user@server
```

### `scp`

Securely copies files between systems.

```bash
scp file user@server:/path/
```

### `chmod +x`

Adds execute permission.

```bash
chmod +x script.sh
```

---

# 45. Final DevOps Lesson

The most important lesson from Day 10 is:

> **Automation should be repeatable, secure, and non-interactive.**

Instead of manually:

```text
Create archive
Copy archive
Enter password
Verify archive
```

we create:

```text
/scripts/news_archive.sh
```

and allow the script to perform the workflow automatically.

This is one of the fundamental principles behind modern DevOps automation.
