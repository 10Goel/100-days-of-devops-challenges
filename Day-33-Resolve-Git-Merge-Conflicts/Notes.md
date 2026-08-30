# Day 33 — Git Merge Conflict Resolution Notes

## Overview

Day 33 focused on resolving a real-world Git collaboration issue where two developers had modified the same repository independently.

Sarah had already pushed changes to the remote repository while Max had additional local changes. Max's attempt to push was rejected because his local `master` branch did not contain Sarah's latest remote commit.

The correct resolution required pulling the remote changes, resolving the merge conflict, committing the merged state, and pushing the final result back to Gitea.

---

## Why the Initial Push Failed

Git normally allows a push when the remote branch can move directly forward to the local commit. This is called a **fast-forward** update.

In this challenge:

```text
Remote master: A --- B
                   ↑
                Sarah

Local master:  A --- C
                   ↑
                  Max
```

Both Sarah and Max created different commits from the same earlier commit `A`.

When Max attempted:

```bash
git push origin master
```

Git refused because updating the remote branch directly to `C` would discard commit `B`.

This is known as a **non-fast-forward push rejection**.

---

## Correct Merge Workflow

The proper approach is:

```text
Remote: A --- B
          Local:     C
```

After pulling:

```text
A --- B
 \     \
  C ---- M
```

`M` represents the merge commit that combines both contributors' work.

The workflow used was:

```text
git push
   ↓
Push rejected
   ↓
git pull --no-rebase
   ↓
Merge conflict detected
   ↓
Resolve conflict manually
   ↓
git add
   ↓
git commit
   ↓
git push
```

---

## Understanding Merge Conflicts

A merge conflict occurs when Git cannot automatically determine how changes from two branches should be combined.

Git inserts special markers into the conflicting file:

```text
<<<<<<< HEAD
Local version
=======
Remote version
>>>>>>> origin/master
```

Meaning:

- `<<<<<<< HEAD` — beginning of the local version
- `=======` — separator between versions
- `>>>>>>>` — end of the incoming version

These markers must never remain in the final production file.

---

## Conflict Resolution in This Challenge

The conflicting file was:

```text
story-index.txt
```

Both sets of changes had to be preserved.

The final required content was:

```text
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

The typo:

```text
The Lion and the Moose
```

was corrected to:

```text
The Lion and the Mouse
```

---

## Why `git add` Is Required After Resolving a Conflict

After manually modifying a conflicted file, Git still considers it unresolved.

Running:

```bash
git add story-index.txt
```

tells Git:

> The conflict in this file has been resolved and this version should be included in the merge commit.

`git add` therefore serves two purposes:

1. Stage ordinary file changes.
2. Mark conflicted files as resolved.

---

## Why a Merge Commit Was Needed

After all conflicts were resolved and staged, the merge operation still needed to be completed:

```bash
git commit -m "Resolve story index merge conflict"
```

The commit records the final integrated state of Sarah's and Max's work.

---

## Why Force Push Was Avoided

The command:

```bash
git push --force
```

would tell Git to update the remote branch even if remote commits are lost.

In this scenario, that could overwrite Sarah's work.

Force pushes should therefore be used only in carefully controlled situations where rewriting remote history is intentional.

A normal collaborative workflow should prefer:

```bash
git pull
# resolve conflicts
git add
git commit
git push
```

---

## Why `sudo` Was Not Used

Although `sudo` is useful for administrative operations such as modifying system files or managing services, it should generally not be used for Git operations in a user-owned repository.

Avoid:

```bash
sudo git add .
sudo git commit
sudo git pull
```

Using `sudo` may create files inside `.git` owned by `root`, which can later cause permission errors for the normal repository owner.

For this challenge, Max had sufficient permissions inside:

```text
/home/max/story-blog
```

so all Git commands were executed as `max`.

---

## Useful Verification Commands

### Check repository state

```bash
git status
```

### Inspect recent history

```bash
git log --oneline --decorate -5
```

### Verify remote configuration

```bash
git remote -v
```

### Check final file content

```bash
cat story-index.txt
```

### Detect unresolved merge markers

```bash
grep -nE '<<<<<<<|=======|>>>>>>>' story-index.txt
```

### Verify typo removal

```bash
grep -nE 'Moose|Mooose' story-index.txt
```

### Compare local and remote commit hashes

```bash
git rev-parse master
git rev-parse origin/master
```

If both hashes are identical after pushing, the local and remote branch references are synchronized.

---

## Troubleshooting Reference

### Push rejected

Symptom:

```text
! [rejected] master -> master
```

Cause:

Remote branch contains commits missing locally.

Resolution:

```bash
git pull --no-rebase origin master
```

---

### Merge conflict reported

Symptom:

```text
CONFLICT
Automatic merge failed; fix conflicts and then commit the result.
```

Resolution:

1. Open the conflicting file.
2. Remove conflict markers.
3. Combine the required content.
4. Stage the file.
5. Commit the merge.

```bash
git add <file>
git commit
```

---

### Git says there are unmerged paths

Check:

```bash
git status
```

Resolve each file listed under:

```text
Unmerged paths
```

Then stage each resolved file:

```bash
git add <file>
```

---

### Git identity error

Possible error:

```text
Author identity unknown
```

Configure the repository identity:

```bash
git config user.name "max"
git config user.email "max@stratos.xfusioncorp.com"
```

---

## Final Validation Checklist

- [x] Connected to the storage server as Max
- [x] Opened `/home/max/story-blog`
- [x] Confirmed the `master` branch
- [x] Reproduced the push rejection
- [x] Pulled the latest remote changes
- [x] Identified the conflict in `story-index.txt`
- [x] Preserved all four story entries
- [x] Corrected `Moose` to `Mouse`
- [x] Removed all conflict markers
- [x] Staged the resolved file
- [x] Committed the merge resolution
- [x] Pushed successfully to `origin/master`
- [x] Verified a clean Git working tree
- [x] Verified the final file in Gitea

---

## Key Takeaway

A rejected Git push in a shared repository should not immediately lead to a force push.

The safe collaborative pattern is:

```text
FETCH/PULL → MERGE → RESOLVE → STAGE → COMMIT → PUSH → VERIFY
```

This protects other developers' work while producing a consistent repository history.

---

## Challenge Status

**Day 33 — Completed Successfully ✅**
