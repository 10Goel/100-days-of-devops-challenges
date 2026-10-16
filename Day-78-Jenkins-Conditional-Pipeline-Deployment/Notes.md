# Day 78 — Jenkins Pipeline & Git Branching Study Notes

## 1. The objective and why it matters
A deployment pipeline should accept a well-defined release target rather than blindly deploy whichever branch happened to be checked out. Day 78 chooses between `master` and `feature` at runtime using a Jenkins String parameter named `BRANCH` and deploys the selected branch to `/var/www/html` on the `stapp01` agent.

## 2. Pipeline essentials
- **Controller:** stores configuration, schedules Pipeline execution, dispatches work.
- **Agent:** executes the steps in an actual workspace on its assigned machine.
- **Executor:** a slot for running a build on a node. One executor means one simultaneous executor task.
- **Node label:** scheduler selector; `agent { label 'stapp01' }` schedules on a matching, online node.
- **Pipeline job:** a flow of code-defined steps. A **Multibranch Pipeline** discovers branch-specific pipelines automatically, which this challenge explicitly forbids.
- **Declarative Pipeline:** structured `pipeline { agent ... stages { stage ... } }` DSL.
- **Scripted Pipeline:** Groovy control flow using `node`, `stage`, and steps; flexible but more imperative.
- **Jenkinsfile:** text representation of Pipeline-as-Code, often committed to SCM. This lab uses an inline **Pipeline script**.

## 3. Key blocks in the Day 78 Jenkinsfile
```groovy
pipeline {
  agent { label 'stapp01' }
  parameters { string(name: 'BRANCH', defaultValue: 'master') }
  options { skipDefaultCheckout(true); disableConcurrentBuilds() }
  stages {
    stage('Deploy') {
      steps {
        sh 'echo "Deploying $BRANCH"'
      }
    }
  }
  post { success { echo 'Deployment succeeded' } }
}
```
- `agent`: chooses **where** work executes.
- `parameters`: defines **build-time inputs**. Jenkins exposes them as `params.BRANCH` in Groovy and usually `$BRANCH` inside the shell environment.
- `stage('Deploy')`: a named phase shown in the Pipeline UI. The case-sensitive task expects exactly `Deploy`.
- `steps`: Jenkins operations such as `sh`, `echo`, `checkout`, `archiveArtifacts`.
- `sh`: executes shell commands on the **agent**, not implicitly on the controller.
- `skipDefaultCheckout(true)`: avoids automatic SCM checkout; helpful because this lab uses an *existing* repo at `/var/www/html` instead of the workspace.
- `disableConcurrentBuilds()`: prevents two builds of this job from racing to change the same working tree.
- `post`: emits status-dependent actions once the pipeline concludes.

### Why both parameter definition and validation?
A String parameter accepts arbitrary text; restrict it explicitly to `master` and `feature`. This reduces accidental deployments and prevents command injection when parameter values enter shell commands.

## 4. Git branch movement, step by step
Suppose the remote has:
```text
origin/master   A--B--C
origin/feature  A--B--C--D--E
```

If `/var/www/html` currently uses `master` and `BRANCH=feature`:
1. `git fetch origin` updates the local **remote-tracking refs** but does not switch branches or update the working tree.
2. `git switch feature` switches to the local branch (or `git switch --track -c feature origin/feature` creates a local tracking branch).
3. `git merge --ff-only origin/feature` moves the local branch forward to the remote-tracking commit only if a fast-forward is possible.
4. The working tree reflects `feature` and Apache can serve its contents directly from `/var/www/html`.

For `BRANCH=master`, the same steps target `origin/master`.

### Fetch vs pull vs checkout vs switch
| Command | Purpose | Working tree effect |
|---|---|---|
| `git fetch origin` | Retrieve refs and objects | Does not switch branches |
| `git pull` | Fetch then merge/rebase depending on config | Updates current branch, not necessarily requested branch |
| `git switch feature` | Change active branch | Updates checked-out files |
| `git merge --ff-only origin/feature` | Fast-forward active branch if possible | Updates files when moving forward |
| `git status --porcelain` | Machine-readable changes | Read-only |

**Important:** `--ff-only` will fail on diverged history, which is preferable to silently producing merge commits or forcing changes away. A clean working tree check protects uncommitted changes; it does not guarantee the deployment is authorized or the remote tracking setup is correct.

## 5. Workspace vs deployed document root
```text
/home/sarah/jenkins_agent/  = Jenkins agent working area
/var/www/html/             = Existing Gitea checkout AND Apache document root
```
Do **not** put an additional `web_app/` directory under Apache's document root. The website must respond at load balancer `/`, not `/web_app/`.

Git checkout into a web-facing directory is lab-friendly but has production drawbacks: transient partial updates, `.git` exposure risks, lack of rollback, and mixing source control with serving. Production deployments commonly build immutable artifacts and deploy them to versioned release directories with atomic symlink switches and protected web configuration.

## 6. Agent launch troubleshooting
Earlier Jenkins challenges demonstrated three distinct failure classes:

**Host verification failure:**
```text
No Known Hosts file was found at /var/lib/jenkins/.ssh/known_hosts
```
The SSH launcher using Known Hosts verification needs an authentic, readable host-key entry for `stapp01`. Connect as the Jenkins Linux service user, verify the host key against trusted details, accept it, then check `ssh-keygen -F stapp01`.

**Java bytecode mismatch:**
```text
UnsupportedClassVersionError ... class file version 61 ... recognizes up to 55
```
Class file version 61 = Java 17; 55 = Java 11. The agent JVM must support the Remoting JAR bytecode. Verify Java 21 directly and configure **Java Path** if default `java` points to Java 11.

**Nonexistent Java executable:**
```text
... /usr/lib/jvm/.../bin/java: No such file or directory
Exit code=127
```
Versioned Java package paths can differ between fresh labs. Find the current executable, test it, and then set Jenkins Java Path. Avoid blindly copying a path from an earlier challenge.

An agent is ready only when it reports **Online** and can run steps on the intended remote host.

## 7. Deployment verification layers
1. **Jenkins:** `Finished: SUCCESS` confirms the Pipeline's steps returned success.
2. **Git:** `git branch --show-current` and `git log -1 --oneline` confirm selected branch and commit.
3. **Apache:** `curl -fsS http://localhost:8080/ -o /dev/null` confirms an HTTP response with a successful status.
4. **Load balancer:** Visit the root URL using the lab's App button.
5. **Content-specific verification:** Confirm a known branch-specific change, e.g., the feature branch's `feature.html` file, is available as intended. This distinguishes serving *some* page from deploying the *right* code.

Observed build #2: active branch `feature`, commit `e93dfcf Added feature.html file`, HTTP check passed, `Finished: SUCCESS`; user verified master separately and KodeKloud marked Day 78 **Passed**.

## 8. Common failure scenarios
| Symptom | Diagnose | Safe action |
|---|---|---|
| Job stuck waiting for node | Agent Offline / label mismatch | Inspect Nodes view and agent log |
| `Permission denied` updating repo | Agent user lacks rights | Check ownership, ACL, approved deployment account |
| `fatal: not a git repository` | Wrong working directory | Confirm `.git` at `/var/www/html` |
| `fatal: invalid reference` | Branch absent | `git fetch origin`, `git branch -a` |
| Fast-forward refused | Local and remote diverged | Inspect graph and coordinate; do not force-reset |
| Website returns 200 but appears stale | Wrong branch / cache / DocumentRoot | Check commit and branch-specific content |
| SSH launch fails after authentication | Java or remote root issue | Use exact launch log and Java executable path |

## 9. Production upgrades (beyond lab scope)
- Restrict deployment identities and SSH credentials via Jenkins Credentials.
- Protect Git remote and document-root access; never embed passwords or keys in GitHub.
- Test and build immutable artifacts instead of deploying a mutable Git checkout.
- Add static validation, unit tests, smoke tests, and a rollback strategy.
- Serialize deployments to a shared target and configure automated approvals for protected environments.
- Add timeouts, audit logs, notifications, and artifact retention.
- Secure Apache so `.git` and other private content cannot be served.

## 10. Interview / self-check questions
1. What is the difference between an agent, node, executor, and label?
2. Why is `Deploy` a stage, while `sh` is a step?
3. Why did this task require a Pipeline rather than a Multibranch Pipeline?
4. How does Jenkins expose the `BRANCH` input to Groovy and Bash?
5. Why does `git fetch` not deploy code by itself?
6. What does `git merge --ff-only` protect against?
7. Why might a clean working tree still be unsafe to deploy?
8. How does `skipDefaultCheckout(true)` differ from removing SCM integration altogether?
9. What happens if two builds try to change `/var/www/html` concurrently?
10. Why do HTTP 200 and `Finished: SUCCESS` not prove the intended branch is live?

**Outcome:** Day 78 passed; mastered parameter-driven branch selection in a Jenkins Declarative Pipeline, with safe Git updates, remote-agent execution, and multi-layer deployment verification.
