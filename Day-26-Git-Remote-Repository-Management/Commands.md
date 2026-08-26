# DevOps Challenge - Day 26 Commands

## Objective

Add a new Git remote named `dev_apps`, copy `/tmp/index.html` into the `apps` repository, commit it to `master`, and push the branch to `/opt/xfusioncorp_apps.git`.

---

## 1. Connect to the Storage Server

```bash
ssh natasha@ststor01
```

---

## 2. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/apps
```

---

## 3. Check Repository Status

```bash
sudo git status
```

---

## 4. View Existing Remotes

```bash
sudo git remote -v
```

---

## 5. Add the New Remote

```bash
sudo git remote add dev_apps /opt/xfusioncorp_apps.git
```

---

## 6. Verify Remote Configuration

```bash
sudo git remote -v
```

Expected:

```text
dev_apps  /opt/xfusioncorp_apps.git (fetch)
dev_apps  /opt/xfusioncorp_apps.git (push)
origin    /opt/apps.git (fetch)
origin    /opt/apps.git (push)
```

---

## 7. Copy the Required File into the Repository

```bash
sudo cp /tmp/index.html /usr/src/kodekloudrepos/apps/
```

---

## 8. Check the New File

```bash
ls -l /usr/src/kodekloudrepos/apps/index.html
```

---

## 9. Check Git Status

```bash
sudo git status
```

---

## 10. Stage `index.html`

```bash
sudo git add index.html
```

---

## 11. Verify the Staging Area

```bash
sudo git status
```

---

## 12. Confirm the Current Branch

```bash
sudo git branch --show-current
```

Expected:

```text
master
```

---

## 13. Commit the File

```bash
sudo git commit -m "Add index.html"
```

Successful challenge commit:

```text
f534023 Add index.html
```

---

## 14. Check Commit History

```bash
sudo git log --oneline -3
```

---

## 15. Push `master` to the New Remote

```bash
sudo git push dev_apps master
```

---

## 16. Final Repository Status

```bash
sudo git status
```

---

## 17. Verify All Remotes

```bash
sudo git remote -v
```

---

## 18. Verify Local Git History

```bash
sudo git log --oneline --decorate -3
```

Verified result:

```text
f534023 (HEAD -> master, dev_apps/master) Add index.html
777dd25 (origin/master) initial commit
```

---

## 19. Verify the Destination Bare Repository

```bash
sudo git --git-dir=/opt/xfusioncorp_apps.git log --oneline -3
```

Verified result:

```text
f534023 (HEAD -> master) Add index.html
777dd25 initial commit
```

---

# Complete Command Sequence

```bash
ssh natasha@ststor01

cd /usr/src/kodekloudrepos/apps

sudo git status
sudo git remote -v

sudo git remote add dev_apps /opt/xfusioncorp_apps.git
sudo git remote -v

sudo cp /tmp/index.html /usr/src/kodekloudrepos/apps/

ls -l /usr/src/kodekloudrepos/apps/index.html
sudo git status

sudo git add index.html
sudo git status

sudo git branch --show-current

sudo git commit -m "Add index.html"

sudo git log --oneline -3

sudo git push dev_apps master

sudo git status
sudo git remote -v
sudo git log --oneline --decorate -3

sudo git --git-dir=/opt/xfusioncorp_apps.git log --oneline -3
```

---

## Optional Troubleshooting Commands

### Show Remote Details

```bash
sudo git remote show dev_apps
```

### Correct the Remote URL if Needed

```bash
sudo git remote set-url dev_apps /opt/xfusioncorp_apps.git
```

### Show All Local and Remote Branches

```bash
sudo git branch -a
```

### Show Recent Commits

```bash
sudo git log --oneline --decorate --graph -5
```

---

**Day 26: Completed Successfully ✅**
