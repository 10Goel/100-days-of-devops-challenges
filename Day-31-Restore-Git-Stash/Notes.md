# DevOps Day 31 — Notes

## 1. Git Stash

`git stash` temporarily saves uncommitted changes from the working directory so that the developer can work with a clean working tree without committing incomplete work.

Example:

```bash
git stash
```

Git stores each stash as a separate entry.

---

## 2. Viewing Stash Entries

Use:

```bash
git stash list
```

Typical output:

```text
stash@{0}: WIP on master: ...
stash@{1}: WIP on master: ...
```

The index identifies the stash entry.

- `stash@{0}` → most recent stash
- `stash@{1}` → second most recent stash
- `stash@{2}` → third most recent stash

For this challenge, the required entry was:

```text
stash@{1}
```

---

## 3. Applying a Specific Stash

To restore a particular stash:

```bash
git stash apply 'stash@{1}'
```

This restores the changes from that stash into the working directory.

Quoting the stash reference is a good shell practice because the braces can otherwise be interpreted unexpectedly in some environments.

---

## 4. `git stash apply` vs `git stash pop`

### `git stash apply`

```bash
git stash apply 'stash@{1}'
```

- Restores the stash.
- Keeps the stash entry in the stash list.
- Safer when verification is needed before deleting the stash.

### `git stash pop`

```bash
git stash pop 'stash@{1}'
```

- Restores the stash.
- Removes the stash after a successful application.

For this task, `git stash apply` was used because the requirement was only to restore the specified stash.

---

## 5. Checking Restored Changes

After applying a stash:

```bash
git status
```

shows which files were modified or created.

To inspect unstaged modifications:

```bash
git diff
```

This is useful before staging and committing the restored work.

---

## 6. Staging Changes

The restored changes were staged using:

```bash
git add .
```

This adds all current changes under the repository to the staging area.

Always verify before committing:

```bash
git status
```

---

## 7. Creating the Commit

The staged changes were committed with:

```bash
git commit -m "Restore stashed changes"
```

The resulting commit was:

```text
da32f2 Restore stashed changes
```

---

## 8. Checking the Current Branch

The active branch can be checked using:

```bash
git branch --show-current
```

For this challenge:

```text
master
```

This verification is useful before pushing because the correct branch should always be sent to the remote repository.

---

## 9. Pushing to the Remote Repository

The new commit was pushed using:

```bash
git push origin master
```

Where:

- `origin` is the remote repository name.
- `master` is the branch being pushed.

The push completed successfully.

---

## 10. Final Verification

The repository was verified using:

```bash
git status
```

Expected successful state:

```text
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

The recent commit history was also checked with:

```bash
git log --oneline --decorate -3
```

This confirmed that both `master` and `origin/master` pointed to the new commit.

---

## 11. Important Git Workflow

A safe workflow when restoring a specific stash is:

```text
Check repository
      ↓
List stashes
      ↓
Apply required stash
      ↓
Review changes
      ↓
Stage changes
      ↓
Commit
      ↓
Push
      ↓
Verify local and remote state
```

---

## 12. Useful Git Stash Commands

| Command | Purpose |
|---|---|
| `git stash` | Save current uncommitted work |
| `git stash list` | List all stash entries |
| `git stash show` | Show summary of a stash |
| `git stash show -p stash@{n}` | Show full patch of a stash |
| `git stash apply stash@{n}` | Restore a stash without deleting it |
| `git stash pop stash@{n}` | Restore and remove a stash |
| `git stash drop stash@{n}` | Delete a specific stash |
| `git stash clear` | Delete all stash entries |

---

## 💡 Key Takeaways

1. Always run `git stash list` before restoring a stash when multiple entries exist.
2. Use the exact stash identifier specified by the task.
3. Verify changes with `git status` and `git diff` before committing.
4. Confirm the active branch before pushing.
5. A clean working tree after pushing is a strong indication that the local repository is in a consistent state.
6. `git stash apply` is useful when the stash should remain available after restoration.

---

## ✅ Challenge Result

The required `stash@{1}` entry was successfully restored in `/usr/src/kodekloudrepos/apps`, committed as `da32f2 Restore stashed changes`, and pushed to `origin/master`.
