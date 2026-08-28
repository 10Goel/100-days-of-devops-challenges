# DevOps Day 31 — Restore Git Stash, Commit, and Push

## 📌 Challenge Overview

On **Day 31** of the DevOps challenge, the Nautilus application development team needed to restore previously stashed work from the Git repository located at:

```bash
/usr/src/kodekloudrepos/apps
```

The required changes were stored in the specific stash entry:

```bash
stash@{1}
```

The goal was to restore those changes, commit them to the current branch, and push the new commit to the remote `origin` repository.

---

## 🎯 Task Requirements

- Access the Git repository on the **Storage Server**.
- Locate the available Git stash entries.
- Restore the changes stored in `stash@{1}`.
- Verify the restored changes.
- Stage the restored files.
- Commit the changes.
- Push the commit to the remote repository.
- Confirm that the local branch is synchronized with `origin`.

---

## 🖥️ Repository Details

| Item | Value |
|---|---|
| Repository Path | `/usr/src/kodekloudrepos/apps` |
| Server | Storage Server |
| Branch | `master` |
| Required Stash | `stash@{1}` |
| Remote | `origin` |
| Commit Message | `Restore stashed changes` |

---

## 🚀 Solution Summary

The repository was accessed on the Storage Server and the current Git state was verified. The available stash entries were checked, after which the required `stash@{1}` entry was applied using `git stash apply`.

The restored changes were reviewed using `git status` and `git diff`, staged with `git add .`, and committed with the message:

```text
Restore stashed changes
```

The new commit was then pushed successfully to `origin/master`.

---

## 🔧 Commands Used

```bash
cd /usr/src/kodekloudrepos/apps

sudo git status
sudo git branch --show-current

sudo git stash list
sudo git stash apply 'stash@{1}'

sudo git status
sudo git diff

sudo git add .
sudo git commit -m "Restore stashed changes"

sudo git log -1 --oneline
sudo git branch --show-current

sudo git push origin master

sudo git status
sudo git log --oneline --decorate -3
```

---

## ✅ Verification

The final verification confirmed that:

- The restored stash changes were committed successfully.
- The commit was pushed successfully to `origin/master`.
- The local `master` branch was synchronized with the remote branch.
- The working tree was clean.

Final Git status:

```text
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

The latest commit was:

```text
da32f2 Restore stashed changes
```

---

## 🧠 Key Learning

This challenge reinforced practical Git stash management. A stash allows developers to temporarily save uncommitted work without creating a commit. Git can maintain multiple stash entries, identified as `stash@{0}`, `stash@{1}`, and so on. When a task requires a specific stash, it is important to explicitly reference that stash instead of restoring the latest one by default.

Using:

```bash
git stash apply 'stash@{1}'
```

restores the requested stash while keeping the stash entry in the stash list. This is different from `git stash pop`, which restores the stash and removes it if the operation succeeds.

---

## 📚 Concepts Covered

- Git stash management
- Viewing stash entries
- Restoring a specific stash
- Working tree verification
- Staging changes
- Git commits
- Git remote repositories
- Pushing commits
- Branch synchronization
- Git history verification

---

## 🏁 Result

**DevOps Day 31 challenge completed successfully.** ✅

The changes from `stash@{1}` were restored, committed, and pushed successfully to the remote `master` branch.
