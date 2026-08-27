# Day 27 Notes – Git Revert

## Challenge Summary

The objective of Day 27 was to safely undo the latest commit in the Git repository:

```bash
/usr/src/kodekloudrepos/games
```

The repository was hosted on the Storage Server in Stratos DC.

The task specifically required:

- Reverting the current `HEAD` commit.
- Confirming that `HEAD^` had the message `initial commit`.
- Preserving the original Git history.
- Creating a new commit with the exact message:

  ```text
  revert games
  ```

---

## 1. Understanding `HEAD`

In Git, `HEAD` points to the commit currently checked out.

Example:

```text
A --- B --- C
          HEAD
```

Here:

- `HEAD` points to commit `C`.
- `HEAD^` refers to the first parent of `HEAD`, which is commit `B`.
- `HEAD~1` is equivalent to `HEAD^` in a simple linear history.

Useful commands:

```bash
sudo git log -1 --format="%s" HEAD
sudo git log -1 --format="%s" HEAD^
```

---

## 2. What Does `git revert` Do?

`git revert` does not delete an existing commit.

Instead, it calculates the inverse of the selected commit and records that inverse as a new commit.

Before:

```text
A --- B --- C
          HEAD
```

After reverting `C`:

```text
A --- B --- C --- D
                    HEAD
```

Commit `D` reverses the file changes introduced by `C`.

This preserves the full history.

---

## 3. Why `git revert` Was Correct for This Task

The requirement was to **revert the latest commit**, not remove it from history.

Therefore:

```bash
git revert
```

was appropriate.

Using:

```bash
git reset --hard HEAD^
```

would instead move the branch pointer backward and remove the latest commit from the visible branch history. That can be dangerous in shared repositories.

---

## 4. Why `--no-commit` Was Used

Normally:

```bash
sudo git revert HEAD
```

creates the revert commit and generates a default message such as:

```text
Revert "add data.txt file"
```

However, this challenge required the exact message:

```text
revert games
```

Therefore the revert was applied without creating a commit:

```bash
sudo git revert --no-commit HEAD
```

Then the staged inverse changes were committed manually:

```bash
sudo git commit -m "revert games"
```

---

## 5. Checking the Repository Before Reverting

Before changing history, inspect the working tree:

```bash
sudo git status
```

Then inspect recent commits:

```bash
sudo git log --oneline -3
```

To verify the parent commit message:

```bash
sudo git log -1 --format="%s" HEAD^
```

Expected:

```text
initial commit
```

---

## 6. Staging Area During `git revert --no-commit`

After:

```bash
sudo git revert --no-commit HEAD
```

Git applies the reverse patch and usually stages the affected tracked files.

The repository can then be inspected with:

```bash
sudo git status
```

The revert changes should appear under:

```text
Changes to be committed:
```

---

## 7. Untracked Files

During the challenge, `git status` also showed an untracked file:

```text
games.txt
```

An untracked file is not automatically part of a commit.

Unless the task explicitly requires it, it should not be added with:

```bash
git add
```

The revert commit should contain only the intended inverse changes.

---

## 8. Final Verification

The resulting history was:

```text
aab77cd revert games
fa3e970 add data.txt file
61c3e3b initial commit
```

This proves that:

1. The original latest commit still exists.
2. A new revert commit was created.
3. Git history was preserved.
4. The new commit message is exactly `revert games`.

The latest commit message can be checked directly:

```bash
sudo git log -1 --format="%s"
```

Expected:

```text
revert games
```

---

## 9. `git revert` vs `git reset`

| Feature | `git revert` | `git reset` |
|---|---|---|
| Creates a new commit | Yes | No |
| Preserves existing history | Yes | Can rewrite history |
| Suitable for shared branches | Generally yes | Use carefully |
| Safely undo a published commit | Yes | Usually not preferred |
| Moves branch pointer backward | No | Yes |

For collaborative repositories, `git revert` is generally the safer method for undoing already-committed changes.

---

## 10. Important Commands Learned

Check status:

```bash
sudo git status
```

View recent history:

```bash
sudo git log --oneline -3
```

Check the previous commit message:

```bash
sudo git log -1 --format="%s" HEAD^
```

Apply a revert without committing:

```bash
sudo git revert --no-commit HEAD
```

Create the required commit:

```bash
sudo git commit -m "revert games"
```

Verify the latest commit:

```bash
sudo git log -1 --format="%s"
```

---

## Key Takeaways

- `HEAD` refers to the current commit.
- `HEAD^` refers to its parent commit.
- `git revert` safely undoes a commit while preserving history.
- `git revert --no-commit` is useful when you need control over the resulting commit.
- Always inspect `git status` before and after Git operations.
- Do not accidentally add unrelated untracked files.
- Commit messages must match task requirements exactly when automated validation is involved.
- `sudo` may be needed for repositories owned by another account or located in protected paths.

---

## Challenge Result

**Day 27 completed successfully ✅**
