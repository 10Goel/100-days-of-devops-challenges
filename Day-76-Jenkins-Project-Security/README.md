# Day 76 — Jenkins Job-Level Matrix Authorization

![Jenkins](https://img.shields.io/badge/Jenkins-Job%20Security-D24939?logo=jenkins&logoColor=white) ![Status](https://img.shields.io/badge/KodeKloud-Passed-brightgreen) ![Topic](https://img.shields.io/badge/Topic-Access%20Control-blue)

## Overview

Configured fine-grained permissions for two existing Jenkins users on the existing **`Packages`** job using **project-based matrix authorization**, while retaining the job's parent ACL inheritance. The KodeKloud Day 76 challenge was **successfully passed**.

This lab demonstrates job-scoped authorization, least-privilege access, ACL inheritance, and verification of saved Jenkins security configuration.

## Task requirements

- Operate on the existing Jenkins job **`Packages`**; do not create a replacement job.
- Use the existing users **`sam`** and **`rohan`**; do not recreate them.
- Enable **project-based security** on `Packages`.
- Set **Inheritance Strategy** to **`Inherit permissions from parent ACL`**.
- Grant the specified permissions without changing unrelated Jenkins jobs or settings.

### Explicit job-level permission matrix

| Permission category | Permission | `sam` | `rohan` |
|---|---|:---:|:---:|
| Job | Build | ✅ | ✅ |
| Job | Cancel | — | ✅ |
| Job | Configure | ✅ | ✅ |
| Job | Read | ✅ | ✅ |
| Run | Update | — | ✅ |
| SCM | Tag | — | ✅ |

**All other explicit permissions were left unchecked** for both users. The permissions above are the *job-specific grants*; effective permissions can additionally include permissions inherited from the parent ACL.

## Architecture and authorization flow

```text
Jenkins global / parent ACL
          |
          | Inherit permissions from parent ACL
          v
    Packages (existing job)
          |
          +-- sam
          |     Job: Build, Configure, Read
          |
          +-- rohan
                Job: Build, Cancel, Configure, Read
                Run: Update
                SCM: Tag
```

> **Important:** Matrix authorization is additive. A job-level matrix adds grants and does not revoke broader inherited grants. For strict effective least privilege, inspect parent/group permissions as well.

## Implementation

1. Signed in to Jenkins as the authorized administrator.
2. Opened **Dashboard → Packages → Configure → General**.
3. Enabled **Enable project-based security**.
4. Selected **Inherit permissions from parent ACL**.
5. Added `sam` and `rohan` as **users**, rather than groups.
6. Granted `sam` **Job/Build**, **Job/Configure**, **Job/Read**.
7. Granted `rohan` **Job/Build**, **Job/Cancel**, **Job/Configure**, **Job/Read**, **Run/Update**, and **SCM/Tag**.
8. Left other checkboxes unchecked and did not change unrelated job configuration.
9. Clicked **Save**, reopened **Packages → Configure**, and verified that the settings persisted.
10. Submitted the challenge; KodeKloud reported **successful completion**.

Detailed steps are in [Commands.md](Commands.md). Conceptual explanations and troubleshooting are in [Notes.md](Notes.md).

## Verification evidence

The Jenkins configuration screen showed:

```text
Enable project-based security: Enabled
Inheritance Strategy: Inherit permissions from parent ACL

rohan: Job: Build, Cancel, Configure, Read · Run: Update · SCM: Tag
sam:   Job: Build, Configure, Read
Anonymous: No explicit permissions granted
Authenticated Users: No explicit permissions granted
```

The final KodeKloud result displayed **“You're done!”** and confirmed that the challenge results were saved.

> These statements summarize the observed job-level configuration. The empty Anonymous and Authenticated Users rows in the job matrix do not, by themselves, prove the absence of inherited permissions.

## Safeguards

- No users or jobs were recreated.
- No unrelated Jenkins job settings were intentionally modified.
- No Jenkins passwords, private keys, or session tokens are stored in this repository.
- Administrator access and existing parent ACL inheritance were retained.
- The configuration was verified after saving rather than relying only on checked boxes in an unsaved form.

## Concepts practiced

**Jenkins security realm vs authorization strategy · Project-based matrix authorization · ACL inheritance · Permission categories · Least privilege · Effective vs explicit permissions · CSRF-safe configuration changes · Configuration verification**

---

**Outcome:** ✅ KodeKloud DevOps Day 76 — Passed.
