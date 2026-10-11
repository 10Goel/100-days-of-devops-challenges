# Day 70 — Commands and UI Runbook

> **Task type:** Jenkins web UI security configuration. No shell commands are required to complete the challenge. The commands below are **optional diagnostic tools**, not mandatory lab actions.
>
> **Never paste secrets into shell history, README files, screenshots, or Git commits.**

## A. Mandatory Jenkins UI Steps (in order)

| Step | UI location | Action | Verification |
|---|---|---|---|
| 1 | Jenkins login | Sign in as administrator | Dashboard visible |
| 2 | Manage Jenkins → Users → Create User | Create `yousuf` / `Yousuf`; enter task-provided password exactly | User exists; login test succeeds |
| 3 | Manage Jenkins → Plugins → Installed plugins | Check `Matrix Authorization Strategy` | Plugin enabled |
| 4 | Available plugins (if needed) | Install `Matrix Authorization Strategy` (`matrix-auth`) | Installation successful |
| 5 | Download progress (if needed) | Restart when installation completes and no builds are running | Login screen reappears |
| 6 | Manage Jenkins → Security | Select **Project-based Matrix Authorization Strategy** | Project-based strategy selected |
| 7 | Global permission matrix | `admin`: Overall/Administer; `yousuf`: Overall/Read only; Anonymous: none | Reopen after Save to check |
| 8 | Dashboard → Helloworld → Configure | Enable project-based security | Job permission matrix visible |
| 9 | Job permission matrix | `yousuf`: **Job/Item → Read** only | No additional boxes checked |
| 10 | Helloworld → Configure (reopen) | Check saved values again | Matrix persisted |
| 11 | Separate user session | Log in as `yousuf` and check access | Can view job, cannot build/configure |
| 12 | KodeKloud task panel | Click **Check** | Successful validation |

### Exact permission summary

```text
GLOBAL:
  admin      Overall/Administer = ALLOW
  yousuf     Overall/Read       = ALLOW
  Anonymous  all permissions    = DENY / UNCHECKED

JOB: Helloworld
  yousuf     Job/Read           = ALLOW
  yousuf     all other job, SCM, and agent permissions = UNCHECKED
```

**Note:** In Jenkins' matrix UI, absence of a grant generally means no permission from that particular matrix. Permissions from another applicable global, group, or inherited entry may still apply.

## B. Optional Diagnostics (only if needed)

### Verify Jenkins is available from the server

```bash
# On the Jenkins server, if terminal access is authorized:
sudo service jenkins status
curl -I http://localhost:8080/login
```

A reachable endpoint does not independently prove that authorization settings are correct.

### Inspect Jenkins logs

```bash
sudo journalctl -u jenkins --no-pager -n 100
```

Some Jenkins environments also write an application log:

```bash
sudo tail -n 100 /var/log/jenkins/jenkins.log
```

The log file may not exist. Inspect the journal first when systemd is used.

### Check plugin installation location (read-only)

```bash
sudo ls -l /var/lib/jenkins/plugins/matrix-auth.*
```

The absence of this file on a custom deployment does not definitively mean the plugin is missing; the UI's **Installed plugins** screen is the authoritative user-facing check.

## C. HTTP 403: Invalid/Missing Crumb

**Observed during an earlier attempt:**

```text
HTTP ERROR 403
No valid crumb was included in the request
URI: /job/Helloworld/configSubmit
```

**Safe response:**

1. Do not assume the form saved.
2. Return to Jenkins through the lab's published Jenkins URL.
3. Reauthenticate if needed.
4. Reopen **Helloworld → Configure** as a new page; do not resubmit an old form.
5. Check job-level permissions and save again.
6. Reopen the job configuration and confirm persistence.
7. If repeated, inspect reverse-proxy URL, browser session consistency, and Jenkins logs; avoid disabling CSRF protection.

## D. Common Mistakes and Fixes

| Symptom | Likely issue | Corrective action |
|---|---|---|
| User exists but grader says password incorrect | Wrong case, stale lab credentials, or unverified password | Copy current attempt's password and test login |
| User cannot see `Helloworld` | Job/Read permission not granted/saved | Grant and verify Item/Read for the job |
| User can build or configure job | Broad permissions inherited globally/from a group | Remove unintended higher-scope grants |
| Jenkins administrator locked out | Overall/Administer removed from all admins | Prevent by checking admin grant before every Save |
| Project matrix absent on job | Wrong authorization strategy or plugin missing | Use Project-based Matrix Authorization Strategy |
| Config Save returns 403 crumb error | Expired/mismatched CSRF/session token | Fresh login and fresh Configure page; retain CSRF protection |

## E. Completion Evidence

The final Day 70 KodeKloud result showed **“You're done!”** and confirmed that the challenge was completed successfully.
