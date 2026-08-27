# DevOps Challenge — Day 28 Commands

## Git Cherry-Pick Command Reference

This file contains the commands used to complete **DevOps Day 28**.

---

## 1. Connect to the Storage Server

```bash
ssh natasha@ststor01
```

---

## 2. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos
ls -l
cd beta
```

Optional verification:

```bash
pwd
```

Expected path:

```text
/usr/src/kodekloudrepos/beta
```

---

## 3. Check Repository Status

```bash
sudo git status
```

Purpose: Verify the current branch and ensure there are no unexpected local changes.

---

## 4. List Branches

```bash
sudo git branch
```

Expected branches:

```text
feature
master
```

---

## 5. Inspect the `feature` Branch Commit History

```bash
sudo git log feature --oneline
```

The required commit was:

```text
8ace519 Update info.txt
```

A more targeted search can also be used:

```bash
sudo git log feature --oneline --grep="Update info.txt"
```

---

## 6. Switch to `master`

```bash
sudo git checkout master
```

Verify:

```bash
sudo git branch
```

Expected:

```text
  feature
* master
```

---

## 7. Cherry-Pick the Required Commit

```bash
sudo git cherry-pick 8ace519
```

Resulting commit on `master`:

```text
74f0c3c Update info.txt
```

---

## 8. Verify Commit History

```bash
sudo git log --oneline -5
```

Observed history:

```text
74f0c3c Update info.txt
c775ea5 Add welcome.txt
2f0dd51 initial commit
```

---

## 9. Verify Working Tree

```bash
sudo git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

## 10. Push `master` to the Remote Repository

```bash
sudo git push origin master
```

Successful push:

```text
c775ea5..74f0c3c  master -> master
```

---

## 11. Final Branch Verification

```bash
sudo git branch
```

---

## 12. Verify `master` History

```bash
sudo git log master --oneline -5
```

---

## 13. Verify Remote Configuration

```bash
sudo git remote -v
```

Observed remote:

```text
origin  /opt/beta.git (fetch)
origin  /opt/beta.git (push)
```

---

## Complete Command Sequence

```bash
ssh natasha@ststor01

cd /usr/src/kodekloudrepos
ls -l
cd beta

sudo git status
sudo git branch
sudo git log feature --oneline

sudo git checkout master

sudo git cherry-pick 8ace519

sudo git log --oneline -5
sudo git status

sudo git push origin master

sudo git branch
sudo git log master --oneline -5
sudo git remote -v
```

---

## Useful Additional Commands

### Inspect the selected commit before cherry-picking

```bash
sudo git show 8ace519
```

### Inspect only files changed in the commit

```bash
sudo git show --stat 8ace519
```

### Verify the remote-tracking branch

```bash
sudo git log origin/master --oneline -5
```

### If a cherry-pick conflict occurs

Check status:

```bash
sudo git status
```

Resolve the conflicting files, then stage them:

```bash
sudo git add <file>
```

Continue:

```bash
sudo git cherry-pick --continue
```

Abort the operation if necessary:

```bash
sudo git cherry-pick --abort
```

---

## Core Command of Day 28

```bash
sudo git cherry-pick 8ace519
```

This command selectively applied the required `Update info.txt` commit from `feature` to `master` without merging the rest of the unfinished feature branch.
