# Day 10 — Bash Script for Website Content Archiving

## 📌 Challenge Overview

The production support team of **xFusionCorp Industries** required a Bash script to automate the archiving of website content files.

The static website is hosted on **App Server 2** in the **Stratos Datacenter**.

The requirement was to create a Bash script named:

```text
news_archive.sh
```

and place it under:

```text
/scripts/
```

The script must:

- Create a ZIP archive of `/var/www/html/news`
- Name the archive `xfusioncorp_news.zip`
- Store the archive in `/archives/` on App Server 2
- Copy the archive to the Nautilus Storage Server under `/archives/`
- Perform the remote copy without asking for a password
- Allow the respective server user to execute the script
- Not use `sudo` inside the script
- Use the `zip` package, which must be installed manually before executing the script

---

## 🎯 Objective

The final workflow is:

```text
App Server 2
     |
     | /var/www/html/news
     |
     | zip -r
     v
/archives/xfusioncorp_news.zip
     |
     | scp
     | SSH key-based authentication
     v
Nautilus Storage Server
     |
     v
/archives/xfusioncorp_news.zip
```

---

## 🖥️ Server Details

| Component | Details |
|---|---|
| Application Server | App Server 2 |
| Hostname | `stapp02` |
| Website Directory | `/var/www/html/news` |
| Script Directory | `/scripts/` |
| Script Name | `news_archive.sh` |
| Local Archive Directory | `/archives/` |
| Archive Name | `xfusioncorp_news.zip` |
| Storage Server | Nautilus Storage Server |
| Storage Hostname | `ststor01` |
| Storage User | `natasha` |
| Remote Archive Directory | `/archives/` |

---

# 🔧 Implementation

## 1. Connect to App Server 2

```bash
ssh <app-server-user>@stapp02
```

---

## 2. Install ZIP

The challenge explicitly requires the `zip` package to be installed manually outside the script.

```bash
sudo yum install -y zip
```

Verify:

```bash
zip -v
```

The installation command is **not included inside the Bash script**.

---

## 3. Create Required Directories

Create the script directory:

```bash
sudo mkdir -p /scripts
```

Create the archive directory:

```bash
sudo mkdir -p /archives
```

---

## 4. Configure Passwordless SSH

The script uses `scp` to copy the archive to the Storage Server.

Generate an SSH key if required:

```bash
ssh-keygen -t rsa -b 4096
```

Copy the public key to the Storage Server:

```bash
ssh-copy-id natasha@ststor01
```

Test:

```bash
ssh natasha@ststor01
```

The SSH connection should work without requesting the `natasha` password.

This is required because the Bash script must be non-interactive.

---

# 📝 Create the Script

Create:

```bash
vi /scripts/news_archive.sh
```

Use the following script:

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

# 🔍 Script Explanation

### Shebang

```bash
#!/bin/bash
```

Specifies that Bash should be used to execute the script.

### Source Directory

```bash
SOURCE_DIR="/var/www/html/news"
```

Stores the location of the website content.

### Archive Directory

```bash
ARCHIVE_DIR="/archives"
```

Defines where the local archive will be stored.

### Archive Name

```bash
ARCHIVE_NAME="xfusioncorp_news.zip"
```

Defines the required archive filename.

### Remote User

```bash
REMOTE_USER="natasha"
```

Defines the account used on the Storage Server.

### Remote Host

```bash
REMOTE_HOST="ststor01"
```

Defines the Storage Server hostname.

### Remote Directory

```bash
REMOTE_DIR="/archives"
```

Defines the destination directory on the Storage Server.

---

# 📦 Creating the Archive

```bash
zip -r "$ARCHIVE_DIR/$ARCHIVE_NAME" "$SOURCE_DIR"
```

This creates:

```text
/archives/xfusioncorp_news.zip
```

from:

```text
/var/www/html/news
```

The `-r` option means **recursive**, so files and subdirectories inside the website directory are included.

---

# 🚚 Copying the Archive

```bash
scp "$ARCHIVE_DIR/$ARCHIVE_NAME" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
```

This transfers:

```text
/archives/xfusioncorp_news.zip
```

to:

```text
natasha@ststor01:/archives/
```

Because SSH key-based authentication was configured, SCP does not require an interactive password.

---

# 🔐 Make the Script Executable

```bash
chmod +x /scripts/news_archive.sh
```

Verify:

```bash
ls -l /scripts/news_archive.sh
```

The permissions should contain `x`.

---

# ▶️ Execute the Script

```bash
/scripts/news_archive.sh
```

The script performs the complete workflow:

```text
1. Ensure /archives exists
2. Read /var/www/html/news
3. Create xfusioncorp_news.zip
4. Store it in /archives/
5. Transfer it using SCP
6. Store the copy on ststor01:/archives/
```

---

# ✅ Verification

## Verify Local Archive

```bash
ls -lh /archives/
```

Expected:

```text
xfusioncorp_news.zip
```

---

## Verify Archive Contents

```bash
unzip -l /archives/xfusioncorp_news.zip
```

This confirms that the website files were included in the archive.

---

## Verify Remote Archive

```bash
ssh natasha@ststor01
```

Then:

```bash
ls -lh /archives/
```

Expected:

```text
xfusioncorp_news.zip
```

Exit:

```bash
exit
```

---

# 📂 Final Structure

### App Server 2

```text
/scripts/
└── news_archive.sh

/archives/
└── xfusioncorp_news.zip
```

### Nautilus Storage Server

```text
/archives/
└── xfusioncorp_news.zip
```

---

# 📚 Concepts Practiced

- Bash scripting
- Bash variables
- Linux file permissions
- ZIP archiving
- SSH
- SSH key-based authentication
- Passwordless authentication
- SCP
- Remote file transfer
- Server-to-server communication
- Backup concepts
- Non-interactive automation

---

# 🏆 Result

**Day 10 completed successfully.**

The website content was archived into `xfusioncorp_news.zip`, stored locally, and copied to the Nautilus Storage Server using passwordless SSH authentication.

The script uses no `sudo` and is executable by the appropriate server user.
