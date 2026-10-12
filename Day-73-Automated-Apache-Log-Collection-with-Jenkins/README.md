# Day 73 — Automated Apache Log Collection with Jenkins

![Jenkins](https://img.shields.io/badge/Jenkins-Freestyle%20Job-D24939?logo=jenkins&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-SSH%20Automation-333333?logo=linux&logoColor=white) ![Status](https://img.shields.io/badge/KodeKloud-Passed-2ea44f)

> **100 Days of DevOps | Day 73**  
> **Outcome:** Created a scheduled Jenkins Freestyle job that copies Apache access and error logs from Application Server 3 to the Storage Server every seven minutes. The KodeKloud challenge was passed.

## Challenge overview

The Nautilus DevOps team needed an interim, automated approach to centralize Apache logs for troubleshooting while a broader logging system was being planned. The solution is a Jenkins job named **`copy-logs`**.

### Acceptance criteria

| Requirement | Configuration |
|---|---|
| Jenkins job | `copy-logs` |
| Job type | Freestyle project |
| Trigger | **Build periodically:** `*/7 * * * *` |
| Source | Application Server 3: `stapp03` (`banner`) |
| Source files | `/var/log/httpd/access_log`, `/var/log/httpd/error_log` |
| Target | Storage Server: `ststor01` (`natasha`) |
| Target directory | `/usr/src/itadmin/` |
| Validation | Execute at least one build and verify both copied files |

## Architecture

```text
                        Jenkins controller
                         job: copy-logs
                      trigger: */7 * * * *
                                |
                   Execute shell (jenkins user)
                                |
                   +------------+------------+
                   |                         |
             SSH read source          SSH write destination
                   |                         |
                   v                         v
      stapp03 (banner)                ststor01 (natasha)
      /var/log/httpd/                 /usr/src/itadmin/
        access_log  ------------------>  access_log
        error_log   ------------------>  error_log
             sudo cat                      sudo tee
```

Logs travel through an SSH-to-SSH pipeline on the Jenkins execution host. This is a **snapshot copy**, not a persistent log-ingestion service.

## Implementation

### 1. Create the Jenkins job

1. Log in through the KodeKloud Jenkins UI.
2. Select **New Item** → enter `copy-logs` → **Freestyle project** → **OK**.
3. Open **Configure → Triggers**; select **Build periodically** and enter:

   ```cron
   */7 * * * *
   ```

4. Under **Build Steps**, choose **Add build step → Execute shell**.

### 2. Verify remote requirements

Connect to `stapp03` and verify Apache's two logs exist, and that `banner` can read them with noninteractive `sudo`. Check that `natasha` can write to `/usr/src/itadmin/` with noninteractive `sudo` on `ststor01`.

```bash
ssh banner@stapp03 'sudo -n ls -l /var/log/httpd/access_log /var/log/httpd/error_log'
ssh natasha@ststor01 'sudo -n mkdir -p /usr/src/itadmin'
```

Run these checks from a host with appropriate SSH authentication; for Jenkins automation the **Jenkins service account** needs working unattended SSH access to both endpoints.

### 3. Configure Jenkins-to-server SSH

Set up SSH key-based access from the Linux account executing Jenkins jobs to both `banner@stapp03` and `natasha@ststor01`. Verify server fingerprints before accepting host keys. Reuse existing keys; do not overwrite them. See [Commands.md](Commands.md) for the runbook.

### 4. Add the build script

Paste the script from **[Commands.md](Commands.md#jenkins-execute-shell-build-step)** into **Execute shell**. It:

- ensures the destination exists;
- reads each file through `sudo -n cat` on App Server 3;
- pipes the content over an authenticated SSH connection to `sudo -n tee` on Storage Server;
- checks both destination files;
- returns nonzero on a failed command/pipeline.

### 5. Run and verify

1. Save and select **Build Now**.
2. Open **Build History → build number → Console Output**; confirm **`Finished: SUCCESS`**.
3. Verify on Storage Server:

   ```bash
   ssh natasha@ststor01 \
     'sudo -n ls -l /usr/src/itadmin/access_log /usr/src/itadmin/error_log'
   ```

4. Reopen the job configuration and confirm the periodic trigger remains enabled.

**Recorded result:** The user confirmed **Day 73 Passed**. No unprovided console transcript or checksum measurements are claimed here.

## Verification checklist

- [x] Challenge marked passed by KodeKloud (user-confirmed)
- [ ] For each new lab run: job named `copy-logs` exists
- [ ] Periodic trigger is `*/7 * * * *`
- [ ] Jenkins SSH works to both source and destination
- [ ] A build reports `Finished: SUCCESS`
- [ ] `access_log` and `error_log` exist under `/usr/src/itadmin/`

## Troubleshooting

| Symptom | Diagnostic | Resolution |
|---|---|---|
| `Permission denied (publickey)` | Try `ssh -o BatchMode=yes ... hostname` as Jenkins user | Configure SSH public key and authorized keys |
| `Host key verification failed` | Inspect SSH known-hosts entry | Independently validate host fingerprint; update trusted entry if changed |
| `sudo: a password is required` | `ssh ... 'sudo -n true'` | Use an approved scoped noninteractive privilege policy |
| Apache file not found | `sudo ls /var/log/httpd/` on `stapp03` | Confirm actual Apache log path and service state |
| Pipeline silently succeeds after first command fails | Check shell options | Use `set -euo pipefail` |
| Jenkins schedule not running | Check trigger, next runs, controller clock | Save job and validate Jenkins cron syntax |
| Destination files missing | SSH + `sudo ls` on `ststor01` | Confirm remote destination, path and write rights |

## Security and operational considerations

- **Do not publish passwords, private SSH keys, or full log content.** Logs may contain client IPs, URLs and other sensitive data.
- In production, prefer **Jenkins-managed credentials**, restricted service accounts and narrowly scoped `sudo` rules over broad passwordless sudo.
- The example overwrites two destination files each run. Consider timestamped archives, rotation, retention, integrity checks, and central log collection tools for real environments.
- `*/7 * * * *` schedules at minutes 0, 7, 14, …, 56 of each hour; the hour boundary produces a 4-minute gap.
- Avoid concurrent writes to the same destination in production: consider disabling concurrent builds or adding locking.

## Key lessons

Jenkins Freestyle automation, cron triggers, SSH public-key authentication, privileged remote log access, pipelines and exit codes, log-copy verification, and the limits of periodic polling compared with centralized logging.

---

**Challenge:** KodeKloud 100 Days of DevOps — Day 73  
**Result:** Passed
