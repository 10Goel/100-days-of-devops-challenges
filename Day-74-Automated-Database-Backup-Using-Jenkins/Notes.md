# Day 74 — Jenkins Database Backup Notes

## 1. What is a database backup?

A backup is a copy of data and associated metadata intended for recovery. A **logical backup** uses database-level tools to export schema and data into statements or another logical format. A **physical backup** copies underlying files or storage snapshots with engine-specific consistency rules.

A SQL dump is not a substitute for a tested recovery plan.

## 2. Why Jenkins?

Jenkins can run repeatable maintenance tasks on a schedule, preserve build history, report failures, and invoke shell scripts or pipelines. Although the job is a **Freestyle project**, the same workflow could later be represented in a version-controlled Jenkinsfile.

### Freestyle job lifecycle

```text
Timer trigger --> Jenkins queues job --> Executor runs shell step
       --> SSH to database host --> SQL dump --> transfer to storage
       --> validate --> Jenkins SUCCESS / FAILURE
```

## 3. Jenkins cron syntax

The required expression is:

```cron
*/10 * * * *
```

Fields: minute, hour, day-of-month, month, day-of-week. `*/10` runs at minute 0, 10, 20, 30, 40 and 50 each hour. Jenkins also supports `H/10` to distribute start times, **but the task explicitly requires `*/10`**. A scheduled run is not proof a backup is usable: validation still matters.

## 4. Database tooling

- `mysqldump` exports logical SQL from MySQL-compatible databases.
- `mariadb-dump` serves the analogous MariaDB use case.
- `--single-transaction` requests a consistent transactional snapshot (mainly useful with InnoDB); concurrent DDL and nontransactional engines can compromise guarantees.
- Dump accounts need appropriate permissions to read requested tables and metadata; exact privileges vary by server version, objects and flags.
- `--defaults-extra-file` reads client credentials from a restricted option file and must appear early in the invocation (immediately following the command in our examples).

Do not use an interactive password prompt in an unattended Jenkins build.

## 5. Authentication, SSH and SCP

`ssh-keygen` creates a private/public key pair. The Jenkins private key remains on the Jenkins server; `ssh-copy-id` adds the public key to a remote user's `authorized_keys`. SSH validates the destination host key through `known_hosts`. SCP copies files over an authenticated SSH transport.

**Production practice:** Use Jenkins Credentials or a managed secrets solution, deploy least-privilege identities, rotate secrets, and restrict network egress.

## 6. Security of database backups

Database dumps may contain API tokens, customer records or other sensitive content. Protect **at rest** and **in transit**:

- Prefer encryption and restricted backups with explicit ownership.
- Keep temporary copies out of shared workspaces where possible.
- Use `umask 077`, `chmod 600` for files, `chmod 700` for private directories.
- Keep credentials out of scripts, Console Output, Git history and build parameters.
- Control access to Jenkins jobs; job configuration or script modification can expose credentials.

## 7. Temporary files and atomic publication

`mktemp` creates an unpredictable temporary path. `trap ... EXIT` attempts cleanup when the shell exits. Uploading first to `*.part` keeps unfinished transfers visibly separate. Renaming the completed `.part` file into `db_YYYY-MM-DD.sql` gives atomic publication when the rename is within the same filesystem.

The final file uses a **date-only** name. Re-running on the same date replaces that day's prior copy. For production, use unique timestamps or versioned object storage with retention.

## 8. Basic verification vs true recovery verification

A green Jenkins build and `test -s` show that a nonempty file was created and transferred. They **do not prove the backup is restorable**. More advanced validation includes checksums, SQL dump structural checks, isolated restore tests, consistency checks and documented Recovery Point Objective (RPO) / Recovery Time Objective (RTO).

## 9. Failure modes and diagnosis

| Failure | Typical symptom | Investigation |
|---|---|---|
| Authentication | `Access denied` | Check user, grants and restricted credential file |
| Missing utility | `command not found` | Verify client/dump installation on App Server |
| SSH | `Permission denied (publickey)` | Jenkins key, permissions, target login |
| Host trust | Host key verification failure | Confirm trusted fingerprint |
| Storage | `No space left on device` | Free space and retention policy |
| Permissions | `Permission denied` writing backup | User, directory mode, ownership |
| Cron | Build never runs | Job enabled, trigger saved, Jenkins service health |
| Partial dump | Small / truncated file | SQL errors, process status, transfer logs |

## 10. Operational improvements beyond the lab

1. Move credentials to a secret manager / Jenkins Credentials binding.
2. Avoid writing plaintext backup data onto the controller; use an ephemeral agent or direct authenticated transfer where practical.
3. Encrypt backup files and implement retention, off-site copies and immutability.
4. Record checksums, timestamps, source version and job execution metadata.
5. Alert on errors, missed backups and abnormal size changes.
6. Perform periodic **restore drills** in an isolated environment.
7. Add mutually exclusive execution or locking to avoid overlapping runs.

## 11. Interview questions

**Q: Is a SQL dump a physical backup?**  
A: No. It is a logical export of database objects and data.

**Q: Why use `--single-transaction`?**  
A: To obtain a transactionally consistent snapshot on supported engines without extensive blocking, with important caveats for nontransactional tables and concurrent DDL.

**Q: How does Jenkins decide whether a shell build succeeds?**  
A: Primarily from the build step's process exit status; use `set -euo pipefail` and explicitly check critical outputs.

**Q: What is the difference between `*/10` and `H/10`?**  
A: The former selects explicit ten-minute boundaries; the latter hashes scheduling to distribute job start times. The lab requires `*/10`.

**Q: Why use a `.part` extension?**  
A: It prevents a partially uploaded file from being treated as the final completed backup.

**Q: What makes a backup trustworthy?**  
A: Controlled generation, protected storage, monitoring and successful restore testing against recovery objectives.

---

**Day 74 result:** KodeKloud challenge passed. Exact console logs were not supplied and are not invented.
