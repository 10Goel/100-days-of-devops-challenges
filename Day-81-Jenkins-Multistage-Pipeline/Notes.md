# Day 81 — Jenkins CI/CD Notes: Agents, Deploy/Test Stages, Health Checks and Troubleshooting

## 1. Jenkins Architecture: Controller vs Agent

The **controller** manages scheduling, job configuration, credentials, and build history. An **agent** provides an execution environment for build steps.

```text
Jenkins controller
  ├─ schedules deploy-job
  ├─ selects label stapp01
  └─ SSH-launches App Server 1
        └─ /home/sarah/jenkins_agent  (remoting/workspace)
             ├─ Deploy: git pull under /var/www/html
             └─ Test: curl http://stlb01:8091/
```

The agent **name** (`App Server 1`), **label** (`stapp01`), **host** (`stapp01`) and **remote root directory** (`/home/sarah/jenkins_agent`) are independent configuration values.

- **Node name**: Human-readable identifier in Jenkins UI.
- **Label**: Used by `agent { label 'stapp01' }` for executor selection.
- **SSH host**: Machine to connect to.
- **Remote root**: Where Jenkins remoting and build workspaces are stored; it is *not* Apache's document root.
- **Apache document root**: `/var/www/html`, a separate directory storing the live website and the pre-existing Git checkout.

### Why `agent any` is not equivalent

`agent any` can schedule on any eligible node, including the controller or unrelated agents. In this challenge the task explicitly requires execution on App Server 1. The label selector binds the job to the intended execution environment.

## 2. SSH Build Agents: Authentication vs Host Trust

Two independent questions must be answered:

1. **Do I trust that this really is stapp01?** `known_hosts` validates the SSH server's host key.
2. **Can Jenkins log into Sarah's Linux account?** A Jenkins credential handles authentication (password or private key).

Correct host-key verification should be maintained. The identity of the jump-host user `thor` does not confer SSH trust on the Jenkins service account. The relevant known-hosts path is typically `/var/lib/jenkins/.ssh/known_hosts`.

### SSH launcher startup sequence

```text
Controller -> TCP/22 -> stapp01
    -> verify server host key
    -> authenticate as sarah
    -> SFTP transfer remoting.jar
    -> launch compatible Java runtime
    -> establish remoting channel
    -> agent ONLINE
```

Read the actual agent log before diagnosing connection failures. A log stating `Authentication successful` and `SSH host key matches` means those checks have already passed.

## 3. Java Compatibility and Remote Root

`remoting.jar` requires a minimum Java class-file version. For example, a launcher compiled for Java 17 (class file version 61) cannot run on Java 11 (class file version 55). An `UnsupportedClassVersionError` reflects runtime compatibility, not a bad SSH key.

Useful commands:

```bash
java -version
readlink -f "$(command -v java)"
ls -ld /home/sarah/jenkins_agent
```

Never blindly paste an absolute Java path from a different lab server. Validate the exact executable on the destination host.

## 4. Declarative Pipeline: Basic Building Blocks

```groovy
pipeline {
    agent { label 'stapp01' }
    stages {
        stage('Deploy') { steps { sh '...' } }
        stage('Test')   { steps { sh '...' } }
    }
    post {
        success { echo 'SUCCESS' }
        failure { echo 'FAILURE' }
    }
}
```

- `pipeline`: Declarative Pipeline entry point.
- `agent`: Selects an execution location for the build.
- `stages`: Ordered group of work.
- `stage`: Logical phase such as deployment, testing or packaging.
- `steps`: Actual commands/plugins run for that phase.
- `sh`: Executes a shell script on the allocated Unix agent.
- `post`: Runs conditional cleanup, notifications or summary logic based on results.

Exact stage capitalization matters for this task: `Deploy` and `Test`.

## 5. Stage Dependency, Exit Codes and Failure Propagation

The stages execute sequentially. `Test` starts only after `Deploy` completes successfully. The `sh` step normally marks the stage failed when a command exits nonzero.

| Deploy | Test | Overall build |
|---|---|---|
| Success | HTTP success | SUCCESS |
| Failure | Skipped | FAILURE |
| Success | Connection failure / HTTP error | FAILURE |

`set -eu` ensures the shell stops on a failing command and errors on unset variables. It works with typical POSIX `/bin/sh`. `set -euo pipefail` is **not** portable to every `/bin/sh`, and previously caused a Jenkins error in this exercise series. Only use `pipefail` under an explicitly selected compatible shell.

## 6. Git Deployment into an Existing Working Tree

This task starts with a Git clone already serving from `/var/www/html`.

```bash
cd /var/www/html
GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master
```

- `git pull` is broadly fetch plus integrate.
- `--ff-only` refuses history-diverging merges rather than creating a merge commit.
- `origin` is the remote name, not the branch.
- `master` is the requested source branch.
- `GIT_TERMINAL_PROMPT=0` prevents CI from blocking on interactive Git credential requests.

**Subtlety:** Editing and committing `index.html` *inside the already-deployed checkout* can make the new content visible before the Jenkins job runs. That is a consequence of the lab architecture, not proof that Jenkins deployed a fresh commit. For production CI/CD, normally build artifacts in an isolated workspace and publish them atomically to a separate serving directory.

### Production caution

An existing checkout may contain local modifications. Do not indiscriminately call `git reset --hard` or `git clean -fd` in a deployment script: those commands can destroy work. Investigate any dirty state before deciding on a recovery method.

## 7. HTTP Health Checks vs Fixed-Content Assertions

The challenge says the `Test` stage must determine whether the website is **working and accessible**, and explicitly permits a `curl` check to the load balancer.

```bash
curl --fail --silent --show-error \
     --connect-timeout 5 \
     --max-time 20 \
     http://stlb01:8091/ \
     -o /dev/null
```

Flag breakdown:

| Flag | Meaning |
|---|---|
| `--fail` | Returns nonzero on HTTP 4xx/5xx |
| `--silent` | Suppresses the progress meter |
| `--show-error` | Still prints useful errors |
| `--connect-timeout 5` | Limits the connection phase |
| `--max-time 20` | Limits the whole request |
| `-o /dev/null` | Discards response body, tests response status |

### The failure that mattered in Day 81

Early iterations added a fixed-content check inside `Test`:

```bash
curl -fsS http://stlb01:8091/ -o response.html
grep -Fq 'Welcome to xFusionCorp Industries' response.html
```

The user's normal Jenkins build passed, but KodeKloud twice reported `Build failed: FAILURE`. This is consistent with a validator that changes the site's content to test deployment. The revised Pipeline removed the fixed-string `grep` from `Test` while keeping a strict HTTP failure check. The user subsequently confirmed **Day 81 PASSED**.

**Principle:** Separate **liveness** (site responds successfully), **readiness** (can serve requests), and **content assertions** (specific content is correct). Content assertions are appropriate when content is part of a stable test contract; they can be inappropriate when an external evaluator intentionally substitutes content.

The task's required welcome message was still verified outside the Jenkins `Test` stage, via browser and `curl | grep`.

## 8. Why Test the Load Balancer Instead of Only localhost?

The paths verify different failure domains:

```text
Local test:
  stapp01 -> localhost:8080 -> Apache

End-to-end test:
  stapp01 -> stlb01:8091 -> load balancer -> backend Apache
```

`localhost` can work even if the load balancer is misconfigured or unable to reach the application server. A real deployment acceptance check should verify the externally expected entry point. The task also explicitly requires serving the site at `/`, not `/web`.

## 9. Common Failures and Targeted Troubleshooting

| Symptom | First diagnostic |
|---|---|
| Agent Offline | SSH agent `Log`; identify earliest genuine error |
| No known hosts | `ssh-keygen -F stapp01` as Linux user `jenkins` |
| SSH authentication failed | Validate Jenkins credentials for Linux user `sarah` |
| `UnsupportedClassVersionError` | Compare required vs installed Java runtime |
| Java executable not found | Check remote `command -v java` and configured Java Path |
| Job waits in queue | Check agent label `stapp01`, Online state, executors |
| `git pull` fails | Check branch, remote, permissions, network and Git auth |
| Test receives HTTP error | `curl -v http://stlb01:8091/` (avoid secrets in output) |
| Website works locally but not via LB | Inspect load balancer routing, backend and ports |
| Normal build SUCCESS, evaluator FAIL | Compare what the evaluator changes or rechecks; avoid fragile assumptions |

Do not solve SSH issues by disabling host verification, and do not solve permission problems using indiscriminate `chmod 777` or global passwordless sudo.

## 10. Interview Questions (with concise answers)

**Q1. Why use an SSH agent instead of executing everything on the Jenkins controller?**  
To isolate workloads, target the machine with the necessary tools/files, and avoid running arbitrary build code on the controller.

**Q2. What happens to Test when Deploy fails?**  
With sequential Declarative stages, the failed deployment stops normal progression; Test is skipped and the Pipeline fails.

**Q3. What is the difference between node name and label?**  
The name identifies a configured node; the label is a scheduling selector used by Pipeline `agent` directives.

**Q4. Why use `git pull --ff-only`?**  
It accepts only a fast-forward update and refuses to create an unintended merge commit during automated deployments.

**Q5. What makes `curl --fail` useful for CI?**  
It returns an error exit status for HTTP 4xx/5xx, allowing Jenkins to mark the test failed instead of blindly logging success.

**Q6. Can a successful `curl localhost` guarantee the public website works?**  
No. It bypasses load balancer, routing and other infrastructure elements used by clients.

**Q7. Why did hardcoded welcome-text testing cause problems here?**  
The page was subject to validator-controlled content changes; an availability test should not assume immutable content unless explicitly required.

**Q8. What did we actually validate as the final outcome?**  
The user confirmed KodeKloud accepted Day 81 after the corrected two-stage Pipeline was used. We do not invent the final build number or unseen private checker logs.

## 11. Further Improvements for Real CI/CD Environments

- Use Jenkins-managed private-key credentials or an ephemeral agent rather than storing static Linux passwords.
- Separate source checkout from the live Apache document root and deploy a versioned artifact atomically.
- Use CI tests for syntax, content, and security before deployment; use readiness/liveness checks after deployment.
- Record deployment commit SHA, artifacts, and environment metadata for traceability.
- Implement rollback to a previous release and monitoring after publication.
- Add authorization controls and a least-privilege deploy account.

---

**Outcome:** **Day 81 PASSED** — deployed code using a dedicated Jenkins SSH agent and a two-stage Declarative Pipeline with HTTP testing through the load balancer.
