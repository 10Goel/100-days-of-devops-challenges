# Notes.md — Jenkins Pipelines: Fundamentals to Intermediate

> Day 77 centers on **Pipeline as Code**: model delivery as reviewable stages rather than a series of unrelated manual shell commands.

## 1. CI, CD, and deployment

- **Continuous Integration (CI):** Frequently integrate code and run automated build/test checks.
- **Continuous Delivery:** Maintain a releasable artifact; deployment to production may require approval.
- **Continuous Deployment:** Every passing change is automatically deployed to production.
- **Deployment pipeline:** A coordinated sequence that moves a verified revision into a runtime environment.

**Day 77** implements a small **deployment pipeline**, not a full CI system: it updates an existing static-site checkout and checks Apache's HTTP response. There is no build, unit-test, or artifact-promotion stage in the task.

## 2. Jenkins Controller / Agent / Executor

```text
Controller: schedules Pipeline & persists job/build metadata
    |
    | label stapp01
    v
Agent: App Server 1 (SSH Build Agent + Java Remoting)
    |
    +-- Executor (1) -> workspace under /home/sarah/jenkins_agent
    |
    +-- Deploy step works on separate /var/www/html Git checkout
```

- **Node:** Machine registered with Jenkins.
- **Agent:** Runtime process on a node executing build steps.
- **Label:** Matching token used for scheduling jobs onto eligible agents.
- **Executor:** Concurrent build slot on a node.
- **Remote root:** Agent runtime/workspace storage location, **not** automatically the web server document root.
- **Remoting:** Jenkins communications layer between controller and agent.
- **SSH Build Agents:** Launches agent Java process via SSH; the agent then communicates over its Remoting channel.

### Why `agent { label 'stapp01' }` matters

With `agent any`, Jenkins may pick a different host where `/var/www/html` and Apache do not exist. Label-based scheduling directs the `Deploy` shell step to the correct application server.

## 3. Pipeline vs Freestyle vs Multibranch

| Feature | Freestyle | Pipeline | Multibranch Pipeline |
|---|---|---|---|
| UI-managed build steps | Primary | Optional script editor | Usually Jenkinsfile per branch |
| Versionable logic | Limited | Strong using Jenkinsfile in SCM | Strong, automatically detects branches |
| Stages / post / parallel | Not native as a Pipeline | Supported | Supported |
| Day 77 requirement | **No** | **Yes** | **No** |

An existing Freestyle job does not expose the Pipeline definition editor. Installing Pipeline plugins doesn't convert its item type automatically.

## 4. Declarative Pipeline anatomy

```groovy
pipeline {
    agent { label 'stapp01' }
    options { skipDefaultCheckout(true) }
    environment { SITE_DIR = '/var/www/html' }
    stages {
        stage('Deploy') {
            steps {
                sh 'echo "Target node: $NODE_NAME"'
            }
        }
    }
    post {
        always { echo 'Pipeline finished running' }
    }
}
```

- `pipeline {}`: Declarative entry point.
- `agent`: Node/workspace allocation strategy.
- `options`: Run behavior, e.g. skipping default SCM checkout.
- `environment`: Variables available to steps.
- `stages`: Ordered delivery phases.
- `stage('Deploy')`: User-visible, case-sensitive phase name.
- `steps`: Operations within a stage.
- `sh`: Executes shell on a Unix/Linux agent.
- `post`: Actions after execution, e.g. result reporting.

### Stages versus steps

A **stage** expresses a purpose: *Build*, *Test*, *Deploy*. A **step** is an operation such as `sh`, `echo`, `checkout`, or `archiveArtifacts`. Stages help with observability, timing and failure diagnosis.

## 5. Declarative vs Scripted Pipeline

**Declarative** uses the structured `pipeline { agent ... stages ... }` grammar; it is often easier to standardize and enforce. **Scripted** uses Groovy control flow, with primitives such as `node('stapp01') { stage('Deploy') { sh '...' } }` and is more flexible but may be harder to review.

```groovy
// Scripted example (illustrative, not the task's submitted definition)
node('stapp01') {
    stage('Deploy') {
        sh 'hostname'
    }
}
```

## 6. Jenkinsfile, Pipeline script, Pipeline script from SCM

- **Pipeline script:** Groovy entered into the Jenkins UI; used for this small lab.
- **Pipeline script from SCM:** Jenkins retrieves a `Jenkinsfile` from a repository; recommended for traceable production pipelines.
- **Jenkinsfile:** Text file typically committed alongside application code.
- **SCM checkout:** A plugin-managed repository clone into the Jenkins workspace. It is distinct from the application's pre-existing checkout.

### Why skipDefaultCheckout?

Some Declarative Pipeline setups automatically check out the configured SCM at the start of the job. Day 77 instead uses `/var/www/html`, an existing checkout on App Server 1. `skipDefaultCheckout(true)` makes that separation explicit; it does not disable the explicit `git pull --ff-only` in the Deploy shell.

## 7. Shell exit codes and Jenkins result

```bash
set -euo pipefail
```

- `-e`: Exit on many unhandled nonzero statuses (important Bash exceptions apply).
- `-u`: Fail on unset shell variables.
- `pipefail`: A pipeline fails when one of its constituent commands fails (not just its last command).
- A failing `sh` command normally fails the Jenkins step/stage, then `post { failure { ... } }` runs.
- Jenkins `Finished: SUCCESS` verifies the defined steps returned success, **not** that every user-facing behavior or new source revision is correct.

## 8. Git deployment semantics

The successful workflow was:

```bash
cd /var/www/html
git status --porcelain
git pull --ff-only
```

- `git status --porcelain`: Stable format for machine parsing. Empty output signals a clean tracked/untracked working tree under typical status configuration; ignored files are not shown.
- `git pull --ff-only`: Fetch + fast-forward-only integration for the configured upstream; it refuses diverged histories that need a merge commit.
- `Already up to date.`: No newer upstream commit was integrated in that execution.
- **Avoid** `git reset --hard`, unreviewed `git clean`, and overwriting the web root: these can destroy developer work.

**Operational caution:** Even with clean status, updating a live checkout is a basic deployment model. Large production systems typically deploy an immutable artifact to a release directory, run health checks and atomically switch a symlink or traffic.

## 9. HTTP smoke checks and load balancers

```bash
curl --fail --silent --show-error http://localhost:8080/ -o /dev/null
```

This verifies that Apache on the agent returns a non-error HTTP response, **not** that every page is correct. The lab's separate **App** button checks the externally routed load balancer root `/`. The public route must not be `/web_app/`.

For stronger verification, compare a unique expected content marker, evaluate redirects, or check the actual deployed Git commit. Do not guess at backend health from a single HTTP 200 response.

## 10. Credentials and security

- Use Jenkins **Credentials** (scoped to the appropriate folder/item when practical) for SSH keys, passwords, and tokens.
- Treat secret text as sensitive; avoid logging it, embedding it in a Jenkinsfile, or checking it into Git.
- Keep SSH known-host verification enabled; `ssh-keyscan` alone is not proof of identity.
- Use dedicated least-privilege deploy credentials rather than unrestricted `sudo`.
- Avoid `chmod 777`, indiscriminate ownership changes and `StrictHostKeyChecking=no` workarounds.
- For real environments, consider branch protections, code review, audit logging, deployment approvals and rollback.

### Jenkins credentials example (learning only)

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                withCredentials([string(credentialsId: 'example-token', variable: 'API_TOKEN')]) {
                    sh '''#!/bin/bash
set +x
# Pass secret to an approved command without printing it
command -v true >/dev/null
'''
                }
            }
        }
    }
}
```

`withCredentials` supplies a secret temporarily; credential masking is a defense-in-depth aid, not permission to print secret values.

## 11. Intermediate Pipeline features (not required by Day 77)

### Timeouts, concurrency and logs

```groovy
options {
    timeout(time: 10, unit: 'MINUTES')
    disableConcurrentBuilds()
    timestamps()
}
```

A timeout prevents indefinitely stalled builds; disabling concurrency avoids overlapping deployments to one live checkout; timestamps make diagnosis easier. Availability depends on the installed Jenkins features/plugins.

### Parameters and conditional execution

```groovy
parameters {
    choice(name: 'TARGET_ENV', choices: ['Development', 'Staging', 'Production'])
}
stages {
    stage('Approve deployment') {
        when { expression { params.TARGET_ENV == 'Production' } }
        steps { echo 'A production approval gate would go here' }
    }
}
```

The above illustrates `parameters` and `when`; it does not automatically create an approval gate.

### Parallel checks

```groovy
stage('Checks') {
    parallel {
        stage('Lint') { steps { sh 'echo lint' } }
        stage('Tests') { steps { sh 'echo tests' } }
    }
}
```

Parallel branches can reduce CI latency when tests are independent and agent resources are sufficient.

### Artifacts and test results

`archiveArtifacts artifacts: 'dist/**', fingerprint: true` retains selected build outputs. `junit 'reports/*.xml'` can publish test results generated by test tooling. These were not needed for Day 77's existing-checkout deployment.

### Shared Libraries

Reusable company-wide stages/steps can be packaged into versioned Jenkins Shared Libraries, reducing duplicate Pipeline logic across projects. This is useful once multiple jobs share the same tested deployment pattern.

## 12. Pipeline versus shell script

A Jenkinsfile **orchestrates** where, when and why work happens: agents, stages, credentials, timeout and success/failure semantics. Shell commands perform underlying OS tasks such as Git updates and HTTP checks. A successful deployment normally needs both.

## 13. Troubleshooting quick reference

| Symptom | Diagnosis |
|---|---|
| No Pipeline configuration | Wrong item type, or Pipeline plugin missing |
| Job queued waiting for agent | Label mismatch or node offline / no executors |
| SSH connects, Remoting fails | Java incompatibility or incorrect Java Path |
| `No such file or directory` Java binary | Use an actual verified Java executable path |
| Git ownership error | Reconcile filesystem identity and permitted Git access |
| Local modifications prevent deploy | Review and preserve developer changes |
| `git pull --ff-only` failure | Branch diverged, upstream missing, or authentication problem |
| Local Apache works but LB site fails | Validate load balancer routing and expected root path |
| SUCCESS but stale site | Confirm commit, branch, rendered files and cache behavior |

## 14. Interview-style questions and answers

1. **What is Pipeline as Code?** Delivery logic represented as reviewable code, usually a Jenkinsfile committed to SCM.
2. **Why use labels?** To route jobs to nodes with the required software/filesystem capabilities.
3. **Does `agent { label 'stapp01' }` mean SSH always runs for each step?** No. The SSH launcher starts/maintains the agent process; steps run through Jenkins Remoting on that agent.
4. **What is the difference between a Pipeline stage and a shell command?** Stage is a named orchestration unit; shell is one execution mechanism within it.
5. **Why can `git pull --ff-only` be safer than ordinary pull?** It refuses implicit merge commits on diverged branches.
6. **What does `skipDefaultCheckout(true)` do?** Skips the Declarative default SCM checkout; explicit Git commands still execute.
7. **What does a green build prove?** Configured steps completed successfully, not necessarily that new code deployed or that all routes work.
8. **Why avoid storing credentials in Jenkinsfiles?** Source control and logs can expose them; use scoped Jenkins Credentials.
9. **How do you prevent two deployments interfering?** Disable concurrent builds or use a controlled locking/deployment mechanism.
10. **What's missing for production readiness?** Explicit artifact/revision selection, full tests, approvals where needed, secure credentials, rollbacks, observability and reliable health checks.

## 15. Outcome and next practice

**Day 77 passed.** The `xfusion-webapp-job` Pipeline executed its required **`Deploy`** stage on the intended app-server agent. The log showed `Already up to date.`, `Apache HTTP check passed.`, and `Finished: SUCCESS`; the load balancer site was also verified through the lab App link.

**Suggested next exercise:** Move this UI-defined Pipeline into a source-controlled `Jenkinsfile` with an explicit Test stage, a chosen commit SHA, and a rollback-safe release directory. Keep that separate from this lab's exact graded requirements.
