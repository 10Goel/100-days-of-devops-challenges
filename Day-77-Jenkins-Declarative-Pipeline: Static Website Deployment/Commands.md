# Commands.md — Day 77 Execution Runbook

> All commands are contextual examples for this lab; verify the real username, permissions, Git upstream and Java path before making changes. **Do not execute shell commands on the wrong host.** This task is mainly configured in the Jenkins UI.

## 1. Pre-deployment checks (jump host -> App Server 1)

```bash
ssh sarah@stapp01
whoami
hostname
java -version
ls -ld /home/sarah /var/www/html
sudo ls -la /var/www/html
```

Check the existing repository with the **same identity that will run the Pipeline**, if it has read access:

```bash
cd /var/www/html
git status --short
git remote -v
git branch -vv
git branch --show-current
```

If permissions block these checks, investigate ownership and safe deployment access. **Do not** globally mark the repository safe or recursively change ownership without determining why. A `git -C` command run with `sudo` may have different credential and ownership behavior than a build run as `sarah`.

Check Apache locally:

```bash
sudo systemctl status httpd --no-pager
curl -I http://localhost:8080/
```

## 2. Prepare the agent directory

Run as the verified SSH agent account on `stapp01`:

```bash
mkdir -p /home/sarah/jenkins_agent
chmod 755 /home/sarah/jenkins_agent
test -w /home/sarah/jenkins_agent && echo 'Agent root writable'
```

If Java is missing on an RPM-based host, and installation is authorized:

```bash
sudo dnf install -y java-21-openjdk-headless
rpm -ql java-21-openjdk-headless | grep '/bin/java$'
```

**Verify a concrete returned path**, e.g.:

```bash
/path/from/previous/command -version
```

Do not paste a Java path from a different lab or assume that `java` on PATH is a suitable version.

## 3. Controller SSH host trust (jump host -> Jenkins server)

```bash
ssh jenkins@jenkins
whoami
echo "$HOME"  # expected /var/lib/jenkins
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts
ssh sarah@stapp01
```

Validate the displayed host key out of band; accept it if correct, enter the SSH password, then `exit` back to the Jenkins server:

```bash
ssh-keygen -F stapp01
```

Host-key trust alone is **not** agent-user authentication. Add a Jenkins **Username with password** (or approved SSH private key) credential for the app-server account.

## 4. Configure Jenkins SSH agent (UI)

Navigate **Manage Jenkins → Plugins**: ensure **SSH Build Agents** (`ssh-slaves`) is enabled; restart only if required by installation.

Navigate **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**. Add the approved SSH credential for `sarah` (no passwords in GitHub). Then **Manage Jenkins → Nodes → New Node**:

| Field | Value |
|---|---|
| Name | `App Server 1` |
| Type | Permanent Agent |
| Remote root | `/home/sarah/jenkins_agent` |
| Label | `stapp01` |
| Executors | `1` |
| Launch | Launch agents via SSH |
| Host / Port | `stapp01` / `22` |
| Credentials | Valid `sarah` SSH credential |
| Host key strategy | Known hosts file |
| Java Path | **Verified actual** compatible Java executable |

Save; **Launch agent → Log** and wait for `Agent successfully connected and online`.

## 5. Correct job type (UI)

**Jenkins → New Item → `xfusion-webapp-job` → Pipeline → OK**.

> During the lab an initial Freestyle project was selected by mistake. That empty/unbuilt item was recreated as a Pipeline. If you have a job containing real history/configuration, back it up rather than deleting it.

**Configure → Pipeline → Definition: Pipeline script** and paste:

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
    echo 'ERROR: Local changes exist; aborting' >&2
    exit 1
fi
git pull --ff-only
echo 'Deployment finished.'
curl --fail --silent --show-error http://localhost:8080/ -o /dev/null
echo 'Apache HTTP check passed.'
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

`skipDefaultCheckout(true)` is deliberate: the job updates the existing deployment checkout rather than creating another `/var/www/html/web_app` directory.

## 6. Execute and verify

1. **Save** the Pipeline.
2. **Build Now → Build #1 → Console Output**.
3. Verify the `Deploy` stage and `Finished: SUCCESS`.
4. Click the lab **App** link and verify the updated site loads at the **load balancer root** `/`, not under `/web_app/`.

Actual observed ending:

```text
Already up to date.
Deployment finished.
Apache HTTP check passed.
Deployment completed successfully.
Finished: SUCCESS
```

The grader subsequently marked **Day 77 Passed**.

## 7. Troubleshooting commands and checks

| Symptom | Diagnostic | Likely next action |
|---|---|---|
| Pipeline section missing | Check job item type and Pipeline plugin | Recreate only an empty wrong-type job as Pipeline |
| Agent offline | Jenkins agent **Log** | Identify authentication, host-key, Java or directory error |
| `UnsupportedClassVersionError` | Check agent Java version | Set agent-specific compatible Java Path |
| Exit code 127 launching Java | Check configured Java path exists | Replace with verified binary path |
| `fatal: detected dubious ownership` | `ls -ld /var/www/html /var/www/html/.git` | Correct approved ownership/access strategy |
| `Permission denied` in web root | `whoami; ls -ld /var/www/html` | Fix least-privilege deployment permissions |
| Git has local changes | `git status --short` | Stop and reconcile, do not destroy changes |
| Git branch lacks upstream | `git branch -vv` | Configure the intended deployment upstream deliberately |
| `git pull --ff-only` refused | `git status -sb; git log --oneline --graph -8` | Investigate divergence, not force/reset |
| HTTP check failed | `curl -v http://localhost:8080/` | Inspect Apache state/DocumentRoot, avoid blind service changes |
| LB shows stale version | Compare deployed commit and content, check LB | Verify routing/cache and requested branch |

**Safety:** No `git reset --hard`, `git clean -fdx`, `rm -rf /var/www/html`, or unconditional `chown -R` is part of this runbook.
