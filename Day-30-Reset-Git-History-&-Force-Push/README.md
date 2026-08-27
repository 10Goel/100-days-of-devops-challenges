# DevOps Challenge – Day 30: Reset Git History and Force Push

## 📌 Challenge Overview

The Nautilus application development team was working on the Git repository located at:

```bash
/usr/src/kodekloudrepos/media
```

on the **Storage Server** in **Stratos DC**.

The repository was being used for testing, and a developer had pushed several unnecessary commits. The requirement was to clean the repository history so that only the following two commits remained:

1. `initial commit`
2. `add data.txt file`

The branch `HEAD` also had to point to the `add data.txt file` commit, and the cleaned history had to be pushed to the remote repository.

---

## 🎯 Objective

- Access the `media` Git repository on the Storage Server.
- Inspect the existing Git commit history.
- Identify the commit with message `add data.txt file`.
- Reset the branch history to that commit.
- Ensure only two commits remain in the repository history.
- Force-push the rewritten history to the remote `master` branch.
- Verify that both local and remote branches point to the correct commit.

---

## 🛠️ Environment

| Component | Value |
|---|---|
| Server | Storage Server |
| Repository | `/usr/src/kodekloudrepos/media` |
| Branch | `master` |
| Target Commit | `add data.txt file` |
| Remote | `origin` |
| Git Operation | Hard Reset + Force Push |

---

## 🚀 Implementation

### 1. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/media
```

### 2. Verify Repository Status and Branch

```bash
sudo git status
sudo git branch -vv
sudo git remote -v
```

### 3. Inspect Commit History

```bash
sudo git log --oneline --decorate --graph
```

The required commit was identified as:

```text
f8befc4 add data.txt file
```

The initial commit was:

```text
291e7a2 initial commit
```

### 4. Reset the Branch History

The branch was reset to the `add data.txt file` commit:

```bash
sudo git reset --hard f8befc4
```

This moved `HEAD` and the local `master` branch back to the required commit and removed the later commits from the current branch history.

### 5. Verify the Cleaned History

```bash
sudo git log --oneline --decorate --graph
```

The history now contained only:

```text
f8befc4 (HEAD -> master) add data.txt file
291e7a2 initial commit
```

### 6. Push the Rewritten History

Because the remote branch still contained the unwanted commits, a normal push would not be sufficient. The rewritten branch was force-pushed:

```bash
sudo git push --force origin master
```

The push completed successfully with a forced update.

### 7. Final Verification

```bash
sudo git log --oneline
sudo git fetch origin
sudo git log --oneline --decorate --all --graph
```

Final result:

```text
f8befc4 (HEAD -> master, origin/master) add data.txt file
291e7a2 initial commit
```

---

## ✅ Final Result

The Git repository was successfully cleaned and now contains only the required two commits:

```text
291e7a2 initial commit
        ↓
f8befc4 add data.txt file
        ↑
HEAD -> master
origin/master
```

Both the local `master` branch and remote `origin/master` branch point to the `add data.txt file` commit.

---

## 🧠 Key Concepts Learned

- Viewing Git commit history
- Understanding `HEAD` and branch pointers
- Using `git reset --hard`
- Rewriting Git history
- Understanding non-fast-forward updates
- Using `git push --force`
- Verifying synchronization between local and remote branches
- Understanding the difference between `git reset` and `git revert`

---

## ⚠️ Important Note

`git push --force` rewrites remote Git history and can overwrite commits that other developers may depend on. It should only be used when history rewriting is intentional and coordinated.

In this challenge, force pushing was required because the task explicitly asked to remove the unwanted commits from the repository history.

---

## 📚 Day 30 Summary

On Day 30 of the DevOps challenge, the Git history of the `media` repository was cleaned by identifying the required `add data.txt file` commit, resetting the `master` branch to that commit using `git reset --hard`, and removing all later test commits from the active history. Since the remote repository still contained the unwanted commits, the rewritten local history was pushed using `git push --force`. Final verification confirmed that both `HEAD -> master` and `origin/master` pointed to the `add data.txt file` commit and that only the required `initial commit` and `add data.txt file` commits remained.
