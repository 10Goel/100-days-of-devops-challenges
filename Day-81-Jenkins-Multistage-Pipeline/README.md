# Day 81 — Jenkins Pipeline: Deploy and Test a Static Website

![Jenkins](https://img.shields.io/badge/Jenkins-Pipeline-D24939?logo=jenkins&logoColor=white) ![Git](https://img.shields.io/badge/Git-Gitea-F05032?logo=git&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-CentOS%20Stream-262577?logo=linux&logoColor=white) ![Status](https://img.shields.io/badge/Challenge-PASSED-16803c)

## Overview

Implemented and validated a **two-stage Jenkins Declarative Pipeline** that deploys the `master` branch of a Gitea-hosted static website to an existing Git checkout on App Server 1, then performs an HTTP availability test against the load balancer. Jenkins executes both stages on a dedicated **SSH-launched agent**, not on the controller.

**Challenge result: PASSED** (confirmed through KodeKloud after an earlier validator-related failure was corrected).

## Requirements

| Item | Configuration |
|---|---|
| Jenkins job | `deploy-job` (Pipeline, **not** Multibranch Pipeline) |
| Gitea repository | `sarah/web` |
| Source branch | `master` |
| Existing deployment checkout | `/var/www/html` on `stapp01` |
| Required `index.html` text | `Welcome to xFusionCorp Industries` |
| Agent node name | `App Server 1` |
| Agent label | `stapp01` |
| Agent SSH user and host | `sarah@stapp01` |
| Agent remote root | `/home/sarah/jenkins_agent` |
| Java runtime | Compatible Java, Java 17 available in the challenge |
| Web server | Apache `httpd`, port `8080` |
| Load balancer smoke-test URL | `http://stlb01:8091/` |
| Exact Pipeline stage names | `Deploy`, `Test` |

## Architecture

```text
 Developer (sarah)
       |
       | edit index.html, commit, git push origin master
       v
 Gitea: sarah/web (master)
       |
       | Jenkins deployment performs git pull --ff-only
       v
 Jenkins controller ───SSH launcher───> App Server 1 (stapp01)
                                        Jenkins agent: /home/sarah/jenkins_agent
                                        |
                                 deploy-job (label: stapp01)
                                   |              |
                                   v              v
                              [1] Deploy      [2] Test
                           /var/www/html      curl LB URL
                                   |              |
                                   v              v
                               Apache :8080 -> Load balancer :8091
                                                       |
                                             HTTP health checked
```

The checkout already exists under Apache's document root. The Pipeline **does not create another `/web` subdirectory**.

## Implementation Summary

1. Verified the existing Git checkout on App Server 1, updated `index.html` to contain the required welcome message, committed and pushed the change to Gitea's `master` branch.
2. Confirmed Apache served the site on port `8080` and the load balancer exposed it at the root URL.
3. Prepared `/home/sarah/jenkins_agent`, verified a compatible Java runtime, and established Jenkins SSH trust using the controller account's `known_hosts`.
4. Added Jenkins credentials for the **Linux SSH account** `sarah` (distinct from the Gitea credentials).
5. Created the permanent SSH agent `App Server 1` with label `stapp01`, and verified its log reported **`Agent successfully connected and online`**.
6. Created `deploy-job` as a standard Jenkins Pipeline and configured precisely two stages: `Deploy` then `Test`.
7. Verified the deployment and load-balancer reachability independently of Jenkins, and submitted the completed challenge successfully.

## Final Pipeline

> The final approach checks **HTTP availability** rather than hardcoding the original welcome message inside the `Test` stage. The welcome message is still required in the published page and verified separately. This distinction matters when the challenge validator updates application content during its own deployment test.

```groovy
pipeline {
    agent {
        label 'stapp01'
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    echo "=== DEPLOY STAGE ==="
                    echo "Running on $(hostname)"

                    cd /var/www/html
                    test -d .git
                    test "$(git branch --show-current)" = "master"

                    echo "Pulling latest changes from origin/master"
                    GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master

                    echo "Current deployed commit:"
                    git log -1 --oneline
                    echo "Deploy stage completed successfully."
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -eu

                    echo "=== TEST STAGE ==="
                    echo "Testing website through the load balancer"

                    curl --fail --silent --show-error \
                         --connect-timeout 5 \
                         --max-time 20 \
                         http://stlb01:8091/ \
                         -o /dev/null

                    echo "Website is accessible. Test passed."
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment and website test successful.'
        }
        failure {
            echo 'Pipeline failed during deployment or testing.'
        }
    }
}
```

## Validation and Evidence

The successfully launched SSH agent previously showed:

```text
Launcher: SSHLauncher
Communication Protocol: Standard in/out
This is a Unix agent
Agent successfully connected and online
```

Local and load-balancer checks confirmed the intended homepage message:

```bash
curl -fsS http://localhost:8080/ | grep -F 'Welcome to xFusionCorp Industries'
curl -fsS http://stlb01:8091/ | grep -F 'Welcome to xFusionCorp Industries'
```

The earlier full Pipeline ran both stages and returned `Finished: SUCCESS`, but the KodeKloud grader initially reported `Build failed: FAILURE`. After revising the Pipeline's Test stage to rely on an HTTP health check instead of a fixed-string assertion, **the user confirmed Day 81 passed**. This is the validated task outcome; no final build number or fabricated checker logs are asserted here.

## Troubleshooting: Why an Earlier Successful Jenkins Run Was Not Enough

| Symptom | Diagnostic insight / correction |
|---|---|
| Jenkins displayed `SUCCESS`, but KodeKloud failed validation | A local success does not prove the evaluator's deployment test will succeed. Re-examine hidden assumptions in the health check. |
| Test required the literal original welcome message | The validator may substitute different content for deployment testing. Use `curl --fail` to check availability; validate the challenge's requested welcome message separately. |
| Agent connection or Java launch problems in earlier Jenkins exercises | Verify the SSH host key, Linux credentials, actual Java executable and compatible runtime before launching. |
| Deploy runs on wrong node | Set `agent { label 'stapp01' }`, with the label assigned to `App Server 1`. |
| Git pull waits for credentials | Avoid interactive authentication in CI; configure authorized Git access and use `GIT_TERMINAL_PROMPT=0`. |
| Deploy succeeds but website does not work | Curl the **load balancer root URL** (not just localhost or `/web`). |

## Key Takeaways

- A build pipeline's success is determined by the exit status of its commands, not by `echo` statements alone.
- The `Test` stage is a *quality gate*: if `Deploy` fails, subsequent stages are skipped; if the HTTP check fails, the Pipeline fails.
- Jenkins SSH agents are distinct from SSH commands run directly inside build steps. Node name, node label and remote root serve different purposes.
- A static website's health check should verify **service availability**, while content verification can be a separate test when the expected content is stable and part of the test contract.
- `git pull --ff-only` avoids accidental merge commits during automated deployment, but an existing working tree must still be checked for local modifications and the required file permissions.

> **Security note:** Lab passwords, webhook tokens, and SSH private keys have intentionally been excluded from this documentation. In real environments, prefer Jenkins-managed SSH private-key credentials and dedicated build agents over running arbitrary workloads on the controller.
