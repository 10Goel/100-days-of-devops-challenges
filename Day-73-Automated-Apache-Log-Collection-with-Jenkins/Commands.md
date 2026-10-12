# Day 73 — Commands and Jenkins Configuration Runbook

> Commands assume authorized access to the **KodeKloud Nautilus lab**. The commands must be run on the correct host. Credentials are intentionally omitted.

## 0. Lab inventory

| Role | Host | User | Purpose |
|---|---|---|---|
| Jump host | `jump-host` | `thor` | Initial shell |
| Jenkins server | `jenkins` | `jenkins` | Job executor / SSH origin |
| App Server 3 | `stapp03` | `banner` | Apache log source |
| Storage Server | `ststor01` | `natasha` | Log destination |

## 1. Verify the source files (from jump host)

```bash
ssh banner@stapp03
hostname
cat /etc/os-release
sudo -n ls -lh /var/log/httpd/access_log /var/log/httpd/error_log
sudo -n true
exit
```

**Expected:** Both log files exist, and `sudo -n true` returns exit status 0 without a password prompt.

## 2. Verify destination permissions (from jump host)

```bash
ssh natasha@ststor01
hostname
sudo -n true
sudo -n ls -ld /usr/src/itadmin
# If missing and permitted by the task:
sudo -n mkdir -p /usr/src/itadmin
exit
```

Do **not** alter log permissions indiscriminately; the task can read/write them through authorized `sudo`.

## 3. Enter Jenkins server

```bash
ssh jenkins@jenkins
whoami
hostname
echo "$HOME"
```

Expected account: `jenkins`; home generally `/var/lib/jenkins` in this lab. Jenkins jobs must run under an account with equivalent SSH access.

## 4. Prepare SSH keys (Jenkins host only)

Check for existing keys:

```bash
ls -la ~/.ssh/ 2>/dev/null || true
```

If an `id_ed25519` key pair **already exists**, reuse it; do not rerun `ssh-keygen` over it.

Only when no key pair exists:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Install the **public** key to both remote users:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub banner@stapp03
ssh-copy-id -i ~/.ssh/id_ed25519.pub natasha@ststor01
```

On each first connection, independently validate the remote SSH host fingerprint before accepting it. Supply account passwords only at the interactive prompts, never inside Jenkins scripts.

Verify unattended access:

```bash
ssh -o BatchMode=yes banner@stapp03 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'hostname'
ssh -o BatchMode=yes banner@stapp03 'sudo -n true'
ssh -o BatchMode=yes natasha@ststor01 'sudo -n true'
```

Expected hostnames: `stapp03`, `ststor01`. All four commands must succeed with exit code 0. If `sudo -n` fails, obtain the required scoped authorization; do not hardcode a password or weaken host security.

## 5. Create Jenkins Freestyle job (UI)

1. Click **Jenkins** in the KodeKloud lab UI and authenticate as the provided Jenkins administrator.
2. Choose **New Item** → Name `copy-logs` → **Freestyle project** → **OK**.
3. **Configure → Triggers → Build periodically**:

```cron
*/7 * * * *
```

4. **Configure → Build Steps → Add build step → Execute shell**.
5. Paste the following script and **Save**.

## Jenkins Execute shell build step

```bash
#!/bin/bash
set -euo pipefail

SOURCE_USER="banner"
SOURCE_HOST="stapp03"
DEST_USER="natasha"
DEST_HOST="ststor01"
DEST_DIR="/usr/src/itadmin"

echo "Starting Apache log collection..."

ssh -o BatchMode=yes "${DEST_USER}@${DEST_HOST}" \
    "sudo -n mkdir -p '${DEST_DIR}'"

for LOG_FILE in access_log error_log; do
    echo "Copying ${LOG_FILE}..."

    ssh -o BatchMode=yes "${SOURCE_USER}@${SOURCE_HOST}" \
        "sudo -n cat '/var/log/httpd/${LOG_FILE}'" \
    | ssh -o BatchMode=yes "${DEST_USER}@${DEST_HOST}" \
        "sudo -n tee '${DEST_DIR}/${LOG_FILE}' > /dev/null"

    ssh -o BatchMode=yes "${DEST_USER}@${DEST_HOST}" \
        "sudo -n test -f '${DEST_DIR}/${LOG_FILE}'"

    echo "${LOG_FILE} copied successfully."
done

echo "Verifying destination files..."
ssh -o BatchMode=yes "${DEST_USER}@${DEST_HOST}" \
    "sudo -n ls -lh '${DEST_DIR}/access_log' '${DEST_DIR}/error_log'"

echo "SUCCESS: Both Apache log files copied to Storage Server."
```

### Why these commands?

- `BatchMode=yes` fails fast rather than asking for interactive SSH credentials.
- `sudo -n` fails if a privilege prompt would be needed.
- `sudo cat` reads source files as an authorized privileged account.
- `sudo tee` writes destination files with appropriate privileges.
- `set -euo pipefail` propagates errors, including failures in either command of the pipe.
- The fixed allowlist of file names (`access_log`, `error_log`) avoids passing untrusted file paths to remote shells.

> Note: This snapshot-copy implementation truncates/replaces destination files on each execution. It is appropriate to the lab objective, not a production retention system.

## 6. Build and inspect Jenkins job (UI)

1. Open **copy-logs → Build Now**.
2. Select the new build under **Build History → Console Output**.
3. Confirm the log-copy messages, file verification and `Finished: SUCCESS`.
4. Verify the trigger remains `*/7 * * * *` under **Configure → Triggers**.

### Representative output (illustrative)

```text
Starting Apache log collection...
Copying access_log...
access_log copied successfully.
Copying error_log...
error_log copied successfully.
Verifying destination files...
...
SUCCESS: Both Apache log files copied to Storage Server.
Finished: SUCCESS
```

## 7. Verify files independently (Storage Server)

```bash
ssh natasha@ststor01
sudo -n ls -lh /usr/src/itadmin/access_log /usr/src/itadmin/error_log
sudo -n head -n 5 /usr/src/itadmin/access_log
sudo -n head -n 5 /usr/src/itadmin/error_log
exit
```

**Do not paste real customer log contents into a public repository.**

### Optional integrity check

Compare file hashes measured from both source and destination when Apache is not actively changing the files:

```bash
# On App Server 3
sudo -n sha256sum /var/log/httpd/access_log /var/log/httpd/error_log

# On Storage Server
sudo -n sha256sum /usr/src/itadmin/access_log /usr/src/itadmin/error_log
```

Run these on their **respective** hosts; if a source log changes during the transfer, byte-for-byte hashes can legitimately differ. A consistent snapshot requires log rotation/snapshot coordination.

## 8. Diagnose failures

```bash
# SSH tests (from Jenkins host)
ssh -o BatchMode=yes banner@stapp03 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'hostname'

# Remote read access (from Jenkins host)
ssh -o BatchMode=yes banner@stapp03 \
  'sudo -n test -f /var/log/httpd/access_log && sudo -n test -f /var/log/httpd/error_log'

# Remote write access (from Jenkins host)
ssh -o BatchMode=yes natasha@ststor01 \
  'sudo -n test -d /usr/src/itadmin'
```

If Jenkins reports **No valid crumb**, reload a fresh authenticated configuration form rather than disabling CSRF protection. For schedule issues, confirm that **Build periodically** (not Poll SCM) is enabled and saved.

## 9. Submission

When both files exist on Storage Server and Jenkins reports success, submit the KodeKloud task. **Day 73 was confirmed passed by the user.**
