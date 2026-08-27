# DevOps Challenge — Day 28: Git Cherry-Pick

## Overview

This repository documents the successful completion of **DevOps Challenge Day 28**, focused on selectively moving a specific Git commit from one branch to another using **`git cherry-pick`**.

The Nautilus application development team was working with the repository `/opt/beta.git`, cloned under `/usr/src/kodekloudrepos/beta` on the Storage Server in **Stratos DC**. The repository contained two branches:

- `master`
- `feature`

A developer was still working on the `feature` branch, but one completed commit with the message **`Update info.txt`** needed to be applied to the `master` branch without merging the developer's other unfinished work.

---

## Task Requirements

The objective was to:

1. Access the cloned `beta` repository on the Storage Server.
2. Inspect the Git branches and commit history.
3. Identify the commit on the `feature` branch with the message:
   ```text
   Update info.txt
   ```
4. Switch to the `master` branch.
5. Apply only the required commit using `git cherry-pick`.
6. Verify that the working tree remained clean.
7. Push the updated `master` branch to the remote repository `/opt/beta.git`.
8. Confirm that the required commit was present on both local `master` and `origin/master`.

---

## Environment

| Component | Value |
|---|---|
| Storage Server | `ststor01` |
| Repository | `/opt/beta.git` |
| Local Clone | `/usr/src/kodekloudrepos/beta` |
| Source Branch | `feature` |
| Target Branch | `master` |
| Required Commit Message | `Update info.txt` |
| Selected Source Commit | `8ace519` |
| Cherry-picked Commit on `master` | `74f0c3c` |

> Git generates a new commit hash when a commit is cherry-picked because the new commit has a different parent history.

---

## Solution

### 1. Navigate to the repository

```bash
cd /usr/src/kodekloudrepos
ls -l
cd beta
```

### 2. Inspect repository status and branches

```bash
sudo git status
sudo git branch
```

The repository contained both `master` and `feature` branches.

### 3. Inspect the `feature` branch history

```bash
sudo git log feature --oneline
```

The commit with message **`Update info.txt`** was identified as:

```text
8ace519 Update info.txt
```

### 4. Switch to the target branch

```bash
sudo git checkout master
```

### 5. Cherry-pick the required commit

```bash
sudo git cherry-pick 8ace519
```

This copied only the required change from `feature` into `master`, while leaving the remaining in-progress feature work untouched.

### 6. Verify the result

```bash
sudo git log --oneline -5
sudo git status
```

The latest commit on `master` became:

```text
74f0c3c Update info.txt
```

The working tree was clean.

### 7. Push the updated branch

```bash
sudo git push origin master
```

The push completed successfully:

```text
c775ea5..74f0c3c  master -> master
```

### 8. Final verification

```bash
sudo git branch
sudo git log master --oneline -5
sudo git remote -v
```

Verified state:

```text
* master
  feature
```

```text
74f0c3c Update info.txt
c775ea5 Add welcome.txt
2f0dd51 initial commit
```

Remote repository:

```text
origin  /opt/beta.git (fetch)
origin  /opt/beta.git (push)
```

---

## Why `git cherry-pick`?

A normal merge such as:

```bash
git merge feature
```

could bring multiple commits from the `feature` branch into `master`.

That would be incorrect because the developer's feature work was still in progress. The requirement was to bring over only one completed commit.

`git cherry-pick` is designed exactly for this scenario:

```bash
git cherry-pick <commit-hash>
```

It takes the changes introduced by a specific commit and reapplies them on the currently checked-out branch.

---

## Key Git Concepts Practiced

- Viewing repository status
- Inspecting branches
- Reading branch-specific commit history
- Switching branches
- Selecting an individual commit
- Cherry-picking commits
- Understanding new commit hashes after cherry-pick
- Verifying local branch history
- Pushing changes to a remote repository
- Confirming remote configuration

---

## Result

✅ The commit **`Update info.txt`** was successfully selected from the `feature` branch.

✅ Only the required commit was applied to `master`.

✅ Unfinished work on `feature` was not merged.

✅ The updated `master` branch was pushed successfully to `/opt/beta.git`.

✅ The KodeKloud **DevOps Day 28 challenge was completed successfully**.

---
## Day 28 Summary

In DevOps Challenge Day 28, the task was to selectively move a completed Git commit from the `feature` branch to the `master` branch without merging the developer's unfinished feature work. After inspecting the repository history, the commit `8ace519` with the message `Update info.txt` was identified on the `feature` branch. The `master` branch was checked out, and `sudo git cherry-pick 8ace519` was used to apply only that commit. Git created the new commit `74f0c3c` on `master`, the repository was verified to have a clean working tree, and the updated branch was successfully pushed to the remote repository `/opt/beta.git`. This challenge demonstrated the practical use of Git cherry-pick for selectively transferring changes between branches while preserving branch isolation.
