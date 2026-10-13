# Day 74 — Commands and Jenkins Configuration Runbook

> Run commands on the indicated machine. Replace any placeholder with your own authorized lab details. **Never commit passwords, SSH private keys, or SQL dumps.**

## 1. Create Jenkins job

1. Open Jenkins and sign in as the provided administrator.
2. **New Item → `database-backup` → Freestyle project → OK**.
3. **Triggers → Build periodically** and paste:

```cron
*/10 * * * *
```

4. Configure **Build Steps → Execute shell** after setting up the remote prerequisites below.

## 2. Verify App Server 1

From the jump host:

```bash
ssh tony@stapp01
hostname
command -v mariadb-dump || command -v mysqldump
command -v mariadb || command -v mysql
```

Create a restricted client option file **only if it does not exist**:

```bash
umask 077
read -rsp 'Enter database password: ' DB_PASS; echo
printf '[client]\nuser=kodekloud_roy\npassword=%s\n' "$DB_PASS" > "$HOME/.db-backup.cnf"
unset DB_PASS
chmod 600 "$HOME/.db-backup.cnf"
```

For passwords containing option-file control characters (e.g. `#`, `\`, quotes), use a secure tool or carefully escaped option-file syntax instead of the simple `printf` pattern above. Avoid putting credentials on the command line where they may appear in process listings.

Test connectivity using whichever client is installed:

```bash
mariadb --defaults-extra-file="$HOME/.db-backup.cnf" -e "SHOW DATABASES LIKE 'kodekloud_db01';"
# Or: mysql --defaults-extra-file="$HOME/.db-backup.cnf" -e "SHOW DATABASES LIKE 'kodekloud_db01';"
```

A test dump (example using MariaDB):

```bash
mariadb-dump --defaults-extra-file="$HOME/.db-backup.cnf" --single-transaction kodekloud_db01 > /dev/null
echo $?
exit
```

**Note:** `--defaults-extra-file` must appear immediately after the executable in the command options above.

## 3. Prepare the Storage Server

```bash
ssh natasha@ststor01
mkdir -p /home/natasha/db_backups
chmod 700 /home/natasha/db_backups
ls -ld /home/natasha/db_backups
exit
```

## 4. Configure Jenkins SSH access

```bash
ssh jenkins@jenkins
whoami
printf 'Home: %s\n' "$HOME"
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ls -l ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub 2>/dev/null || true
```

Only if no key pair exists:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
```

Verify trusted SSH server fingerprints **before** accepting them:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub tony@stapp01
ssh-copy-id -i ~/.ssh/id_ed25519.pub natasha@ststor01
ssh -o BatchMode=yes tony@stapp01 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'hostname'
ssh -o BatchMode=yes tony@stapp01 'test -r /home/tony/.db-backup.cnf'
ssh -o BatchMode=yes natasha@ststor01 'test -w /home/natasha/db_backups'
```

## 5. Jenkins Execute shell script

Paste into **database-backup → Configure → Build Steps → Execute shell**:

```bash
#!/bin/bash
set -euo pipefail
umask 077

DB_NAME='kodekloud_db01'
SOURCE='tony@stapp01'
DEST='natasha@ststor01'
DEST_DIR='/home/natasha/db_backups'
BACKUP_FILE="db_$(date +%F).sql"

# Local temporary file on Jenkins is deleted on all normal shell exits.
LOCAL_TMP=$(mktemp "${WORKSPACE:-/tmp}/dbbackup.XXXXXX")
trap 'rm -f "$LOCAL_TMP"' EXIT

echo "Starting backup of $DB_NAME"
echo "Destination filename: $BACKUP_FILE"

ssh -o BatchMode=yes "$SOURCE" '
  set -e
  if command -v mariadb-dump >/dev/null 2>&1; then
    mariadb-dump --defaults-extra-file=/home/tony/.db-backup.cnf \
      --single-transaction kodekloud_db01
  elif command -v mysqldump >/dev/null 2>&1; then
    mysqldump --defaults-extra-file=/home/tony/.db-backup.cnf \
      --single-transaction kodekloud_db01
  else
    echo "No SQL dump utility available" >&2
    exit 1
  fi
' > "$LOCAL_TMP"

test -s "$LOCAL_TMP"

# Stage transfer under a .part extension first.
scp -q -o BatchMode=yes "$LOCAL_TMP" "$DEST:$DEST_DIR/$BACKUP_FILE.part"

# Publish only after transfer is complete.
ssh -o BatchMode=yes "$DEST" "
  set -e
  chmod 600 '$DEST_DIR/$BACKUP_FILE.part'
  test -s '$DEST_DIR/$BACKUP_FILE.part'
  mv -f '$DEST_DIR/$BACKUP_FILE.part' '$DEST_DIR/$BACKUP_FILE'
  ls -lh '$DEST_DIR/$BACKUP_FILE'
"

echo "SUCCESS: $DEST_DIR/$BACKUP_FILE"
```

The filename format uses the Jenkins machine's local date. Ensure expected timezone alignment with the grading environment.

## 6. Run and verify

1. Click **Save** on the job.
2. Click **Build Now**.
3. Open **Build History → build → Console Output**.
4. Confirm the job finishes successfully.

Independently on Storage Server:

```bash
ssh natasha@ststor01
ls -lh /home/natasha/db_backups
BACKUP="/home/natasha/db_backups/db_$(date +%F).sql"
test -s "$BACKUP" && echo 'Backup exists and is nonempty'
head -n 5 "$BACKUP"   # Inspect headers only; avoid revealing sensitive rows
exit
```

Do **not** restore a test backup over a live database. Perform full restore validation only in a separate authorized test database/environment.

## Troubleshooting quick reference

| Symptom | Check |
|---|---|
| SSH permission denied | Jenkins service account key and remote `authorized_keys` |
| Host key verification failed | Trusted host fingerprint / `known_hosts` |
| Access denied for database user | Username, option-file password, grants, database host |
| Dump command not found | `command -v mariadb-dump || command -v mysqldump` |
| Destination permission denied | Ownership/mode of `/home/natasha/db_backups` |
| Empty or partial output | Exit status, `pipefail`, disk availability, `.part` file |
| Cron not triggering | Exact `*/10 * * * *`, job enabled, Jenkins running |
