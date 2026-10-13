# Day 74 — Automated Database Backup Using Jenkins

![Jenkins](https://img.shields.io/badge/Jenkins-Freestyle-blue) ![Status](https://img.shields.io/badge/KodeKloud-Passed-brightgreen) ![Focus](https://img.shields.io/badge/Focus-Database%20Backups-informational)

## Overview

This challenge implements a **scheduled Jenkins Freestyle job** to create a logical SQL backup of `kodekloud_db01` on **Application Server 1 (`stapp01`)** and copy it to **Storage Server (`ststor01`)**. The job is scheduled with the exact cron expression `*/10 * * * *`. The KodeKloud Day 74 challenge was **passed**.

## Task requirements

| Item | Requirement |
|---|---|
| Jenkins job | `database-backup` |
| Source machine | `stapp01` (SSH user `tony`) |
| Database | `kodekloud_db01` |
| Database account | `kodekloud_roy` |
| SQL filename | `db_$(date +%F).sql` |
| Destination | `ststor01:/home/natasha/db_backups/` |
| Scheduler | `*/10 * * * *` (exact) |

> Passwords, private keys, SQL dump contents, and other secrets are deliberately excluded from this repository.

## Architecture

```text
                   Jenkins Controller / Freestyle Job
                         database-backup
                         */10 * * * *
                                  |
                    SSH: tony@stapp01
                                  v
                 App Server 1 — kodekloud_db01
                       mariadb-dump / mysqldump
                                  |
                   SQL stream to temporary file
                                  |
                        SCP over SSH
                                  v
                  Storage Server — ststor01
             /home/natasha/db_backups/db_YYYY-MM-DD.sql
```

## Implementation summary

1. Created the Jenkins **Freestyle project** named `database-backup`.
2. Enabled **Build periodically** with **`*/10 * * * *`**.
3. Verified SQL dump tooling and database connectivity on `stapp01`.
4. Prepared `/home/natasha/db_backups` with restrictive permissions on `ststor01`.
5. Established SSH key authentication from the Jenkins service account to both servers.
6. Configured **Execute shell** to generate a SQL dump, transfer it, and verify the destination file.
7. Executed the job and verified the backup before submitting the challenge.

See [Commands.md](Commands.md) for the detailed runbook and script, and [Notes.md](Notes.md) for concepts, security, and troubleshooting.

## Jenkins configuration

**New Item → Freestyle project → `database-backup`**

**Triggers → Build periodically**:

```cron
*/10 * * * *
```

**Build Steps → Execute shell**: use the reviewed script in [Commands.md](Commands.md).

## Verification

The intended verification checks are:

```bash
# On Storage Server as natasha
ls -lh /home/natasha/db_backups/
test -s "/home/natasha/db_backups/db_$(date +%F).sql" && echo 'Nonempty backup found'
```

Also confirm Jenkins Console Output ends with `Finished: SUCCESS` and that the job's trigger remains set to the exact required expression. The challenge's **Passed** result is confirmed; precise run numbers, backup sizes, and database versions were not captured here and are not fabricated.

## Operational cautions

- SQL dumps often contain confidential data; restrict access to keys, client configuration, and backup files.
- `--single-transaction` gives a consistent snapshot for transactional tables such as InnoDB, subject to concurrent DDL and engine limitations.
- Backups must eventually be **restored and tested** in an isolated environment to establish recoverability.
- A date-only filename overwrites an existing backup from the same date on later successful runs. For production, implement versioned backups and retention.
- Prefer a secret manager or Jenkins Credentials in production rather than long-lived credentials saved in a local configuration file.
- The task specifies App Server 1 as the source even though other environments may host their database on a dedicated database server.

## Learning outcomes

Jenkins scheduled jobs · cron syntax · SSH key-based automation · SQL logical backup · secure client authentication · SCP · safe temporary-file handling · destination verification · backup security · restoration planning.

---

**Challenge:** KodeKloud 100 Days of DevOps, Day 74  
**Outcome:** Passed
