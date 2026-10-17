# Day 79 — Notes: Event-Driven Jenkins CI/CD, Webhooks, SSH, Git and Apache

## 1. What Day 79 teaches

Day 79 moves from **manual deployment** to **event-driven deployment**. A developer changes a Git repository, Gitea emits a webhook, Jenkins starts a Pipeline, the Pipeline connects to the deployment server, pulls `master`, copies the *website tree*, and checks the running Apache site. The challenge passed with this exact flow.

## 2. Jenkins concepts: controller, executor, agent and workspace

- **Controller:** orchestrates Pipeline jobs and stores configuration.
- **Agent:** a machine/process running steps assigned by Jenkins.
- **Executor:** a slot capable of running a job at a time.
- **Workspace:** working directory allocated for a job; it need not be the website's source checkout.
- **`agent any`:** requests an available eligible executor. In this lab the only eligible node was the **Built-In Node**, online with 2 executors and no custom labels.
- **`agent { label 'x' }`:** requests an executor on a node carrying label `x`; the machine's hostname **does not automatically create that label**.
- **`skipDefaultCheckout(true)`:** avoids Jenkins automatically checking out SCM in its workspace, since the pipeline intentionally uses the existing `/home/sarah/web` repository on `stapp01`.
- **`disableConcurrentBuilds()`:** serializes deployments so two pushes do not cause overlapping modifications to the same document root.

**Previous issue:** `agent { label 'built-in' }` and `agent { label 'jenkins' }` stalled because neither label existed. This was a *scheduling* failure, before the Deploy stage.

## 3. Declarative Pipeline structure

```groovy
pipeline {
    agent any
    options { disableConcurrentBuilds() }
    stages {
        stage('Deploy') {
            steps {
                sh 'echo deploy'
            }
        }
    }
    post {
        success { echo 'Passed' }
        failure { echo 'Inspect console' }
    }
}
```

The `stages` block describes *what happens*, `steps` contains *how it happens*, and `post` describes actions after success/failure. Day 79 used one deployment stage; webhook filtering is configured at the job trigger level rather than by adding a second stage.

## 4. Jenkins `sh` versus Bash

Jenkins `sh` often runs scripts using `/bin/sh`. POSIX `sh` supports `set -e` and `set -u`, but on common systems it does not support `set -o pipefail`.

**Failure:**

```text
set: Illegal option -o pipefail
```

**Fix:** use `set -eu` in the controller-side POSIX shell, and explicitly run `bash -se` over SSH on the remote server before calling `set -euo pipefail`.

When using a heredoc, the closing marker must be aligned correctly. A wrong indentation or an unmatched delimiter can lead to a hanging or malformed shell step. Always preserve the exact Jenkinsfile line layout, and read the *first* error in Console Output.

## 5. Webhook versus polling

**Webhook (push based):** Gitea sends an HTTP POST when the repository changes. Jenkins responds to the event, avoiding periodic repository checks.

**Polling:** Jenkins periodically asks the repository whether anything changed. Polling is not equivalent to verifying the required push-event automation.

The Generic Webhook Trigger endpoint is:

```text
/JENKINS-PATH/generic-webhook-trigger/invoke?token=<SECRET>
```

The exact external Jenkins hostname changes between temporary KodeKloud labs. The webhook URL must use the *current* service address and private token and must be reachable from the Gitea server.

**Security:** a URL token is an authentication secret. Do not include one in public documentation, screenshots or debug output. Rotate exposed tokens.

## 6. How the JSONPath master filter works

A typical Gitea push event includes:

```json
{"ref":"refs/heads/master"}
```

The trigger extraction rule `$.ref` assigns the payload value to variable `ref`.

```text
Variable:            ref
JSONPath:            $.ref
Filter text:         $ref
Filter regexp:       ^refs/heads/master$
```

The regex anchors `^` and `$` ensure that the whole ref is precisely `refs/heads/master`. The Gitea-side Branch filter is `master` as an additional safeguard. **Value filter** inside Post content parameters should be left blank: it transforms extracted values and is not the same thing as the trigger's Optional regexp filter.

## 7. Event delivery, trigger and build: separate checkpoints

```text
Git push successfully reaches origin/master
  ↓
Gitea sends POST (inspect recent deliveries)
  ↓
Jenkins endpoint accepts request and token
  ↓
Payload passes branch regexp
  ↓
New Jenkins build appears with webhook cause
  ↓
Deploy stage finishes SUCCESS
  ↓
Homepage returns the required text
```

An HTTP 2xx from the webhook endpoint confirms some level of request acceptance, **not** automatically a successful build. Conversely, a manual Build Now proves the deployment script works but **not** that webhooks function. Day 79's screenshot showed **Generic Cause** and the user observed the build start automatically after the push.

## 8. SSH authentication versus host verification

They answer distinct questions:

- **Host verification:** “Am I talking to the correct `stapp01`?” Stored in `/var/lib/jenkins/.ssh/known_hosts` for the Jenkins execution account.
- **User authentication:** “Am I authorized to log in as `sarah`?” Uses the Jenkins SSH private key and Sarah's `authorized_keys`.

`ssh-copy-id` installs the *public* key, never the private one. `BatchMode=yes` prevents interactive password prompts; `StrictHostKeyChecking=yes` refuses unknown/untrusted hosts. The successful preflight output was `sarah` and `stapp01`, then source repository, writable destination and tool checks succeeded.

For real CI systems, prefer Jenkins Credentials, scoped deploy keys or SSH agents, least privilege and key rotation, rather than relying on a long-lived controller service-account key.

## 9. Git fetch, pull and `--ff-only`

`git pull --ff-only origin master` is roughly `git fetch origin master` followed by a fast-forward integration of fetched commits, if possible. It refuses to create a merge commit when histories have diverged.

```text
Before deploy:
local master: A---B
origin/master: A---B---C

After successful fast-forward pull:
local master: A---B---C
origin/master: A---B---C
```

It does **not** force-reset the checkout. A dirty source checkout is deliberately rejected before deployment, so Jenkins does not silently replace Sarah's ongoing local work. `GIT_TERMINAL_PROMPT=0` avoids unattended jobs hanging for interactive Git credentials.

## 10. Why `rsync` and not `cp index.html`

The task explicitly required deploying the **entire repository**. With:

```bash
rsync -av --exclude='.git/' /home/sarah/web/ /var/www/html/
```

- trailing `/` on the source means *copy the contents* into the destination;
- recursive traversal includes nested asset directories, CSS and JS;
- `.git` remains out of the Apache document root;
- `-a` uses archive mode and `-v` shows transferred files;
- with no `--delete`, destination-only old files are not removed.

**Production caution:** archive mode preserves timestamps/permissions subject to the executing user's capabilities. Validate file ownership, SELinux context, retention and deletions for your environment. If exact mirroring including deletions is required, review dry runs and backup/rollback strategy before using `--delete`.

## 11. Smoke testing and correctness

```bash
curl -fsS http://localhost:8080/
```

- `-f`: fail with nonzero status on HTTP errors;
- `-sS`: quiet progress but show errors.

This verifies service reachability, not necessarily that the correct content deployed. Check exact page text:

```bash
curl -fsS http://localhost:8080/ | grep -F 'Welcome to the xFusionCorp Industries'
```

Compare source and deployed copies using `diff -q` or `rsync -avnc --exclude='.git/'`. Finally confirm the load balancer serves the **root path** `/`, not `/web`.

## 12. Failure table from the Day 79 learning journey

| Failure | Layer | Diagnosis | Fix |
| --- | --- | --- | --- |
| `Invalid option type "timestamps"` | Pipeline compilation | Declarative option unavailable | Remove optional `timestamps()` |
| Stuck: `doesn't have label 'built-in'` | Scheduling | No matching node label | Check Nodes and executors; select verified node |
| Stuck: `doesn't have label 'jenkins'` | Scheduling | Controller name ≠ automatic label | Use verified `agent any` in the single-node lab |
| `Illegal option -o pipefail` | Local shell runtime | `/bin/sh` not Bash | POSIX flags locally, Bash explicitly remotely |
| Host-key failure | SSH verification | Missing `known_hosts` entry | Trust verified host key under Jenkins account |
| SSH prompts for password | SSH authentication | Public key not authorized | Install authorized key; test BatchMode |
| Repo already up to date | Git | No new changes to fetch | Normal for manual dry-run; test real push |
| Webhook runs wrong branch | Trigger | Bad JSONPath or regexp | Use `$.ref`, `^refs/heads/master$` |
| Site loads old content | Deployment | Bad source/dest/branch or cache | Verify HEAD, rsync output, file diff and response |

## 13. Real-world improvement ideas

1. Move the Jenkinsfile into source control after the lab.
2. Use a dedicated low-privilege deployment agent and Jenkins Credentials.
3. Validate build artifacts, lint HTML, and add unit/integration checks before deploying.
4. Use versioned releases and atomic symlink swaps for safer rollbacks.
5. Add notifications for webhook failure and deployment errors.
6. Audit who can push to `master`; enforce branch protection and reviews.
7. Restrict webhook callers with network policy and secrets as supported by your deployment.

## 14. Interview practice

**Q: Does Build Now verify webhook automation?**  No. The webhook must actually trigger a new build after a push.

**Q: Why did `agent { label 'jenkins' }` hang?**  Jenkins couldn't find an executor carrying that label; the controller's name is not automatically a label.

**Q: What is the difference between `known_hosts` and `authorized_keys`?**  The former verifies *server identity*; the latter authorizes a *client public key* for login.

**Q: Why use `--ff-only`?**  It prevents an unattended deployment from manufacturing merge commits in a diverged repository.

**Q: Why shouldn't `.git` be deployed into Apache's document root?**  It contains repository internals and metadata unnecessary for serving the website and potentially sensitive if exposed.

**Q: Why are webhook token, branch filter and Git push three distinct controls?**  The token authenticates the request, the filter scopes events, and the push supplies a real event and new commit for validation.

---

**Verified outcome:** Commit `b6b6262` was pushed to `origin/master`; webhook-triggered Jenkins Build #3 completed successfully; Apache served `Welcome to the xFusionCorp Industries`; KodeKloud marked Day 79 **PASSED**.
