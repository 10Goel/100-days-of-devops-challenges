# Day 73 — Jenkins Scheduling, SSH, and Centralized Apache Log Collection: Notes

## 1. What problem are we solving?

Apache emits HTTP request data to `access_log` and server/runtime diagnostic messages to `error_log`. When logs are scattered across multiple hosts, diagnosis becomes slow. This lab uses Jenkins as a temporary scheduler to move copies from one app server into one storage directory.

**Source:** `stapp03:/var/log/httpd/{access_log,error_log}`  
**Destination:** `ststor01:/usr/src/itadmin/`  
**Job:** `copy-logs` every seven minutes.

This is a **periodic snapshot copy**, not real-time centralized log management. A production logging platform might use Fluent Bit, Filebeat, Vector, journald forwarding, OpenSearch, Elasticsearch, Loki, or another managed observability stack.

## 2. Jenkins jobs, builds, triggers

- **Job:** Saved definition of work Jenkins can execute.
- **Build:** One execution of the job with its own logs and status.
- **Freestyle project:** UI-configured sequence of build steps, with optional triggers.
- **Build periodically:** Runs according to a cron-like calendar schedule independently of SCM changes.
- **Poll SCM:** Checks repositories for changes; *not* what Day 73 requests.
- **Workspace:** Jenkins-local working directory for a job. In this challenge, the actual target files are on `ststor01`, not in the workspace.

### Cron syntax

```text
minute  hour  day-of-month  month  day-of-week
  */7     *         *         *         *
```

`*/7` in the minute field fires at `00,07,14,21,28,35,42,49,56` each hour, then again at `00` next hour. So there is a **four-minute gap** at the hour boundary. `H/7` is Jenkins' hash-balanced schedule and may shift minute positions; use the explicit value the lab requires.

## 3. SSH authentication and trust

SSH provides encrypted communication and remote command execution.

```text
Jenkins private key                   Remote public key
/var/lib/jenkins/.ssh/id_ed25519 ---> ~/.ssh/authorized_keys
     never copied                     on banner and natasha accounts
```

- **Private key:** Confidential proof held by the client (Jenkins).
- **Public key:** Placed in the remote user's `authorized_keys`.
- **Host key:** Identifies the server; Jenkins stores approved identities in `known_hosts`.
- **`ssh-copy-id`:** Conveniently appends a public key to a remote user's allowed keys.
- **`BatchMode=yes`:** Disables interactive password/passphrase prompts so builds fail deterministically when SSH authentication is not ready.
- **Host fingerprint validation:** Guards against connecting to an impostor. Do not turn off strict host-key checking as a generic fix.

In production, store/manage credentials in Jenkins Credentials, rotate them, and scope access narrowly. Day 73's lab-specific key approach is an illustrative setup.

## 4. Why use `sudo -n`?

Apache logs may be readable only by root or a privileged service group. The destination directory can likewise require administrative writes. `sudo -n` is **noninteractive**: if a password would be needed, the command fails immediately instead of blocking a Jenkins job.

For a production deployment, prefer narrowly scoped allowed commands and restricted service accounts; do not grant unrestricted NOPASSWD sudo without appropriate controls.

## 5. Understanding the transfer command

```bash
ssh banner@stapp03 'sudo -n cat /var/log/httpd/access_log' \
| ssh natasha@ststor01 'sudo -n tee /usr/src/itadmin/access_log >/dev/null'
```

1. Jenkins opens SSH session A to `stapp03`.
2. `sudo cat` streams the source file's bytes to standard output.
3. The local pipe sends those bytes to SSH session B's standard input.
4. `sudo tee` writes the received bytes to a destination file.
5. `>/dev/null` discards the echoed duplicate output of `tee` while keeping file writes.

The file contents stream through Jenkins; they need not persist as a local workspace file.

### `tee` versus redirection

```bash
sudo echo hello > /protected/file    # Often fails: the *shell* performs the redirect
printf 'hello\n' | sudo tee /protected/file
```

The second form executes the file write under the elevated `tee` process.

### `tee` overwrites by default

`tee /path` truncates and replaces the destination with the received stream. `tee -a` appends; blindly appending an entire source log every seven minutes would duplicate existing lines. The challenge asks for copies, so replacement is appropriate.

## 6. Exit codes and safe Bash practices

```bash
set -euo pipefail
```

| Option | Effect |
|---|---|
| `-e` | Causes script to exit for many unhandled failed commands |
| `-u` | Treats unset variables as errors |
| `pipefail` | Makes a pipeline fail when *any* command in it fails |

Without `pipefail`, an SSH read failure could be hidden by a successful final `tee` process. Even with these options, test meaningful outputs rather than assuming all logical conditions are met.

## 7. Why verification matters

**A green Jenkins build is not enough unless it checks the required destination.**

Useful checks:

```bash
sudo ls -lh /usr/src/itadmin/access_log /usr/src/itadmin/error_log
sudo test -f /usr/src/itadmin/access_log
sudo test -f /usr/src/itadmin/error_log
```

For stronger verification, compare source/destination hashes if the source file is stable during transfer. File size and existence alone do not guarantee identical bytes.

## 8. Troubleshooting flow

```text
Scheduled job not firing?
  -> Is Build periodically enabled and saved?
  -> Is cron syntax correct?
  -> Is Jenkins available and scheduling builds?

Job runs but fails?
  -> SSH from Jenkins to both hosts works?
  -> Host fingerprints trusted?
  -> sudo -n works for the necessary actions?
  -> Apache log files exist and are readable?
  -> Destination exists and is writable?
  -> Console output contains an SSH, sudo or pipeline error?

Job succeeds but copies look wrong?
  -> Was the correct server/path used?
  -> Are files overwritten versus appended?
  -> Did the source log rotate or change mid-copy?
```

## 9. Operational limitations and improvements

**This approach is intentionally basic.** Key production considerations include:

- **Rotation:** `access_log` and `error_log` can grow or rotate during the copy.
- **Retention:** Repeated overwrites destroy historical snapshots; consider dated objects and lifecycle policies.
- **Concurrency:** Multiple builds writing the same files can race; disable concurrent builds or lock the copy process.
- **Security:** Apache logs can include sensitive request data; restrict access and do not expose raw logs in public Jenkins consoles or GitHub.
- **Scale:** Streaming complete files every seven minutes is inefficient for large logs. Agents, incremental shipping, and queue/buffer-based forwarding are more robust.
- **Observability:** Add alerts, destination free-space checks, checksum/integrity validation, and failure notifications.
- **Credential lifecycle:** Prefer centralized secrets management instead of ad hoc persistent private keys.

## 10. Interview questions and answers

**Q1. What is a Jenkins periodic build?**  
A scheduler-triggered job using Jenkins cron syntax, regardless of whether a Git repository changes.

**Q2. What's the difference between `*/7` and `H/7`?**  
`*/7` runs at fixed seven-minute marks within each hour; `H/7` uses a job-dependent hashed offset to distribute load.

**Q3. Why is `BatchMode=yes` useful in CI?**  
It prevents jobs from hanging waiting for an interactive SSH password.

**Q4. Why use `sudo tee` instead of `sudo echo > file`?**  
Because redirection is handled by the calling shell, not by `sudo echo`; `sudo tee` performs the privileged file write itself.

**Q5. What does `pipefail` solve?**  
It prevents an earlier failing command in a pipeline from being masked by a successful last command.

**Q6. Is this an ideal production logging architecture?**  
No. It is a periodic batch copy, not incremental resilient streaming, retention, search and alerting. A log shipping agent and central platform would typically be better.

**Q7. How would you avoid duplicate logs?**  
For this snapshot model, overwrite the destination rather than append the entire source. For incremental shipping, track file offsets and rotation using an appropriate agent.

**Q8. Why verify the destination separately?**  
It proves the files exist where the task needs them, not just that Jenkins ran a shell step successfully.

## 11. Outcome

**Day 73 was reported as passed.** This documentation describes the verified lab requirements and the solution procedure; it does not fabricate build numbers, console transcripts, file hashes or measured copy times.

---

Return to [README.md](README.md) or follow the executable procedure in [Commands.md](Commands.md).
