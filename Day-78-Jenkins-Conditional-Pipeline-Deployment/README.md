# Day 78 — Jenkins Conditional Pipeline Deployment

![Jenkins](https://img.shields.io/badge/Jenkins-Declarative_Pipeline-blue) ![Git](https://img.shields.io/badge/Git-Branch_Deployment-orange) ![Status](https://img.shields.io/badge/KodeKloud-Passed-success)

## Objective
Create a **Jenkins Pipeline** named `datacenter-webapp-job` to deploy either the `master` or `feature` branch of the Gitea `web_app` repository, determined at build time by a **String parameter** named `BRANCH`. The existing website checkout is `/var/www/html` on App Server 1. Serve it via Apache (port `8080`) and the load balancer root URL.

## Requirements
| Item | Required configuration |
|---|---|
| Job | `datacenter-webapp-job` (Pipeline, not Multibranch) |
| Agent | `App Server 1`, label `stapp01` |
| Remote root | `/home/sarah/jenkins_agent` |
| Repository | Gitea `web_app`, existing checkout `/var/www/html` |
| Parameter | String `BRANCH` (`master` or `feature`) |
| Required stage | `Deploy` (case-sensitive) |
| Web server | Apache on port `8080` |
| Verification | Website at load balancer `/`, successful builds for both branches |

## Architecture
```text
Developer / Gitea: web_app (master, feature)
                    |
                    v
Jenkins: datacenter-webapp-job
  BRANCH=master or BRANCH=feature
                    |
                    v
App Server 1 SSH agent [stapp01]
  /home/sarah/jenkins_agent  <-- Jenkins workspace
                    |
                    v
Deploy stage updates existing checkout
  /var/www/html                <-- Apache document root
                    |
                    v
Apache :8080 -> Load Balancer -> https://<LB-URL>/
```

## Implementation overview
1. Prepared the SSH agent on `stapp01` using `/home/sarah/jenkins_agent`, Java 21 and trusted SSH host keys.
2. Created a regular Jenkins **Pipeline** job (not a Freestyle or Multibranch job).
3. Configured the `BRANCH` string parameter and a Declarative Pipeline containing the single deployment stage `Deploy`.
4. Validated the requested branch, fetched remote refs, switched the existing checkout to the selected branch and fast-forwarded it.
5. Confirmed HTTP responsiveness on local Apache port `8080` and checked the load-balanced website.

> **Safety:** The existing checkout in `/var/www/html` was retained. Do not `git clone` into `/var/www/html/web_app`, as the site must be served at `/`. The deployment should fail on uncommitted changes or divergent history instead of deleting developer work.

## Pipeline
See [Commands.md](Commands.md#jenkins-pipeline-script) for the complete reusable Jenkinsfile and [Notes.md](Notes.md) for its behavior and limitations.

## Verified results
- **Master deployment:** User verified a successful build with `BRANCH=master`.
- **Feature deployment:** Jenkins build **#2** completed with `Finished: SUCCESS`.
- The build log showed `git branch --show-current` returning `feature` and `git log -1 --oneline` returning **`e93dfcf Added feature.html file`**.
- Apache localhost health check on port `8080` succeeded; KodeKloud reported **Day 78 Passed**.

```text
Deployment Verification
feature
e93dfcf Added feature.html file
SUCCESS: feature deployment completed
Conditional deployment completed successfully.
Finished: SUCCESS
```

## Troubleshooting learned
| Symptom | Explanation / resolution |
|---|---|
| Agent cannot launch: missing `known_hosts` | Add and verify the SSH host key under `/var/lib/jenkins/.ssh/known_hosts` for the Jenkins Linux user |
| SSH works but Java agent fails | Verify the agent's actual Java 21 executable path and remote directory |
| `git switch` cannot find branch | Fetch and inspect `origin/master` / `origin/feature` |
| Working tree has modifications | Stop instead of discarding changes; coordinate with the repository owner |
| Git credentials or permissions fail | Validate filesystem ownership and authorized access to Gitea; do not embed secrets in code |
| Successful Pipeline but incorrect page | Check active branch, commit, Apache DocumentRoot, and load balancer root URL |

## Learning outcomes
Parameterized CI/CD, Declarative Pipelines, controller–agent architecture, Git remote tracking and fast-forward deployment, safe handling of working trees, and post-deployment HTTP checks.

**Security:** Lab passwords, private keys, and operational secrets are intentionally excluded.
