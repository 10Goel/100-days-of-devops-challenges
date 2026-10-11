# 📘 Day 68 — Jenkins Notes: Fundamentals to Intermediate

> **Learning objective:** Understand not just how to install Jenkins, but *why* it is used in DevOps, how a Jenkins controller runs jobs, how Pipelines are structured, and the essential security and troubleshooting practices.

## Table of Contents

1. [CI, CD and Jenkins](#1-ci-cd-and-jenkins)
2. [Jenkins architecture](#2-jenkins-architecture)
3. [Controller, agents, executors and nodes](#3-controller-agents-executors-and-nodes)
4. [Installation components](#4-installation-components)
5. [Initial setup and security](#5-initial-setup-and-security)
6. [Jenkins jobs and builds](#6-jenkins-jobs-and-builds)
7. [Freestyle versus Pipeline](#7-freestyle-versus-pipeline)
8. [Pipeline fundamentals and Jenkinsfile](#8-pipeline-fundamentals-and-jenkinsfile)
9. [Stages, steps, agents and post conditions](#9-stages-steps-agents-and-post-conditions)
10. [Artifacts, reports and workspaces](#10-artifacts-reports-and-workspaces)
11. [Triggers, webhooks and SCM](#11-triggers-webhooks-and-scm)
12. [Credentials and secrets](#12-credentials-and-secrets)
13. [Plugins and integrations](#13-plugins-and-integrations)
14. [Common failure diagnosis](#14-common-failure-diagnosis)
15. [Practical CI/CD example](#15-practical-cicd-example)
16. [Intermediate concepts and next steps](#16-intermediate-concepts-and-next-steps)
17. [Interview recap and self-test](#17-interview-recap-and-self-test)

---

## 1. CI, CD and Jenkins

### Continuous Integration (CI)

Developers frequently merge code into a shared repository. CI automatically **checks out**, **builds**, and **tests** changes to catch integration failures early.

```text
Developer commits → Git repository → Jenkins job → Build → Tests → Feedback
```

**Example:** A developer pushes Python code; Jenkins checks syntax, runs `pytest`, and reports pass/fail.

### Continuous Delivery vs Continuous Deployment

| Term | Meaning | Production release |
|---|---|---|
| Continuous Integration | Automatically integrate, build, and test changes | Not necessarily deployed |
| Continuous Delivery | Keep releases deployable; production release can require approval | Usually manually authorized |
| Continuous Deployment | Automatically deploy each change that passes all required checks | No manual production gate |

**Important:** Jenkins is an **automation/orchestration server**. It does not magically include Git, Docker, Kubernetes, cloud credentials, or tests: you configure tools and integrations.

## 2. Jenkins architecture

```text
                    GitHub / GitLab / Git server
                               │
                         webhook / poll
                               ▼
                     ┌─────────────────┐
                     │ Jenkins         │
                     │ Controller      │
                     │ schedules jobs  │
                     │ stores config   │
                     └───────┬─────────┘
                             │ assigns build
           ┌─────────────────┼─────────────────┐
           ▼                 ▼                 ▼
      Linux Agent       Windows Agent      Docker/K8s Agent
      compile/test      platform tests     container build
           │                 │                 │
           └─────────────────┼─────────────────┘
                             ▼
                    logs / artifacts / status
```

The **controller** schedules and coordinates jobs, serves the UI, stores Jenkins configuration, and manages agents. **Agents** perform workloads. For production usage, prefer running builds on agents, not the controller, to reduce risks and contention.

## 3. Controller, agents, executors and nodes

| Term | Meaning | Example |
|---|---|---|
| Controller | Manages Jenkins and schedules work | `jenkins.example.com` |
| Node | Machine registered to Jenkins | VM called `linux-build-01` |
| Agent | Process on a node that communicates with controller | Agent running over SSH/inbound protocol |
| Executor | Capacity slot for executing a build | Node configured with `2` executors |
| Workspace | Directory used by a specific job/build for checked-out files | Agent work directory |
| Label | Name/tag used for agent selection | `linux`, `docker` |

**Numerical example:** Two nodes, each with two executors, can execute up to four compatible build slots simultaneously, assuming there are no other job/resource restrictions. Executors are *slots*, not CPU cores. More executors do not create more physical hardware.

### Why agents matter

Separating the controller and agents isolates tools, permits scaling, and allows platform-specific builds. Production controllers should normally have **zero executors** and delegate builds to agents.

## 4. Installation components

### Java runtime

Jenkins runs on Java. Confirm a version supported by the Jenkins release. Java 21 is commonly appropriate for contemporary releases, subject to OS package availability and the current support matrix.

### APT package manager

`apt update` retrieves package metadata. `apt install jenkins` downloads a signed Debian package and configures the installed software. `dpkg -l jenkins` checks package status.

### Service management

`service jenkins start` launches the daemon through the system service layer. `service jenkins status` checks whether it started. Systemd-based installations also support journal inspection using `journalctl -u jenkins`, even though the lab mandates `service` specifically for starting.

### Ports and directories

| Item | Common default | Purpose |
|---|---|---|
| `8080/tcp` | Jenkins HTTP | Web UI and API |
| `/var/lib/jenkins` | `JENKINS_HOME` on Debian packages | Configuration, jobs, plugins, secrets, workspaces and data |
| `/var/lib/jenkins/secrets/initialAdminPassword` | Initial setup | One-time initial unlock secret |
| `journalctl -u jenkins` | Service journal on systemd | Startup/runtime logs |

**Do not mistake a running port for successful initial setup:** the browser wizard may still be incomplete.

## 5. Initial setup and security

The first launch typically requires: unlock with initial password → install plugins → create administrator → configure URL → open dashboard.

### Why the initial password is different

The initial password temporarily authorizes **first-run unlocking**. The task's designated admin password is set **afterward** while creating `theadmin`.

### Security fundamentals

- Apply **least privilege**: use admin accounts for administration, scoped roles for developers.
- Restrict controller and web UI exposure; use HTTPS behind a properly configured reverse proxy where appropriate.
- Patch Jenkins core and plugins regularly with testing and change control.
- Keep secrets in Jenkins Credentials (or a managed external secret system), not in `Jenkinsfile`, SCM, shell history, or screenshots.
- Back up and protect `JENKINS_HOME`, with attention to sensitive secret and key material; verify restores.
- Avoid executing untrusted repository code with privileged credentials.

## 6. Jenkins jobs and builds

- **Job:** Definition of an automation task (e.g., build a Java app).
- **Build:** One execution of a job, typically numbered (`#1`, `#2`, etc.).
- **Queue:** Jobs waiting for available executor/resources.
- **Console Output:** Logs of a job's execution.
- **Build result:** Common states are `SUCCESS`, `FAILURE`, `UNSTABLE`, `ABORTED`, and `NOT_BUILT`.

Example:

```text
Job: api-ci
  Build #21: SUCCESS
  Build #22: FAILURE (unit test failed)
  Build #23: SUCCESS (developer fixes test)
```

**`UNSTABLE`** often signifies that the build completed but quality checks, such as some test results, did not meet expectations.

## 7. Freestyle versus Pipeline

| Freestyle job | Pipeline job |
|---|---|
| Mostly configured via UI | Defined as code (ideally a `Jenkinsfile`) |
| Quick for small/simple tasks | Best for multi-stage CI/CD workflows |
| Harder to review changes | Version-controlled, reviewable workflow |
| Limited workflow modeling | Stages, parallelism, conditions, approvals, post actions |

**Recommendation:** Learn Freestyle to understand jobs, then adopt **Pipeline as Code** for real projects.

## 8. Pipeline fundamentals and Jenkinsfile

A `Jenkinsfile` is a text file (Groovy-based Pipeline DSL) typically committed at the repository root.

### Declarative Pipeline — starter example

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Building project"'
            }
        }
        stage('Test') {
            steps {
                sh 'echo "Running tests"'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'All stages succeeded.'
        }
        failure {
            echo 'At least one stage failed.'
        }
    }
}
```

> `checkout scm` requires a Pipeline job with an SCM context, such as a Multibranch Pipeline or Pipeline script from SCM. `sh` requires a Unix-like agent. The build/test commands shown are demonstrations, not real builds or tests.

### Declarative vs Scripted

- **Declarative:** Structured `pipeline { ... }`, easier to enforce conventions and reason about; suitable for most teams.
- **Scripted:** General Groovy-oriented `node { ... }` workflow, powerful but requires extra care and expertise.

## 9. Stages, steps, agents and post conditions

| Construct | Purpose |
|---|---|
| `pipeline` | Top-level declarative Pipeline |
| `agent any` | Run on any eligible available executor |
| `agent { label 'linux' }` | Choose agents with a label |
| `stages` | Container holding stages |
| `stage('Test')` | Logical unit visible in UI |
| `steps` | Commands/actions executed in a stage |
| `sh '...'` | Run shell command on Unix-like agent |
| `bat '...'` | Run Windows batch command |
| `environment` | Define environment variables |
| `post` | Actions conditioned on outcome |
| `when` | Conditionally execute a stage |
| `input` | Pause for approval/input, often before deployment |

**Flow:** `Checkout → Build → Test → Package → Deploy`. A failed stage generally prevents later ordinary stages; `post` blocks can still execute.

### Sequential vs parallel tests

```groovy
stage('Test') {
    parallel {
        stage('Unit') {
            steps { sh 'echo Unit tests' }
        }
        stage('Lint') {
            steps { sh 'echo Lint checks' }
        }
    }
}
```

Parallel branches require suitable executor resources. Parallelism is not guaranteed to make jobs faster when agents are saturated.

## 10. Artifacts, reports and workspaces

- **Workspace:** Files available while a build executes (checkout, output, test files).
- **Artifact:** A build output retained for later download or deployment (`.jar`, `.zip`, etc.).
- **Test report:** Structured test results, often JUnit XML, that Jenkins can display.

```groovy
post {
    always {
        junit allowEmptyResults: true, testResults: 'reports/junit/*.xml'
        archiveArtifacts allowEmptyArchive: true, artifacts: 'dist/**'
    }
}
```

This snippet assumes the JUnit/archiving steps are available and the build produces corresponding files. Do not treat `allowEmptyResults: true` as a substitute for actual automated testing; it can hide missing report generation if used carelessly.

**Key distinction:** Artifacts are outputs of a build; a workspace is the build's working area. The workspace may be cleaned while archived artifacts remain.

## 11. Triggers, webhooks and SCM

### Common execution modes

1. **Manual** — user clicks *Build Now*.
2. **Webhook** — Git hosting service notifies Jenkins about a push or pull request.
3. **SCM polling** — Jenkins periodically queries SCM to detect changes.
4. **Scheduled** — cron-like schedule triggers work.
5. **Upstream / downstream** — another Jenkins job initiates a job.

```text
Git push → hosting platform webhook → Jenkins → Checkout → Test → Result
```

**Webhook vs polling:** Webhooks are push notifications (usually lower latency and less wasted polling); polling repeatedly checks for changes. Both require correctly configured repositories, access and plugins.

### Multibranch Pipeline

A Multibranch Pipeline discovers branches/PRs with a Jenkinsfile and can build them independently. Branch trust settings and credential exposure deserve special attention for forked/untrusted pull requests.

## 12. Credentials and secrets

Jenkins can manage credentials such as username/password, secret text, SSH keys, and certificates (depending on plugins).

**Good practice:** Give each automation the minimum credential scope/permissions necessary. Prefer short-lived cloud identities over static long-lived keys when supported.

### Safe example: binding a username/password

```groovy
pipeline {
    agent any
    stages {
        stage('Use Credential') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'example-service-login',
                    usernameVariable: 'APP_USER',
                    passwordVariable: 'APP_PASS'
                )]) {
                    sh '''#!/bin/sh
                      set +x
                      # Call a trusted script that reads APP_USER and APP_PASS
                      # from the environment. Do not print them.
                      echo 'Credentials loaded for trusted tool'
                    '''
                }
            }
        }
    }
}
```

**Important:** Environment variables may still be exposed to processes/users with sufficient access on a shared agent. Use trusted, isolated agents for secret-bearing builds; Jenkins masking is not a comprehensive security boundary.

## 13. Plugins and integrations

Jenkins plugins add support for SCM systems, Pipelines, authorization, credentials, test reporting, notifications, artifact management, and cloud/agent provisioning.

Examples:

| Integration | Use case |
|---|---|
| Git / GitHub integration | Checkout and source change triggers |
| Pipeline | Run Jenkinsfile-based workflows |
| Credentials Binding | Provide credentials to steps |
| JUnit | Publish test results |
| Docker tools / CLI installed on agents | Build and push container images |
| Kubernetes plugin | Provision ephemeral Jenkins agents as pods |

**Nuance:** A Jenkins plugin does not necessarily install the target software. A Docker-related plugin does not automatically provide a Docker daemon or correct permissions on agents.

## 14. Common failure diagnosis

### Jenkins service startup failure

```text
Problem → service jenkins status
        → journalctl -u jenkins -n 100 --no-pager
        → java -version
        → ss -lntp | grep ':8080'
        → inspect memory/disk/repository and permissions
```

### Job fails even though Jenkins is healthy

Controller running ≠ every build will work. Check the specific build's **Console Output**, agent connectivity/labels, tool versions, credentials, repository authorization, test failures, and network reachability.

### Typical status interpretation

- **Queued:** No compatible idle executor, a job constraint is active, or resources are not available.
- **Agent offline:** Controller cannot schedule build there until connectivity is restored.
- **Checkout failed:** SCM URL, branch, credential or network issue.
- **Tests failed:** Application/test problem, not automatically a Jenkins service issue.
- **HTTP 503/UI inaccessible:** Check local service first, then firewall, reverse proxy and lab routing.

## 15. Practical CI/CD example

Imagine a Python API with `requirements.txt` and `tests/`.

```text
Developer pushes change
        │
        ▼
GitHub webhook triggers Jenkins
        │
        ▼
Jenkins checks out repository
        │
        ▼
Installs dependencies in virtualenv
        │
        ▼
Runs unit tests (pytest)
        │
        ├── FAIL → job fails; developer fixes code
        │
        ▼ PASS
Package artifact → optional staging deploy → approval → production
```

Example **illustrative** Linux Pipeline (requires Python 3, venv support, access to dependencies, a compatible agent, and a configured SCM-backed Pipeline job):

```groovy
pipeline {
    agent { label 'linux' }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Prepare') {
            steps {
                sh '''
                  python3 -m venv .venv
                  .venv/bin/python -m pip install -r requirements.txt
                  .venv/bin/python -m pip install pytest
                '''
            }
        }
        stage('Test') {
            steps {
                sh '.venv/bin/python -m pytest -q'
            }
        }
    }
    post {
        always { echo 'Build completed. Review console output.' }
    }
}
```

This Pipeline tests code; it is **not** a production-grade deployment pipeline. Real deployments require release gates, rollback plans, trusted image/artifact storage, and environment-specific authorization.

## 16. Intermediate concepts and next steps

| Concept | Why it matters | Suggested next practice |
|---|---|---|
| Pipeline as Code | Reviewable/repeatable automation | Store a Jenkinsfile in Git |
| Agent labels | Put workloads on compatible nodes | Route Linux builds to `linux` node |
| Parameters | Reuse pipelines for controlled options | Add a `choice` parameter |
| `when` conditions | Avoid inappropriate deploys | Deploy only from protected branch |
| `input` approval | Enforce release gates | Approve staging-to-production |
| Parallel stages | Run independent checks concurrently | Unit + lint branches |
| Shared Libraries | Centralize common Pipeline code | Reusable test/deploy helper |
| Credentials | Authenticate without hardcoded secrets | Scoped SCM registry credentials |
| Multibranch | Branch/PR-aware CI | Detect Jenkinsfiles per branch |
| Jenkins Configuration as Code (JCasC) | Reproducible controller configuration | Configure a disposable controller |
| Backup/restore | Recover configuration and build history | Test backup restoration securely |
| Observability | Diagnose long queues and failures | Track queue, agent and build metrics |

**Learning sequence:** Jenkins setup → first Freestyle job → first Jenkinsfile → Git webhook → real build and tests → agents → credentials → artifact packaging → Docker image CI → staging deploy → approval/production workflows.

## 17. Interview recap and self-test

### Short Q&A

**Q: Is Jenkins a CI tool or CD tool?**  
A: An automation server that can orchestrate both CI and CD when the needed build, test, integration, and deployment capabilities are configured.

**Q: Jenkins controller vs agent?**  
A: Controller manages jobs, state and scheduling; agents execute assigned workloads.

**Q: What is a Jenkinsfile?**  
A: Version-controlled source defining a Jenkins Pipeline.

**Q: Why prefer Pipeline over Freestyle?**  
A: Pipeline supports auditable, reviewable, multi-stage automation as code.

**Q: Where does Jenkins store persistent data?**  
A: In `JENKINS_HOME`, commonly `/var/lib/jenkins` for Debian/Ubuntu packages.

**Q: What is the difference between `stage` and `step`?**  
A: A stage is a named logical phase; steps are the executable operations within it.

**Q: Do Jenkins plugins install Docker/Kubernetes?**  
A: No; plugins offer integrations, while runtimes/tools and permissions still need to be provisioned appropriately.

**Q: Can `service jenkins status` verify admin-account correctness?**  
A: No. It checks service health; inspect user management in the UI for account details.

### Mini exercise

1. Explain why a `SUCCESS` service status and a failing Pipeline are not contradictory.
2. Sketch a 3-stage CI workflow for a React or Python application.
3. Identify where a webhook, secret, executor and artifact fit in that workflow.
4. Explain why an `input` stage before production is continuous **delivery**, not continuous **deployment**.
5. Diagnose an offline Linux agent versus a controller HTTP-port problem.

---

## Official Documentation

- [Installing Jenkins on Linux](https://www.jenkins.io/doc/book/installing/linux/)
- [Jenkins User Handbook](https://www.jenkins.io/doc/book/using/)
- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Jenkins Architecture](https://www.jenkins.io/doc/book/scaling/jenkins-controller/)
- [Jenkins Security](https://www.jenkins.io/doc/book/security/)

**Key takeaway:** Day 68 is about establishing a functioning Jenkins **controller** and securely completing its initial configuration. The natural next step is to execute real, version-controlled CI jobs on suitable **agents**.
