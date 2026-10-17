# Day 80 — Jenkins Chained Builds and Apache Service Management

![Jenkins](https://img.shields.io/badge/Jenkins-Chained%20Builds-1D5C91?logo=jenkins&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-Apache%20httpd-333333?logo=linux&logoColor=white) ![Status](https://img.shields.io/badge/KodeKloud-PASSED-218838)

## Overview

Implemented an **upstream/downstream Jenkins build chain** for the Nautilus environment. The upstream job updates an existing website Git checkout on App Server 1. The downstream job restarts the Apache service **only when the upstream build is stable (SUCCESS)**. The solution separates deployment from service administration, and uses noninteractive SSH and narrowly scoped sudo permissions.

**Result:** KodeKloud Day 80 **PASSED** (user-confirmed). The initial downstream build failed with `sudo: a password is required`; the issue was diagnosed and fixed before successful task validation.

## Requirements

| Requirement | Implementation |
|---|---|
| Upstream job | `xfusion-app-deployment` |
| Downstream job | `manage-services` |
| Source | Gitea `sarah/web`, branch `master` |
| Existing deployment checkout | `/var/www/html` on `stapp01` |
| Upstream behavior | Update existing checkout from `origin/master` |
| Downstream behavior | Restart `httpd` on App Server 1 |
| Trigger policy | **Build after other projects are built** → **Stable only** |
| Application | Root URL of the provided load balancer (no `/web` suffix) |

## Architecture

```text
Gitea: sarah/web (master)
          |
          v
 Jenkins: xfusion-app-deployment
          | SSH as sarah@stapp01
          | git pull --ff-only origin master
          v
  /var/www/html (existing Git working tree)
          |
          | only if upstream result = SUCCESS / stable
          v
 Jenkins: manage-services
          | SSH as sarah@stapp01
          | sudo -n systemctl restart httpd
          v
 Apache running -> Load balancer -> Website at /

 FAILED/UNSTABLE upstream -> no downstream restart
```

## Implementation summary

1. Inspected the existing Git repository and the Apache service on App Server 1; did not clone into a nested directory.
2. Established Jenkins-service-account SSH host trust and public-key authentication for `sarah@stapp01`.
3. Created the **Freestyle** job `xfusion-app-deployment` to update the existing checkout safely.
4. Created the **Freestyle** job `manage-services` with a post-upstream **stable-only** build trigger.
5. Configured a remote shell step to restart Apache and verify service health and HTTP response.
6. Diagnosed and corrected noninteractive sudo authorization for the narrowly required Apache commands.
7. Validated the Jenkins job chain and application; the challenge was marked **PASSED**.

## Actual troubleshooting evidence

First downstream build log excerpt:

```text
Started by upstream project "xfusion-app-deployment" build number 3
Host: stapp01
User: sarah
Restarting httpd...
sudo: a password is required
Build step 'Execute shell' marked build as failure
Finished: FAILURE
```

This showed the **job dependency worked** and SSH authenticated correctly. The missing capability was permission for the remote user to execute `systemctl restart httpd` without an interactive sudo prompt. The fix was a tightly scoped sudoers rule, **not** a Jenkins trigger change or a general passwordless-root grant.

## Verification criteria

- Upstream console finishes `SUCCESS` after updating `master`.
- `manage-services` starts automatically and its build cause identifies `xfusion-app-deployment`.
- Downstream console finishes `SUCCESS` after `httpd` restart and HTTP smoke test.
- Apache is active and the load balancer serves the website at the root path.
- Failed or unstable upstream builds do **not** trigger service management.

> **Evidence note:** The passed result is confirmed. The visible failure log is included above; no unprovided downstream success build number or console output is invented.

## Security and operating notes

- Never put SSH passwords, Gitea credentials, or private keys into a Jenkins shell step or public repository.
- Use verified host keys (`StrictHostKeyChecking=yes`).
- Grant only the specific sudo commands necessary; preserve existing policy and validate with `visudo`.
- Production environments should use isolated Jenkins agents, managed credentials, reviewed privileged automation, and rollback/monitoring procedures.

---
**Learning outcome:** Implemented a success-gated deployment → restart job chain and troubleshot the boundary between SSH authentication and privileged service management.
