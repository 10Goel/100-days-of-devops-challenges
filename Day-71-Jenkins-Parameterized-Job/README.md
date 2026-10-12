# Day 71 — Jenkins Parameterized Job for Remote Package Installation

![Jenkins](https://img.shields.io/badge/Jenkins-Freestyle%20Job-D24939?logo=jenkins&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-Ubuntu%20%2B%20CentOS%20Stream%209-333) ![Status](https://img.shields.io/badge/KodeKloud-Passed-success) ![Automation](https://img.shields.io/badge/Automation-SSH%20%2B%20DNF-blue)

## Overview

**100 Days of DevOps — Day 71**: Built and successfully executed a **parameterized Jenkins Freestyle job** named `install-packages` to install user-specified Linux packages on the **Storage Server** (`ststor01`), rather than on the Jenkins controller. The job used key-based SSH authentication, noninteractive sudo, DNF, and RPM verification. The KodeKloud challenge was **passed**.

> **Security:** This public write-up omits all lab passwords, SSH private keys, and internal secrets. Hostnames and usernames refer to an ephemeral training environment.

## Task Requirements

| Requirement | Implementation |
|---|---|
| Job name | `install-packages` |
| Job type | Freestyle project |
| Build parameter | String parameter, exactly `PACKAGE` |
| Target | Storage Server `ststor01` |
| Automation | Install the supplied `$PACKAGE` remotely |
| Example test | `PACKAGE=vim-enhanced` |
| Validation | Run at least one build and verify installation on Storage Server |
| Outcome | **Passed** |

## Architecture and Execution Flow

```text
Operator → Jenkins UI → Build with Parameters (PACKAGE=vim-enhanced)
                           │
                           ▼
              Freestyle job: install-packages
                           │
                           ▼
                 Execute shell build step
                           │
             key-based SSH (noninteractive)
                           │
                           ▼
                natasha@ststor01 (CentOS 9)
                           │
                           ▼
                sudo -n dnf install -y
                           │
                           ▼
                  rpm -q package name
                           │
                           ▼
                  Jenkins: SUCCESS / FAIL
```

**Execution environment:** Jenkins ran on an Ubuntu 24.04 server under the `jenkins` OS account; the remote Storage Server used CentOS Stream 9 and `dnf`.

## Implementation

### 1. Create the Jenkins job

1. Sign in to Jenkins with the lab-provided administrator account.
2. Select **New Item → `install-packages` → Freestyle project → OK**.
3. Under **General**, enable **This project is parameterized**.
4. Add a **String Parameter**:

   | Property | Value |
   |---|---|
   | Name | `PACKAGE` |
   | Default value | `vim-enhanced` |
   | Description | Name of the Linux package to install on the Storage Server |
   | Trim the string | Enabled |

### 2. Validate remote prerequisites

On the Storage Server, confirm the hostname, OS, package manager, and noninteractive sudo:

```bash
hostname
cat /etc/os-release
command -v dnf
sudo -n true
```

The observed environment returned `ststor01`, CentOS Stream 9, `/usr/bin/dnf`, and successful passwordless sudo.

### 3. Establish SSH access for the Jenkins service account

The `jenkins` user's home was `/var/lib/jenkins`. No `.ssh` directory initially existed. A dedicated ED25519 key pair was generated on the Jenkins server, and its **public key** was authorized on the Storage Server. The private key stayed on the Jenkins server.

```bash
# Run as the jenkins OS user on the Jenkins server
mkdir -p ~/.ssh && chmod 700 ~/.ssh
# Only if a key does not already exist; do not overwrite existing keys
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
ssh-copy-id -i ~/.ssh/id_ed25519.pub natasha@ststor01

ssh -o BatchMode=yes natasha@ststor01 'hostname'
ssh -o BatchMode=yes natasha@ststor01 'sudo -n true'
ssh -o BatchMode=yes natasha@ststor01 'command -v dnf'
```

**Observed verification:** SSH returned `ststor01`; remote sudo exited successfully; package manager was `/usr/bin/dnf`.

### 4. Add the Execute shell build step

Under **Build Steps → Add build step → Execute shell**, use:

```bash
#!/bin/bash
set -euo pipefail

: "${PACKAGE:?PACKAGE parameter is required}"

# Reject whitespace, shell metacharacters and additional command arguments.
if [[ ! "$PACKAGE" =~ ^[A-Za-z0-9][A-Za-z0-9._+:-]*$ ]]; then
    echo "ERROR: Invalid package name"
    exit 1
fi

echo "Installing package: $PACKAGE on ststor01"

ssh -o BatchMode=yes natasha@ststor01     "sudo -n dnf install -y -- '$PACKAGE'"

echo "Verifying installed package..."
ssh -o BatchMode=yes natasha@ststor01     "rpm -q -- '$PACKAGE'"

echo "SUCCESS: $PACKAGE installed on ststor01"
```

**Prerequisite:** This script assumes the Jenkins process executes as the OS account that owns the authorized key, the remote SSH host fingerprint has been verified, and the remote account is permitted to perform the relevant `sudo -n dnf` operation.

### 5. Run and verify

1. Click **Save**.
2. Select **Build with Parameters** and enter `vim-enhanced`.
3. Open **Build History → #1 → Console Output**.
4. Confirm the package transaction and remote RPM query complete successfully.
5. Optionally repeat the build to check idempotent package-manager behavior.

**Observed first-build output (excerpt):**

```text
Installed:
  vim-enhanced-2:8.2.2637-41.el9.x86_64
Complete!
Verifying installed package...
vim-enhanced-8.2.2637-41.el9.x86_64
SUCCESS: vim-enhanced installed on ststor01
Finished: SUCCESS
```

The user subsequently confirmed **Day 71 passed**. A second-build result was not independently supplied and is not asserted here.

## Troubleshooting Reference

| Symptom | Likely reason | Diagnostic or fix |
|---|---|---|
| `Permission denied (publickey)` | Key not authorized or wrong Jenkins OS account | Test SSH with `BatchMode=yes` as the build account |
| `Host key verification failed` | Missing or changed SSH host key | Validate fingerprint and trusted known-hosts entry |
| `sudo: a password is required` | Sudo demands interactive authentication | Use approved, narrowly scoped noninteractive sudo or a secure alternative |
| `dnf: command not found` | Wrong distribution or binary | Inspect `/etc/os-release` and `command -v dnf` |
| `PACKAGE: parameter is required` | Missing/misnamed parameter | Check exact uppercase `PACKAGE` |
| `rpm -q` fails | Package did not install, or naming mismatch | Check DNF output and package naming |
| Build is green but wrong target | Command executed on controller | Ensure install/verification commands run inside remote SSH |

## What This Challenge Demonstrates

- Parameter-driven Jenkins Freestyle automation
- Controller-to-target SSH execution and key trust
- Noninteractive privilege escalation for unattended jobs
- Basic package-name validation and shell failure handling
- Separation of orchestration (Jenkins) from execution target (Storage Server)
- Verification-driven automation using RPM exit status
- Repeatable package installation with DNF

## Production Considerations

This is a temporary educational lab. For production, prefer Jenkins-managed credentials/SSH agents or a restricted deployment identity, pin trusted host keys, restrict sudo to approved commands, authorize supported package names, avoid giving arbitrary users package-install rights, and consider tools such as Ansible for scalable fleet management.

---

**Result:** ✅ **Day 71 completed successfully (KodeKloud grader passed).**
