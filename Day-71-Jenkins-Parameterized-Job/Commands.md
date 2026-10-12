# Day 71 — Commands and Jenkins UI Runbook

> **Scope:** Training-lab runbook for `install-packages`. Perform each block on the machine indicated. No real passwords or private keys are included. Only modify the designated Storage Server.

## A. Configure the Jenkins Freestyle job (web UI)

```text
Jenkins Dashboard → New Item
Name: install-packages
Type: Freestyle project

General → This project is parameterized: YES
Add Parameter → String Parameter
  Name: PACKAGE
  Default Value: vim-enhanced
  Description: Name of the Linux package to install on the Storage Server
  Trim the string: YES

Build Steps → Add build step → Execute shell
```

## B. Inspect Storage Server (from jump host)

```bash
ssh natasha@ststor01
hostname
cat /etc/os-release
command -v dnf || command -v yum || command -v apt-get
sudo -n true
exit
```

Observed: `ststor01`, CentOS Stream 9, `/usr/bin/dnf`, and successful noninteractive sudo.

## C. Connect to Jenkins OS account (from jump host)

```bash
ssh jenkins@jenkins
whoami
hostname
echo "$HOME"
ls -la ~/.ssh/
```

Observed account: `jenkins`; home directory: `/var/lib/jenkins`. If `.ssh` is absent, create it in the next step.

## D. Set up Jenkins → Storage SSH authorization (on Jenkins server)

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Stop and reuse an existing key if this path already exists:
test ! -e ~/.ssh/id_ed25519 && test ! -e ~/.ssh/id_ed25519.pub &&   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''

ls -l ~/.ssh/
ssh-copy-id -i ~/.ssh/id_ed25519.pub natasha@ststor01
```

When accepting the server key for the first time, confirm its fingerprint through the lab's trusted infrastructure information. `ssh-copy-id` requires Storage Server authentication during bootstrap; the Jenkins job does not.

## E. Verify unattended remote access (on Jenkins server)

```bash
ssh -o BatchMode=yes natasha@ststor01 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'sudo -n true'
ssh -o BatchMode=yes natasha@ststor01 'command -v dnf'
```

Expected: `ststor01`, silent zero-exit sudo check, and `/usr/bin/dnf`. If any command fails, resolve it before creating the build step.

## F. Paste into Jenkins → Build Steps → Execute shell

```bash
#!/bin/bash
set -euo pipefail

: "${PACKAGE:?PACKAGE parameter is required}"

if [[ ! "$PACKAGE" =~ ^[A-Za-z0-9][A-Za-z0-9._+:-]*$ ]]; then
  echo "ERROR: Invalid package name"
  exit 1
fi

echo "Installing package: $PACKAGE on ststor01"

ssh -o BatchMode=yes natasha@ststor01   "sudo -n dnf install -y -- '$PACKAGE'"

echo "Verifying installed package..."
ssh -o BatchMode=yes natasha@ststor01   "rpm -q -- '$PACKAGE'"

echo "SUCCESS: $PACKAGE installed on ststor01"
```

Click **Save**. `BatchMode=yes` prevents interactive prompts; `set -e` makes a failed SSH, install, or verification command fail the build. The package-name pattern rejects shell quotes and separators; this is basic input validation, not a complete production authorization policy.

## G. Build and inspect

```text
install-packages → Build with Parameters
PACKAGE: vim-enhanced
Click Build
Build History → #1 → Console Output
```

Successful recorded output excerpt:

```text
Complete!
Verifying installed package...
vim-enhanced-8.2.2637-41.el9.x86_64
SUCCESS: vim-enhanced installed on ststor01
Finished: SUCCESS
```

## H. Verify on Storage Server (from jump host)

```bash
ssh natasha@ststor01
rpm -q vim-enhanced
exit
```

Expected: installed `vim-enhanced` package version. This direct check is optional independent verification in addition to the successful `rpm -q` over SSH in Jenkins.

## I. Re-run to check repeatability (recommended)

```text
install-packages → Build with Parameters
PACKAGE: vim-enhanced
Build History → #2 → Console Output
Expected: Finished: SUCCESS
```

DNF generally reports that the package is already installed instead of reinstalling it. Do not claim a second run was successful unless its console output shows so.

## J. Diagnostics (run on the indicated machine)

```bash
# Jenkins server, as jenkins:
ssh -vv -o BatchMode=yes natasha@ststor01 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'sudo -n true'

# Storage Server:
command -v dnf
rpm -q vim-enhanced
sudo -n dnf info vim-enhanced
```

Avoid copying SSH private keys into a Jenkins build step, turning off SSH host-key verification, or embedding remote passwords in shell code.
