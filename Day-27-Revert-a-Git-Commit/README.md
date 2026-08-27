# DevOps Challenge – Day 27: Revert a Git Commit

## Overview

In this challenge, the Nautilus application development team reported an issue with the most recent commit in the Git repository located at:

```bash
/usr/src/kodekloudrepos/games
```

on the **Storage Server** in **Stratos DC**.

The requirement was to **revert the latest commit (`HEAD`) without rewriting Git history**. The commit immediately before `HEAD` had the message `initial commit`, and the new revert commit had to use the exact commit message:

```text
revert games
```

The task was completed successfully using `git revert`.

---

## Task Requirements

- Work with the repository:

  ```bash
  /usr/src/kodekloudrepos/games
  ```

- Verify that the commit immediately before `HEAD` is:

  ```text
  initial commit
  ```

- Revert the latest commit.
- Preserve Git history rather than deleting the problematic commit.
- Create the new revert commit with the exact lowercase message:

  ```text
  revert games
  ```

- Use `sudo` where elevated permissions are required.
- Verify the resulting commit history.

---

## Solution Approach

### 1. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/games
```

### 2. Check Repository Status

```bash
sudo git status
```

This confirms the current working-tree state before making changes.

### 3. Inspect Recent Commit History

```bash
sudo git log --oneline -3
```

The history confirmed that the previous commit had the required message:

```text
initial commit
```

For direct verification:

```bash
sudo git log -1 --format="%s" HEAD^
```

### 4. Revert the Latest Commit

The latest commit was reverted without automatically creating the revert commit:

```bash
sudo git revert --no-commit HEAD
```

Using `--no-commit` applies the inverse changes to the index and working tree while allowing the required custom commit message to be supplied manually.

### 5. Verify the Staged Revert Changes

```bash
sudo git status
```

The revert changes were staged and ready to commit.

### 6. Create the Required Revert Commit

```bash
sudo git commit -m "revert games"
```

This created the new commit with the exact task-required message.

### 7. Verify the Final Commit History

```bash
sudo git log --oneline -3
```

The final history showed:

```text
aab77cd revert games
fa3e970 add data.txt file
61c3e3b initial commit
```

The exact `HEAD` commit message was also verified with:

```bash
sudo git log -1 --format="%s"
```

Output:

```text
revert games
```

---

## Final Result

The challenge was completed successfully.

The repository history retained the original problematic commit and added a new commit that reversed its changes:

```text
aab77cd revert games
fa3e970 add data.txt file
61c3e3b initial commit
```

This is the correct use of `git revert`, because it safely preserves commit history while undoing the effect of an earlier commit.

---

## Key Git Concept

### `git revert` vs `git reset`

`git revert` creates a **new commit** containing the inverse of the selected commit.

```text
A --- B --- C
          HEAD
```

After reverting `C`:

```text
A --- B --- C --- D
                    HEAD
```

Where `D` reverses the changes introduced by `C`.

This makes `git revert` suitable for shared repositories because existing history is preserved.

By contrast, commands such as `git reset --hard HEAD^` move the branch pointer backward and can rewrite history, which was not appropriate for this task.

---

## Skills Practiced

- Git repository inspection
- Git commit history analysis
- `HEAD` and `HEAD^`
- Safe commit reversal using `git revert`
- Custom revert commit messages
- Working-tree and staging-area verification
- Git history validation
- Linux permission handling with `sudo`

---
## Challenge Status

**Completed Successfully ✅**

**Day 27 – Git Revert Challenge**
