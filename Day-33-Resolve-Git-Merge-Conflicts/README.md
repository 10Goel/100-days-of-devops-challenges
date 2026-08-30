# Day 33 — Resolve Git Merge Conflicts

## 100 Days of DevOps Challenge

### Objective

The goal of Day 33 was to troubleshoot and resolve a Git push failure caused by conflicting changes made by two contributors, **Sarah** and **Max**, in the same repository.

Max had local changes in the `story-blog` repository that needed to be pushed to the remote repository. However, Sarah had already pushed other changes to the remote `master` branch. This caused Max's push to be rejected because his local branch was behind the remote branch.

The task required preserving the changes from both contributors, resolving the merge conflict in `story-index.txt`, correcting the typo **"The Lion and the Moose" → "The Lion and the Mouse"**, and successfully pushing the final merged version to the remote repository.

---

## Environment

| Component | Details |
|---|---|
| Server | Storage Server |
| User | `max` |
| Repository | `/home/max/story-blog` |
| Remote | Gitea repository |
| Branch | `master` |
| Conflicting File | `story-index.txt` |
| Version Control | Git |

---

## Problem

A normal push from Max's local repository failed because the remote `master` branch contained commits that were not available in Max's local branch.

Typical error:

```text
! [rejected] master -> master (fetch first)
error: failed to push some refs
```

This happens when the remote repository has new commits and Git prevents the local branch from overwriting those changes.

The correct solution is **not** to force push. Instead, the latest remote changes must be integrated into the local branch, conflicts must be resolved manually, and the merged result must then be pushed normally.

---

## Resolution

### 1. Connected to the storage server

```bash
ssh max@ststor01
```

### 2. Navigated to the repository

```bash
cd /home/max/story-blog
```

### 3. Verified repository state

```bash
git status
git branch
git log --oneline --decorate -5
git remote -v
```

### 4. Reproduced the push failure

```bash
git push origin master
```

The push was rejected because the remote repository contained newer commits.

### 5. Pulled the remote changes

```bash
git pull --no-rebase origin master
```

Git attempted to merge the remote changes with Max's local changes and reported a conflict in:

```text
story-index.txt
```

### 6. Resolved the merge conflict

The conflict markers were removed and the file was updated so that all four stories were retained.

Final content:

```text
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

The typo was also corrected:

```text
The Lion and the Moose
```

to:

```text
The Lion and the Mouse
```

### 7. Staged the resolved file

```bash
git add story-index.txt
```

### 8. Committed the merge resolution

```bash
git commit -m "Resolve story index merge conflict"
```

### 9. Pushed the final changes

```bash
git push origin master
```

The push completed successfully.

---

## Verification

The repository was verified both from the terminal and through the **Gitea UI**.

### Final Git status

```bash
git status
```

Expected state:

```text
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Final story index

```bash
cat story-index.txt
```

Output:

```text
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

### Conflict-marker check

```bash
grep -nE '<<<<<<<|=======|>>>>>>>' story-index.txt
```

No output confirms that all Git conflict markers were removed.

### Remote verification

The `master` branch was opened in Gitea and `story-index.txt` was verified successfully. The final merge-resolution commit was visible in the repository history.

---

## Key Concepts Learned

- Understanding why Git rejects non-fast-forward pushes
- Safely integrating remote changes before pushing
- Difference between `git pull`, merge, and force pushing
- Identifying Git conflict markers
- Manually resolving merge conflicts
- Preserving changes from multiple contributors
- Staging resolved files with `git add`
- Completing a merge with `git commit`
- Verifying local and remote branch synchronization
- Using Gitea to validate repository contents
- Avoiding unnecessary `sudo` usage with Git repositories

---

## Important DevOps Practice

A force push was intentionally avoided.

```bash
git push --force
```

can overwrite remote history and potentially destroy another contributor's work.

The safe workflow used in this challenge was:

```text
Push rejected
     ↓
Pull latest remote changes
     ↓
Git detects merge conflict
     ↓
Resolve conflict manually
     ↓
Stage resolved file
     ↓
Commit merge resolution
     ↓
Push normally
     ↓
Verify in remote repository
```

---

## Result

**Day 33 completed successfully.**

The Git merge conflict was resolved correctly, all four story entries were preserved, the typo was fixed, the merged changes were committed, and the final `master` branch was successfully pushed and verified in Gitea.
