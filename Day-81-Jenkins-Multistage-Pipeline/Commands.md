# Day 81 — Commands and Configuration Runbook

This runbook documents the **passed** Jenkins `Deploy` + `Test` challenge. Replace any lab-specific passwords with the values from KodeKloud's **Details of all Users and Servers**. Do **not** commit credentials to Git.

## 1. App Server 1: Inspect the existing deployment repository

From `thor@jumphost`:

```bash
ssh sarah@stapp01
whoami
hostname
cd /var/www/html
pwd
git status -sb
git branch --show-current
git remote -v
ls -ld /var/www/html /var/www/html/.git
```

Expected: Linux user `sarah`, host `stapp01`, branch `master`, and `origin` configured for the `sarah/web` Gitea repository. If working-tree changes are already present, review them first. Never use destructive Git cleanup as a routine shortcut.

## 2. Update the homepage and push to Gitea

```bash
cd /var/www/html
vi index.html
```

Edit existing HTML so the page displays this exact text:

```text
Welcome to xFusionCorp Industries
```

Then:

```bash
git diff -- index.html
git add index.html
git commit -m "Update homepage for Day 81"
git push origin master
git status -sb
git log -1 --oneline
```

If the message was already committed and the working tree is clean, don't create an artificial extra commit.

## 3. Check Apache and the website

```bash
sudo systemctl is-active httpd
curl -fsS http://localhost:8080/
curl -fsS http://stlb01:8091/
```

To verify the task's expected message **outside** the Pipeline:

```bash
curl -fsS http://localhost:8080/ | grep -F 'Welcome to xFusionCorp Industries'
curl -fsS http://stlb01:8091/ | grep -F 'Welcome to xFusionCorp Industries'
```

The load balancer URL must serve content at the **root** (`/`), not at `/web`.

## 4. Prepare Java and the Jenkins SSH agent work directory

On App Server 1:

```bash
java -version
command -v java
readlink -f "$(command -v java)"
```

If Java is missing and the challenge's host uses DNF:

```bash
sudo dnf install -y java-17-openjdk
```

**Important:** Confirm the remoting plugin supports the installed Java version. Do not hardcode version-specific Java paths from previous lab attempts.

```bash
mkdir -p /home/sarah/jenkins_agent
chmod 755 /home/sarah/jenkins_agent
ls -ld /home/sarah/jenkins_agent
test -w /home/sarah/jenkins_agent && echo 'Agent directory writable'
exit
```

## 5. Establish trusted host-key verification from the Jenkins Linux account

From the jump host:

```bash
ssh jenkins@jenkins
whoami
echo "$HOME"
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts
ssh sarah@stapp01
```

At the host-key prompt, **verify the fingerprint against trusted lab data** before accepting. Supply the App Server 1 **Linux** password, not Gitea login credentials. On App Server 1:

```bash
whoami
hostname
exit
```

Back on Jenkins:

```bash
ssh-keygen -F stapp01
exit
```

This builds trust for the controller-side **SSH Build Agents plugin**. Adding the host key to the jump host's own `known_hosts` is insufficient.

## 6. Jenkins UI: Create an SSH credential

1. **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**.
2. Kind: **Username with password**.
3. Username: `sarah`.
4. Password: App Server 1's Linux SSH password (enter only in Jenkins credential storage).
5. ID: `day81-sarah-ssh`.
6. Description: `SSH Agent - App Server 1`.
7. Save.

For production, SSH private-key credentials usually offer better lifecycle controls; the lab used username/password authentication.

## 7. Jenkins UI: Create and launch the agent

First ensure **SSH Build Agents**, **Pipeline**, and **Credentials** plugins are installed and enabled. Restart Jenkins after plugin installation if necessary.

**Manage Jenkins → Nodes → New Node**:

| Field | Exact value |
|---|---|
| Node name | `App Server 1` |
| Node type | Permanent Agent |
| Executors | `1` |
| Remote root | `/home/sarah/jenkins_agent` |
| Labels | `stapp01` |
| Usage | Use this node as much as possible |
| Launch method | Launch agents via SSH |
| Host | `stapp01` |
| Port | `22` |
| Credentials | `day81-sarah-ssh` |
| Host-key strategy | Known hosts file Verification Strategy |
| Java Path | Verified compatible Java executable, e.g. `/usr/bin/java` if valid |

**Save → Nodes → App Server 1 → Launch agent → Log**.

Success indicator:

```text
Agent successfully connected and online
```

Do not continue with the Pipeline while the agent is offline.

## 8. Jenkins UI: Create the Pipeline job

**Dashboard → New Item → `deploy-job` → Pipeline → OK**. Use **Pipeline script** for the definition. Do not select Multibranch Pipeline.

Paste the final Pipeline:

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
                    GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master
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

**Save → Build Now → Build #1 → Console Output**.

Expected *patterns* (not a claim about the final numbered build):

```text
Running on App Server 1
[Pipeline] { (Deploy)
Deploy stage completed successfully.
[Pipeline] { (Test)
Website is accessible. Test passed.
Finished: SUCCESS
```

## 9. Final validation before clicking KodeKloud Check

Check Jenkins job type, node label, stage names, browser output and HTTP responses. On App Server 1:

```bash
cd /var/www/html
git status -sb
git branch --show-current
git log -1 --oneline
curl -fsS http://localhost:8080/ | grep -F 'Welcome to xFusionCorp Industries'
curl -fsS http://stlb01:8091/ | grep -F 'Welcome to xFusionCorp Industries'
```

Click **App** and verify the page at `/`. After the final correct health-check configuration, **KodeKloud marked Day 81 PASSED**.

## 10. Troubleshooting commands

```bash
# Check local service state
sudo systemctl status httpd --no-pager
sudo ss -lntp | grep ':8080'

# Check git state without changing history
git -C /var/www/html status --porcelain
git -C /var/www/html remote -v

# Confirm the host key Jenkins recorded (run as Linux user jenkins)
ssh-keygen -F stapp01

# Print HTTP status code against the load balancer
curl -sS -o /dev/null -w '%{http_code}\n' http://stlb01:8091/

# Check if installed Java exists
command -v java
java -version
```

**Do not** disable host-key verification, add passwords to scripts, delete Git history, or grant unrestricted sudo to fix an unrelated failure.
