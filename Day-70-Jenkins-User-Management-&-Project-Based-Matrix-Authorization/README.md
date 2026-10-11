# Day 70 — Jenkins User Management & Project-Based Matrix Authorization

![Status](https://img.shields.io/badge/Challenge-Passed-2ea44f) ![Jenkins](https://img.shields.io/badge/Jenkins-Security-2671be) ![Topic](https://img.shields.io/badge/Topic-Least%20Privilege-555)

## Overview

The Nautilus DevOps team needed to onboard a developer to Jenkins with **read-only access** to an existing job while preserving full administrator control. This challenge demonstrates Jenkins authentication, global authorization, job-scoped permissions, and practical verification of least-privilege access.

**Outcome:** KodeKloud confirmed **Day 70 successfully completed** on the final attempt.

> **Security:** Never commit Jenkins login passwords, API tokens, session cookies, or CSRF crumbs to GitHub. The lab credentials are intentionally omitted here.

## Requirements

| Scope | Identity | Requirement |
|---|---|---|
| User database | `yousuf` | Create the user with the task-specified password and full name `Yousuf` |
| Global authorization | `admin` | Retain **Overall → Administer** |
| Global authorization | `yousuf` | Grant **Overall → Read** only |
| Global authorization | `Anonymous` | Remove all permissions |
| Job `Helloworld` | `yousuf` | Grant **Job/Item → Read** only |
| Jenkins | All | Use **Project-based Matrix Authorization Strategy** |

## Architecture and Permission Scope

```text
Jenkins authentication (user database)
   ├── admin   → privileged administrator
   └── yousuf  → authenticated developer
                       |
Jenkins authorization (Project-based Matrix)
   ├── Global matrix
   |    ├── admin     → Overall/Administer
   |    ├── yousuf    → Overall/Read
   |    └── Anonymous → no permissions
   |
   └── Helloworld job-level matrix
        └── yousuf → Job/Read only
                    (no Build / Configure / Delete / Workspace)
```

**Important:** Job-level permissions supplement inherited/global permissions. A job-level matrix cannot revoke broader permissions already inherited from global roles or groups.

## Implementation

### 1. Create the developer account

1. Sign in to Jenkins with the administrator account.
2. Navigate to **Manage Jenkins → Users → Create User**.
3. Create user `yousuf` with full name `Yousuf` and the exact password specified in the current KodeKloud task.
4. Verify the account appears under **Users**.
5. Validate the user can **authenticate** in a separate browser session before changing authorization settings. An authorization-denied page after login is different from a failed password.

### 2. Ensure Matrix Authorization Strategy is installed

1. Open **Manage Jenkins → Plugins → Installed plugins**.
2. Find **Matrix Authorization Strategy** (`matrix-auth`).
3. If missing, install it from **Available plugins**.
4. If Jenkins asks for a restart, wait until plugin installation finishes and no jobs are running. Restart and wait for the login screen to return.

### 3. Configure global permissions

1. Open **Manage Jenkins → Security**.
2. Choose **Project-based Matrix Authorization Strategy**.
3. Configure the global matrix:

| Identity | Overall/Administer | Overall/Read | Other grants |
|---|:---:|:---:|---|
| `admin` | ✅ | Included by admin privilege | No additional checkboxes necessary |
| `yousuf` | ❌ | ✅ | **None** |
| `Anonymous` | ❌ | ❌ | **None** |

4. Check any `authenticated` or other group entries for accidental broad permissions.
5. Before saving, **confirm `admin` retains Overall/Administer** to prevent administrator lockout.
6. Save, reopen Security, and verify the matrix persisted.

### 4. Configure `Helloworld` job permissions

1. Open **Dashboard → Helloworld → Configure**.
2. Find and enable **project-based security** in the job configuration.
3. Add `yousuf` to the project's permission matrix.
4. Check **only Job/Item → Read** for that identity. Leave all Build, Configure, Delete, Workspace, Discover, SCM, Agent, and other permissions unchecked.
5. Save. **Reopen Configure** and verify the checkbox and user row persisted.

### 5. Confirm effective permissions

Use a different browser/private session to sign in as `yousuf`:

| Action | Expected |
|---|---|
| Sign in | Allowed |
| View Jenkins dashboard | Allowed |
| Open `Helloworld` | Allowed |
| Run/build `Helloworld` | Denied |
| Configure/delete `Helloworld` | Denied |
| Manage Jenkins security | Denied |

Finally submit through the KodeKloud **Check** button only after confirming both configuration pages saved correctly.

## Challenges Encountered and Lessons Learned

### Attempt 1 — Job read permission not detected

The validator reported that the user did not have read-only access on `Helloworld`. A prior attempt to save the job configuration had shown **HTTP 403: No valid crumb was included in the request**. That indicates Jenkins rejected the configuration submission, so the job-specific permission may not have been persisted.

**Resolution/workflow improvement:** Open a fresh authenticated Jenkins configuration page, save, then reopen the job and verify the permission matrix. Do **not** disable CSRF protection.

### Attempt 2 — Account/password validation failed

The validator reported that the user was not created or the password was incorrect. The Users page alone proves an account appears; it does not prove that the password matches the grader's expected value.

**Resolution/workflow improvement:** On a fresh retry, use the newly assigned (potentially different) username/password and test authentication early. Avoid reproducing credentials from a previous lab attempt.

### Final attempt — Passed

For the final attempt, the lab specified user `yousuf` and full name `Yousuf`. The completion screen confirmed the challenge passed. This repository documents the successful configuration pattern and key verification checks; it does not claim access to backend grader internals.

## Security Principles Demonstrated

- **Authentication vs authorization:** Identity verification is distinct from permissions.
- **Least privilege:** Grant only Overall/Read globally and Job/Read on the target job.
- **Permission scope:** Global grants can apply across jobs, while project-level grants target an individual job.
- **Administrative safety:** Preserve an administrator with Overall/Administer.
- **Secure change validation:** Test the effective privileges and verify settings persisted after Save.
- **CSRF protection:** Jenkins uses request crumbs to protect state-changing requests; do not bypass this control to fix a failed save.

## Result

**✅ Day 70 — Passed (confirmed by the KodeKloud completion screen).**
