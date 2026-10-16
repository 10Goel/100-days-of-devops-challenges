# Day 77 — Jenkins Declarative Pipeline: Static Website Deployment

![Jenkins](https://img.shields.io/badge/Jenkins-Pipeline-2670A8?logo=jenkins&logoColor=white) ![Git](https://img.shields.io/badge/SCM-Git-F05032?logo=git&logoColor=white) ![Status](https://img.shields.io/badge/KodeKloud-Passed-success)

> **100 Days of DevOps | Day 77** — Deploy an existing Gitea-backed static website checkout from Jenkins to Apache on App Server 1, validate the local HTTP service, and confirm the site is accessible through the load balancer.

## Challenge requirements

| Item | Required value |
|---|---|
| Jenkins item | `xfusion-webapp-job` |
| Job type | **Pipeline** (not Freestyle or Multibranch) |
| Agent name | `App Server 1` |
| Agent label | `stapp01` |
| Agent remote root | `/home/sarah/jenkins_agent` |
| SCM | Gitea repository `web_app` |
| Existing checkout / Apache web root | `/var/www/html` on App Server 1 |
| Apache listener | `8080` |
| Required stage | `Deploy` (case-sensitive) |
| Public verification | Load balancer **root** URL `/`, not `/web_app/` |

## Architecture

```text
Developer (sarah) -> Gitea (web_app)
                        |
                        | existing Git checkout / configured upstream
                        v
Jenkins Controller -- SSH --> App Server 1 (label stapp01)
   |                            | /home/sarah/jenkins_agent (agent workspace)
   |                            | /var/www/html (existing repository + web root)
   +-- xfusion-webapp-job       | git pull --ff-only
          stage('Deploy') ------+ HTTP check localhost:8080
                                            |
                                            v
                                   Apache :8080 -> Load Balancer -> / 
```

**Separation of concerns:** The Jenkins agent's root is *not* the website root. Jenkins Remoting and temporary workspaces live under `/home/sarah/jenkins_agent`; the website stays in the existing Git checkout at `/var/www/html`.

## Implementation summary

1. Inspected the App Server 1 user, Java runtime, existing Git checkout, upstream branch, filesystem access and Apache HTTP endpoint.
2. Prepared SSH agent prerequisites and used the verified Java executable for the SSH Build Agents launcher.
3. Created/verified the permanent agent **`App Server 1`** with label `stapp01` and remote root `/home/sarah/jenkins_agent`; confirmed it was **Online**.
4. Created **`xfusion-webapp-job` as a Pipeline item**. An initial Freestyle selection was corrected by recreating the job with the required type.
5. Configured a Declarative **`Deploy`** stage that runs on the `stapp01` agent, updates the pre-existing checkout without discarding local changes, and verifies the Apache endpoint.
6. Ran the pipeline and checked Jenkins Console Output and the website via the lab's **App** link.

## Jenkins Pipeline used

> The following captures the successful workflow. It assumes `/var/www/html` already contains the correctly configured repository, the agent account can update it, and the appropriate Gitea authentication has already been established. See [Commands.md](Commands.md) for prerequisites and cautions.

```groovy
pipeline {
    agent { label 'stapp01' }

    options { skipDefaultCheckout(true) }

    stages {
        stage('Deploy') {
            steps {
                sh '''#!/bin/bash
set -euo pipefail

DEPLOY_DIR="/var/www/html"
test -d "$DEPLOY_DIR/.git"
cd "$DEPLOY_DIR"

echo "Deploying on $(hostname)"
git remote -v
git branch --show-current

if [ -n "$(git status --porcelain)" ]; then
    echo "ERROR: Local modifications exist; refusing to overwrite them" >&2
    exit 1
fi

git pull --ff-only
echo "Deployment finished."
curl --fail --silent --show-error http://localhost:8080/ -o /dev/null
echo "Apache HTTP check passed."
'''
            }
        }
    }

    post {
        success { echo 'Deployment completed successfully.' }
        failure { echo 'Deployment failed. Inspect Console Output.' }
    }
}
```

### Why these choices?

- `agent { label 'stapp01' }` schedules execution on the correct remote host.
- `skipDefaultCheckout(true)` prevents unnecessary checkout into Jenkins's workspace; this task deliberately updates the **pre-existing** repository under `/var/www/html`.
- `stage('Deploy')` satisfies the exact task stage name.
- `git status --porcelain` stops the job when local changes might be overwritten.
- `git pull --ff-only` avoids introducing a merge commit, but requires a configured upstream and an unblocked fast-forward; it does **not** fetch a specific revision unless that is what the current branch tracks.
- `curl --fail` checks HTTP success from the app server; the load balancer is separately validated in the browser.

## Observed execution and validation

The actual Jenkins build (#1) ended with:

```text
master
+ git status --porcelain
+ git pull --ff-only
Already up to date.
Deployment finished.
Apache HTTP check passed.
Deployment completed successfully.
Finished: SUCCESS
```

The user also confirmed that the application loaded correctly through the **App** button and that **KodeKloud marked Day 77 as passed**.

> `Already up to date.` means this particular build found no newer upstream commits to integrate. It confirms the update step worked, **not** that a new commit was deployed during that run.

## Important troubleshooting: Freestyle vs Pipeline

At first `xfusion-webapp-job` had been created as a **Freestyle** job, so the configuration page did not offer the **Pipeline → Definition → Pipeline script** editor. The incorrect new/unbuilt job was recreated as a **Pipeline** item (with the Pipeline plugins installed). A Freestyle job should not be mistaken for a Jenkinsfile-backed Pipeline, and no existing application checkout was deleted in this correction.

## Security and operational considerations

- Store SSH/Gitea secrets in Jenkins Credentials; do not hardcode passwords or private keys in scripts or GitHub.
- Keep host-key verification enabled, and use a compatible agent Java runtime (Java 21 is appropriate for this lab controller).
- Do not use `git reset --hard`, `git clean -fdx`, broad `chown -R`, or an unconditional directory replacement on an active checkout.
- `git pull` updates to the configured upstream tip: for audited production deployments, consider pinned commits, immutable artifacts, explicit approvals, and rollback.
- The HTTP smoke test is not a substitute for application-level testing or an external load-balancer health check.

## Further study

Continue with [Notes.md](Notes.md) to learn Declarative vs Scripted Pipelines, controllers and agents, pipeline execution, credentials, checkout patterns, parallelism, post actions, and deployment safety.
