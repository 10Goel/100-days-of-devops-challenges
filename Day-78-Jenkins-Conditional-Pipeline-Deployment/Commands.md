# Day 78 — Commands and Configuration Runbook

> Run commands only in the specified environment. Substitute values discovered in your current lab; never reuse version-specific Java paths from a prior lab.

## 1. App Server 1 — inspect the checkout (read-only)
From the jump host, using the account supplied in lab infrastructure details:

```bash
ssh sarah@stapp01
whoami
hostname
java -version
ls -ld /var/www/html /var/www/html/.git
cd /var/www/html
git status --short
git remote -v
git branch -a
git branch -vv
```

Do not use `git reset --hard` or `git clean` to solve an unexpected working-tree state.

## 2. Prepare remote Jenkins root and Java
On `stapp01`:

```bash
mkdir -p /home/sarah/jenkins_agent
chmod 755 /home/sarah/jenkins_agent
test -w /home/sarah/jenkins_agent && echo 'Agent root writable'

# Only if a suitable Java runtime is unavailable:
sudo dnf install -y java-21-openjdk-headless
JAVA21=$(rpm -ql java-21-openjdk-headless | grep '/bin/java$' | head -n 1)
test -n "$JAVA21" && test -x "$JAVA21" && "$JAVA21" -version
printf 'Jenkins agent Java Path: %s\n' "$JAVA21"
exit
```

## 3. Jenkins controller — trusted SSH host keys
From jump host:

```bash
ssh jenkins@jenkins
whoami
echo "$HOME" # commonly /var/lib/jenkins
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts
ssh sarah@stapp01
# Verify the host fingerprint against trusted lab information before accepting.
# Authenticate, then on stapp01:
exit
# Back on jenkins@jenkins:
ssh-keygen -F stapp01
ls -l ~/.ssh/known_hosts
exit
```

**Note:** A host key in `known_hosts` establishes trust in the server identity; it does not replace SSH user credentials.

## 4. Jenkins agent configuration
Jenkins UI: **Manage Jenkins → Credentials**: add the authorized `sarah` SSH credential, stored as a Jenkins secret.

**Manage Jenkins → Nodes → New Node**:

| Setting | Value |
|---|---|
| Name | `App Server 1` |
| Type | Permanent Agent |
| Executors | 1 |
| Remote root directory | `/home/sarah/jenkins_agent` |
| Label | `stapp01` |
| Launcher | Launch agents via SSH |
| Host / Port | `stapp01` / `22` |
| Credentials | Jenkins credential for `sarah` |
| Host key strategy | Known hosts file |
| Java Path | Actual, tested Java 21 executable |

Save, launch, and verify the node displays **Online**.

## 5. Jenkins Pipeline job configuration
- New Item: `datacenter-webapp-job`
- Type: **Pipeline**, not Freestyle / Multibranch
- General: **This project is parameterized**, add **String Parameter** `BRANCH`, default `master`
- Pipeline: **Definition → Pipeline script**; add the script below.

### Jenkins Pipeline script

```groovy
pipeline {
    agent { label 'stapp01' }

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master',
               description: 'Branch to deploy: master or feature')
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''#!/bin/bash
set -euo pipefail

DEPLOY_DIR="/var/www/html"
: "${BRANCH:?BRANCH is required}"

if [[ "$BRANCH" != "master" && "$BRANCH" != "feature" ]]; then
    echo "ERROR: Unsupported BRANCH: $BRANCH" >&2
    exit 1
fi

echo "Deploying branch $BRANCH on $(hostname)"
cd "$DEPLOY_DIR"
git rev-parse --is-inside-work-tree

if [[ -n "$(git status --porcelain)" ]]; then
    echo 'ERROR: Existing checkout has uncommitted changes' >&2
    exit 1
fi

git fetch origin
git show-ref --verify --quiet "refs/remotes/origin/$BRANCH"

if git show-ref --verify --quiet "refs/heads/$BRANCH"; then
    git switch "$BRANCH"
else
    git switch --track -c "$BRANCH" "origin/$BRANCH"
fi

git merge --ff-only "origin/$BRANCH"
echo "Deployed branch: $(git branch --show-current)"
git log -1 --oneline
curl --fail --silent --show-error http://localhost:8080/ -o /dev/null
echo "SUCCESS: $BRANCH deployment completed"
'''
            }
        }
    }

    post {
        success { echo 'Conditional deployment completed successfully.' }
        failure { echo 'Deployment failed. Inspect Console Output.' }
    }
}
```

> If you configure the parameter in both UI and Jenkinsfile, Jenkins may update the job's parameter definition after its first Pipeline execution. The finalized job needs only one authoritative definition. Confirm `Build with Parameters` exposes `BRANCH`.

**Run prerequisite:** The Jenkins agent user must be allowed to write/update `/var/www/html` and fetch from the correct Gitea remote. Resolve authorization issues through approved permissions, not broad recursive `chmod`/`chown` or a password embedded in source.

## 6. Run and verify two builds

| Build | BRANCH value | Verify |
|---|---|---|
| 1 | `master` | `Finished: SUCCESS`, deployed branch `master` |
| 2 | `feature` | `Finished: SUCCESS`, deployed branch `feature` |

On App Server 1, verify current deployed branch and commit:

```bash
cd /var/www/html
git branch --show-current
git log -1 --oneline
curl -fsS http://localhost:8080/ -o /dev/null && echo 'Apache responds'
```

Use the lab's **App** button and inspect the site at the load balancer's **root URL** (`/`). Check a branch-specific page or change if available; HTTP 200 alone doesn't prove which branch is deployed.

## 7. Useful diagnostics

```bash
# Host and agent runtime
hostname
whoami
java -version

# Repository safety and tracking
git -C /var/www/html status --short
git -C /var/www/html remote -v
git -C /var/www/html branch -vv
git -C /var/www/html rev-parse HEAD

# Web serving
curl -I http://localhost:8080/
```

**Final evidence:** Build #2 printed `feature`, commit `e93dfcf Added feature.html file`, `SUCCESS: feature deployment completed`, and `Finished: SUCCESS`. KodeKloud Day 78 was marked **Passed**.
