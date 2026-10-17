# Day 79 — Commands and Configuration Runbook

> Commands are grouped by **where they run**. This is a reproducible lab runbook; adjust hostnames/credentials to the *current* lab. Do not paste secrets into GitHub. Execute destructive or environment-changing steps only after checking their targets.

## 0. Task values

```text
Jenkins job:         xfusion-app-deployment
Source host:         stapp01
Source SSH user:     sarah
Source Git checkout: /home/sarah/web
Source branch:       master
Destination:         /var/www/html
Web server:          httpd :8080
Jenkins trigger:     Generic Webhook Trigger
```

## 1. Jump host → application server (run as Sarah)

```bash
ssh sarah@stapp01
whoami
hostname
pwd
git -C /home/sarah/web status -sb
git -C /home/sarah/web branch -vv
git -C /home/sarah/web remote -v
ls -la /home/sarah/web
ls -ld /var/www/html
ls -la /var/www/html
```

Set the directory owner **only because the challenge expressly requires Sarah to own it**:

```bash
sudo chown -R sarah:sarah /var/www/html
stat -c '%U:%G %n' /var/www/html
test -w /var/www/html && echo 'Deployment directory writable'
```

Verify Apache and tools:

```bash
sudo systemctl is-active httpd
sudo ss -lntp | grep ':8080'
curl -I http://localhost:8080/
command -v git
command -v rsync
command -v curl
command -v bash
```

If needed, **only on the app server**:

```bash
sudo systemctl start httpd
# On a DNF-based lab, only if rsync is absent:
sudo dnf install -y rsync
```

Then `exit` to return to the jump host.

## 2. Jump host → Jenkins Linux server

```bash
ssh jenkins@jenkins
whoami
hostname
echo "$HOME"                  # /var/lib/jenkins
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts
```

Establish trusted SSH identity and confirm the correct account:

```bash
ssh sarah@stapp01
# Verify host key against trusted lab information before accepting.
whoami
hostname
exit
ssh-keygen -F stapp01
```

Create a key **only if one does not already exist**:

```bash
ls -la ~/.ssh
# If needed (never overwrite an existing key):
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
ssh-copy-id -i ~/.ssh/id_ed25519.pub sarah@stapp01
```

**Validated SSH access used for the successful challenge:**

```bash
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes   sarah@stapp01 'whoami && hostname'
ssh -o BatchMode=yes sarah@stapp01   'test -d /home/sarah/web/.git && echo "Repository found"'
ssh -o BatchMode=yes sarah@stapp01   'test -w /var/www/html && echo "Document root writable"'
ssh -o BatchMode=yes sarah@stapp01   'command -v rsync && command -v git && command -v curl'
exit
```

## 3. Jenkins UI: Built-In Node and Pipeline job

1. `Manage Jenkins → Nodes → Built-In Node`.
2. Confirm **Online**, **2 executors**, and **no configured labels** in the lab.
3. Create **New Item → xfusion-app-deployment → Pipeline** (not Freestyle, not Multibranch).
4. `Configure → Pipeline → Definition: Pipeline script`.
5. Paste the following complete code and **Save**.

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

                    echo "Starting Day 79 website deployment"

                    ssh -o BatchMode=yes -o StrictHostKeyChecking=yes sarah@stapp01 'bash -se' <<'REMOTE'
set -euo pipefail

SOURCE="/home/sarah/web"
DEST="/var/www/html"

echo "Remote host: $(hostname)"
echo "Remote user: $(whoami)"

cd "$SOURCE"
test -d .git
test -w "$DEST"

if [ -n "$(git status --porcelain)" ]; then
    echo "ERROR: Source repository contains uncommitted changes."
    exit 1
fi

git switch master
GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master

rsync -av --exclude='.git/' "$SOURCE/" "$DEST/"

test -f "$DEST/index.html"
curl -fsS http://localhost:8080/ -o /dev/null

echo "Deployed commit: $(git rev-parse --short HEAD)"
echo "SUCCESS: Website deployment completed."
REMOTE
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment failed. Review Console Output.'
        }
    }
}
```

**Heredoc caution:** The closing `REMOTE` must begin at column 1 in the script material Jenkins executes. Preserve the code above intact. The Jenkins-side `sh` uses portable `set -eu`; the remote side explicitly invokes Bash for `pipefail`.

### Manual smoke test

`xfusion-app-deployment → Build Now → #1 → Console Output`

Verify `Running on Jenkins`, `Deploy`, remote host `stapp01`, and `Finished: SUCCESS`. Do this *before* introducing the webhook.

## 4. Jenkins UI: Generic Webhook Trigger

Install/enable the **Generic Webhook Trigger** plugin if absent. Then:

`xfusion-app-deployment → Configure → Triggers → Generic Webhook Trigger`

**Post content parameters**:

| Field | Value |
| --- | --- |
| Variable | `ref` |
| Expression | `$.ref` |
| Expression type | `JSONPath` |
| Value filter | Empty |
| Default value | Empty |

Generate a unique secret token and store it in the trigger's **Token** field:

```bash
openssl rand -hex 24
```

**Optional filter**:

```text
Expression (regexp): ^refs/heads/master$
Text:                $ref
```

Save. Never publish the actual token or live webhook URL.

## 5. Gitea UI: repository webhook

`Gitea → sarah/web → Settings → Webhooks → Add Webhook → Gitea`

**Target URL pattern** (replace placeholders with the current lab URL and private token):

```text
https://<CURRENT-JENKINS-BASE-URL>/generic-webhook-trigger/invoke?token=<PRIVATE-TOKEN>
```

| Setting | Value |
| --- | --- |
| HTTP Method | `POST` |
| POST Content Type | `application/json` |
| Active | Checked |
| Trigger On | `Push Events` |
| Branch filter | `master` |
| Authorization header | Empty for URL token configuration |
| Secret | Empty for URL token configuration |

After **Add Webhook**, inspect **Recent Deliveries**. An HTTP success response establishes delivery but not necessarily a started build. For the definitive test, push a real commit and inspect Jenkins Build History.

## 6. App Server 1: developer change and Git push

```bash
ssh sarah@stapp01
cd /home/sarah/web
git status -sb
git branch --show-current
git remote -v
vi index.html
```

Update the homepage text to the exact required phrase:

```text
Welcome to the xFusionCorp Industries
```

Verify and push:

```bash
git diff -- index.html
git add index.html
git commit -m "Update homepage welcome message"
git push origin master
git log -1 --oneline
```

**Observed successful task evidence:** commit `b6b6262` reached `origin/master`. The webhook then automatically started Jenkins Build `#3` with **Generic Cause**, and it finished **SUCCESS**. Do not manually trigger the build after this push when validating the automatic flow.

## 7. Deployment verification

```bash
# On stapp01:
diff -q /home/sarah/web/index.html /var/www/html/index.html
curl -fsS http://localhost:8080/ | grep -F   'Welcome to the xFusionCorp Industries'
rsync -avnc --exclude='.git/' /home/sarah/web/ /var/www/html/
ls -la /var/www/html
```

The `diff -q` should be silent for identical files. Check the actual homepage via KodeKloud **App** at the **root** path (`/`), not `/web`.

## 8. Troubleshooting quick reference

```text
Build waits forever              → inspect Nodes; avoid guessed labels
Invalid option type timestamps  → remove unsupported timestamps() option
Illegal option -o pipefail      → default Jenkins /bin/sh is not Bash
Permission denied (SSH)         → verify Jenkins key and authorized_keys
Host key verification failure   → check Jenkins user's known_hosts
Git asks for credentials        → provide authorized noninteractive Git auth
rsync command not found         → install rsync on stapp01
HTTP success, old page          → compare Git commit, source, dest and page content
Webhook delivered, no build     → inspect token, $.ref, master regexp and trigger state
```

## Operational cautions

- `sudo chown -R` was a **lab-specific requirement**, not a general production recommendation.
- `rsync -av` copies the entire website tree without deleting files removed upstream; review synchronization policy for production.
- Keep the Jenkins private key, webhook token and Gitea credentials out of the repository.
- Jenkins controller-side build execution is acceptable for the exercise, but dedicated agents are preferred in production.
