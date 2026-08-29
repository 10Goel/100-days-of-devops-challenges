# Day 32 — Git Rebase: Rebase `feature` onto `master`

## Overview

This challenge focused on using **Git rebase** to update an in-progress `feature` branch with the latest changes from `master` while preserving all feature work and avoiding an unnecessary merge commit.

The repository was already cloned on the storage server at:

```text
/usr/src/kodekloudrepos/apps
```

The remote repository was:

```text
/opt/apps.git
```

The required outcome was to rebase `feature` onto `master`, verify a clean linear history, and push the rebased branch back to the remote repository.

---

## Task Requirements

- Work with the repository cloned at `/usr/src/kodekloudrepos/apps`.
- Preserve all work already present on the `feature` branch.
- Incorporate the latest changes from `master`.
- **Do not create a merge commit.**
- Rebase the `feature` branch onto `master`.
- Push the updated `feature` branch to the remote repository.
- Use elevated permissions where required.

---

## Environment

| Component | Value |
|---|---|
| Server | Storage Server |
| User | `natasha` |
| Repository | `/usr/src/kodekloudrepos/apps` |
| Remote | `/opt/apps.git` |
| Base Branch | `master` |
| Working Branch | `feature` |

---

## Solution Approach

### 1. Enter the repository

```bash
cd /usr/src/kodekloudrepos/apps
```

### 2. Inspect repository state

```bash
sudo git status
sudo git branch -a
sudo git remote -v
```

This confirmed the repository state, available branches, and remote configuration before making changes.

### 3. Fetch the latest remote state

```bash
sudo git fetch origin
```

Fetching first ensured that the local repository had the latest remote branch references.

### 4. Update `master`

```bash
sudo git checkout master
sudo git pull --ff-only origin master
```

Using `--ff-only` ensured that updating `master` did not create an accidental merge commit.

### 5. Switch to `feature`

```bash
sudo git checkout feature
sudo git status
```

### 6. Rebase `feature` onto `master`

```bash
sudo git rebase master
```

Git replayed the feature commit(s) on top of the latest `master` commit, producing a clean linear history without adding a merge commit.

### 7. Verify the history

```bash
sudo git log --oneline --graph --decorate --all
```

Final verified history:

```text
* bbdfda0 (HEAD -> feature, origin/feature) Add new feature
* b995b83 (origin/master, origin/HEAD, master) Update info.txt
* 7c669da initial commit
```

This confirms that `feature` now sits directly on top of `master`.

### 8. Push the rebased branch

The first normal push was attempted:

```bash
sudo git push origin feature
```

Because rebase rewrites commit history, the remote branch required a history update. The branch was safely pushed with:

```bash
sudo git push --force-with-lease origin feature
```

`--force-with-lease` is preferred over plain `--force` because it refuses to overwrite unexpected remote changes.

### 9. Final verification

```bash
sudo git status
sudo git log --oneline --graph --decorate --all
sudo git ls-remote --heads origin
```

The final working tree was clean, and both remote branches were verified.

---

## Final Result

The challenge was completed successfully.

- `feature` was rebased onto the latest `master`.
- No merge commit was created.
- Existing feature work was preserved.
- The history became linear.
- The rebased branch was pushed successfully using `--force-with-lease`.
- The local and remote `feature` branch both pointed to commit `bbdfda0`.

---

## Final Git History

```text
feature / origin/feature
        |
        v
bbdfda0  Add new feature
   |
   v
b995b83  Update info.txt
   |
   v
7c669da  initial commit
```

This is the expected rebase result: the feature work is replayed on top of the updated `master` branch without introducing a merge commit.

---

## Key Learning

A **merge** combines branch histories and may create an additional merge commit, while a **rebase** moves/replays commits onto a new base and produces a cleaner linear history.

For already-published branches, rebasing changes commit hashes. Therefore, a normal push can be rejected as non-fast-forward. In such cases, `git push --force-with-lease` is safer than `git push --force`.

---

## Challenge Status

**Day 32: Completed Successfully ✅**
