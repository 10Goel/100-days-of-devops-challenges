# Day 80 — Commands and Execution Runbook

> **Scope:** KodeKloud lab. Verify account names, Git remotes, sudo policy, and Apache port in your environment before executing changes. Commands assume App Server 1 is `stapp01` and its login is `sarah`. Do not run any destructive Git cleanup.

## 1. Inspect the existing website on App Server 1

From the jump host:

```bash
ssh sarah@stapp01
whoami
hostname
ls -ld /var/www/html /var/www/html/.git
git -C /var/www/html status -sb
git -C /var/www/html branch -vv
git -C /var/www/html remote -v
sudo systemctl is-active httpd
sudo ss -lntp | grep httpd
curl -I http://localhost:8080/
```

Record any uncommitted or untracked files. Confirm the upstream branch and remote before updating. If `/var/www/html` is owned by another user, inspect permissions rather than immediately changing them recursively.

Check privileges (may prompt once interactively):

```bash
sudo -l
sudo -n -l
command -v systemctl
exit
```

## 2. Establish Jenkins → App Server SSH

```bash
# On jumphost
ssh jenkins@jenkins
whoami
echo "$HOME"
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts

# Verify fingerprint against the trusted lab host before accepting
ssh sarah@stapp01
# after successful login:
exit
ssh-keygen -F stapp01

# Generate only if ~/.ssh/id_ed25519 does not already exist
ls -la ~/.ssh/
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
ssh-copy-id -i ~/.ssh/id_ed25519.pub sarah@stapp01

# Verify noninteractive login
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes \
    sarah@stapp01 'whoami && hostname'
ssh -o BatchMode=yes sarah@stapp01 \
    'git -C /var/www/html status -sb'
exit
```

Do **not** overwrite a pre-existing SSH key: omit the `ssh-keygen` command if it exists. Keep private keys on the Jenkins host. Use a managed Jenkins credential in production.

## 3. Create upstream Jenkins job (GUI)

1. **Jenkins Dashboard → New Item**.
2. Name: `xfusion-app-deployment`.
3. Type: **Freestyle project**.
4. **Source Code Management: None** — the checkout already resides on App Server 1.
5. **Build Steps → Add build step → Execute shell**.
6. Paste:

```bash
#!/bin/sh
set -eu

echo 'Day 80 upstream: updating existing website checkout'
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes sarah@stapp01 '
set -eu
cd /var/www/html

test -d .git
test -w .
test -w .git

if [ -n "$(git status --porcelain)" ]; then
    echo "ERROR: working tree is not clean; not overwriting edits"
    exit 1
fi

branch=$(git branch --show-current)
if [ "$branch" != master ]; then
    echo "ERROR: expected master; found $branch"
    exit 1
fi

GIT_TERMINAL_PROMPT=0 git pull --ff-only origin master
git log -1 --oneline
test -f index.html
echo "SUCCESS: deployment update completed"
'
```

7. Save. **Build Now** and inspect console. Require `Finished: SUCCESS` before configuring the dependent trigger.

> `git pull --ff-only` is intentionally non-destructive; if the branch diverges, diagnose rather than forcibly resetting it.

## 4. Create downstream Jenkins job (GUI)

1. **New Item → `manage-services` → Freestyle project**.
2. Under **Build Triggers** check **Build after other projects are built**.
3. **Projects to watch:** `xfusion-app-deployment`.
4. **Trigger only if build is:** **Stable** (not always/unstable).
5. Under **Build Steps → Execute shell**, paste:

```bash
#!/bin/sh
set -eu

echo 'Day 80 downstream: restarting Apache'
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes sarah@stapp01 '
set -eu
echo "Host: $(hostname)"
echo "User: $(whoami)"
sudo -n systemctl restart httpd
sudo -n systemctl is-active --quiet httpd
curl --fail --silent --show-error http://localhost:8080/ -o /dev/null
echo "SUCCESS: Apache restarted and website responded"
'
```

6. Save. **Do not manually trigger downstream** during dependency verification.

## 5. Resolve the observed sudo failure

**Actual failure:**

```text
Started by upstream project "xfusion-app-deployment" build number 3
Host: stapp01
User: sarah
Restarting httpd...
sudo: a password is required
Finished: FAILURE
```

SSH and chained triggers were functioning, but `sudo -n` could not prompt for a password. On App Server 1, after reviewing the existing sudo policy and confirming administrative authorization:

```bash
ssh sarah@stapp01
command -v systemctl
sudo -l
```

Use `sudo visudo -f /etc/sudoers.d/jenkins-httpd` to safely add a **narrow** rule (only if `/usr/bin/systemctl` is confirmed as the executable path):

```sudoers
sarah ALL=(root) NOPASSWD: /usr/bin/systemctl restart httpd, /usr/bin/systemctl is-active --quiet httpd
```

Save, then verify:

```bash
sudo visudo -cf /etc/sudoers.d/jenkins-httpd
sudo -n -l
sudo -n systemctl is-active --quiet httpd && echo 'Status check authorized'
exit
```

If `systemctl` is located elsewhere, substitute the actual absolute path in the rule. Do not change general sudo authentication requirements or grant `NOPASSWD: ALL`.

Check the same access from the Jenkins Linux account:

```bash
ssh jenkins@jenkins
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes sarah@stapp01 \
  'sudo -n systemctl is-active --quiet httpd && echo "Authorized"'
exit
```

## 6. Verify automatic chaining

1. In Jenkins, open **xfusion-app-deployment → Build Now**.
2. Wait for upstream build result **SUCCESS**.
3. Open **manage-services → Build History**: a new build should start automatically.
4. Console should show **Started by upstream project ...**, then Apache restart and HTTP check.
5. Downstream should finish **SUCCESS**.
6. Check site via KodeKloud **App** at the load balancer root URL.

Optional host verification:

```bash
ssh sarah@stapp01
git -C /var/www/html branch --show-current
git -C /var/www/html log -1 --oneline
sudo systemctl is-active httpd
curl -I http://localhost:8080/
exit
```

**Outcome:** Day 80 validated as **PASSED**. Avoid inventing a final build number; record the real number from your Jenkins console when publishing screenshots.
