# Day 79 — Automated Website Deployment with Jenkins and Gitea Webhooks

![Jenkins](https://img.shields.io/badge/Jenkins-Pipeline-blue?logo=jenkins) ![Gitea](https://img.shields.io/badge/Gitea-Webhook-green?logo=gitea) ![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen)

> **100 Days of DevOps · Day 79** | Event-driven CI/CD for a static website hosted by Apache on the Nautilus application server.

## Overview

Configured a Jenkins Declarative Pipeline to deploy a website automatically when a developer pushes to the `master` branch of Gitea repository `sarah/web`. Jenkins receives a push webhook, connects to App Server 1 via SSH, updates the existing Git checkout, synchronizes all website files into Apache's document root, and verifies the HTTP endpoint.

**Result:** KodeKloud validation **PASSED**. The final webhook-triggered build completed successfully, and the deployed homepage returned the required welcome message.

## Requirements and environment

| Component | Verified task configuration |
| --- | --- |
| Jenkins job | `xfusion-app-deployment` (Pipeline) |
| Git service | Gitea; repository `sarah/web` |
| Source checkout | `/home/sarah/web` on `stapp01` |
| Tracked deployment branch | `master` |
| Target | `/var/www/html` on `stapp01` |
| Deployment user | `sarah` |
| Web server | Apache `httpd` on port `8080` |
| Event source | Gitea `Push Events` webhook |
| Jenkins plugin | Generic Webhook Trigger |
| Required homepage text | `Welcome to the xFusionCorp Industries` |
| Public endpoint | Load balancer root URL (`/`) |

## Architecture

```text
Developer (sarah)                Gitea (sarah/web)
/home/sarah/web                   origin/master
    | git commit && git push            |
    +---------------------------------->|
                                       | Push event (JSON)
                                       v
                             Gitea repository webhook
                                       | HTTP POST
                                       v
                      Jenkins Generic Webhook Trigger
                       JSONPath $.ref; master-only filter
                                       |
                                       v
                       xfusion-app-deployment Pipeline
                           Jenkins built-in executor
                                       | SSH (key + verified host key)
                                       v
                               sarah@stapp01
                       git pull --ff-only origin master
                                       | rsync (exclude .git)
                                       v
                            /var/www/html (Apache)
                                       | http://localhost:8080/
                                       v
                            Load balancer main URL
```

## Implementation highlights

1. **Validated the application host.** Inspected the repository, verified Apache on port `8080`, ensured `rsync`, Git and curl were available, and set document-root ownership to `sarah` as required by the lab.
2. **Established noninteractive SSH.** From the Jenkins Linux service account, verified `stapp01`'s host key, installed an SSH public key for Sarah, and successfully tested `BatchMode=yes` authentication.
3. **Verified Jenkins scheduling.** The Built-In Node was online with two executors and no custom labels. Using `agent any` avoided non-existent labels used in prior attempts.
4. **Created a Pipeline job.** Set the exact name `xfusion-app-deployment`, used a single `Deploy` stage, and configured safe, noninteractive remote execution.
5. **Configured a push webhook.** Extracted `ref` from the JSON body with `$.ref`, filtered to `^refs/heads/master$`, and configured Gitea to send push events for `master` to Jenkins's Generic Webhook Trigger endpoint.
6. **Tested the full workflow.** Edited `/home/sarah/web/index.html`, committed and pushed to `origin/master`; the webhook automatically initiated the successful Jenkins deployment.

## Deployment Pipeline

The actual workflow is documented in full in [Commands.md](Commands.md). Its key behavior is:

```groovy
pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }
    stages {
        stage('Deploy') {
            steps {
                sh '''
                    set -eu
                    ssh -o BatchMode=yes -o StrictHostKeyChecking=yes sarah@stapp01 'bash -se' <<'REMOTE'
set -euo pipefail
SOURCE=/home/sarah/web
DEST=/var/www/html
cd "$SOURCE"
test -d .git
test -w "$DEST"
if [ -n "$(git status --porcelain)" ]; then
    echo 'ERROR: Source checkout has uncommitted changes' >&2
    exit 1
fi
git switch master
GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master
rsync -av --exclude='.git/' "$SOURCE/" "$DEST/"
test -f "$DEST/index.html"
curl -fsS http://localhost:8080/ -o /dev/null
echo "Deployed commit: $(git rev-parse --short HEAD)"
echo 'SUCCESS: Website deployment completed.'
REMOTE
                '''
            }
        }
    }
    post {
        success { echo 'Deployment completed successfully.' }
        failure { echo 'Deployment failed. Review Console Output.' }
    }
}
```

> **Scope:** This is the lab's working deployment pattern, not a universal production deployment design. `rsync` copies files recursively but deliberately omits Git metadata and does not delete old destination-only files. Repository access must be noninteractive. Production systems should use Jenkins-managed credentials and preferably a dedicated build agent rather than the controller.

## Evidence of success

The verified Git push created commit `b6b6262` with message **Update homepage welcome message** and updated `origin/master`. The webhook-triggered Jenkins console showed:

```text
Generic Cause
Running on Jenkins .../workspace/xfusion-app-deployment
Remote host: stapp01
Remote user: sarah
Already up to date.
Deploying repository files...
index.html
Deployed commit: b6b6262
SUCCESS: Website deployment completed.
Finished: SUCCESS
```

The Apache response contained:

```text
Welcome to the xFusionCorp Industries
```

**KodeKloud final validator:** **PASSED** (completion confirmed in the challenge result screen).

## Troubleshooting lessons

| Symptom from earlier attempts | Root cause | Correction |
| --- | --- | --- |
| `Invalid option type "timestamps"` | Missing/incompatible Declarative Pipeline option | Omit `timestamps()` |
| `doesn't have label 'built-in'` / `'jenkins'` | Guessed labels not assigned to node | Inspect nodes; use `agent any` in the verified single-node lab |
| `Illegal option -o pipefail` | Jenkins `sh` ran under POSIX shell, not Bash | Use portable `set -eu` on Jenkins; explicitly invoke `bash -se` remotely |
| Build never starts | No eligible executor for requested label | Verify Built-In Node online, executors > 0 |
| Webhook request but no deployment | Incorrect target/token/ref filter or unreachable endpoint | Inspect Gitea delivery logs, `$.ref` and exact `master` regexp |

More detail: [Notes.md](Notes.md).

## Security & operational considerations

- Do not publish Jenkins admin passwords, Gitea credentials, webhook tokens, or SSH private keys.
- Verify SSH host fingerprints; do **not** disable strict host-key checking.
- Prefer Jenkins Credentials and dedicated deployment credentials for real workloads.
- Avoid `git reset --hard`, `git clean`, or `rsync --delete` without explicit approval and a backup plan.
- Disable overlapping deployments with `disableConcurrentBuilds()`; filter triggers to `master`.
- HTTP `200` alone confirms service availability, not exact page content: also inspect the rendered response and the deployed commit.

## Outcome

**Day 79: PASS** — implemented an end-to-end, push-driven CI/CD workflow from Gitea to Jenkins to Apache, with automated branch update, complete website file synchronization, and a deployed-content verification step.
