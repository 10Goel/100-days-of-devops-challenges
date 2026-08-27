# DevOps Challenge – Day 30: Commands Reference

## Repository Location

```bash
cd /usr/src/kodekloudrepos/media
```

---

## 1. Check Repository Status

```bash
sudo git status
```

Purpose:
- Confirms the current branch.
- Shows whether the working tree contains modified or untracked files.
- Displays local branch synchronization status with the remote branch.

---

## 2. Check Branch Information

```bash
sudo git branch -vv
```

Purpose:
- Displays local branches.
- Shows the currently checked-out branch.
- Shows upstream tracking information.

To display only the current branch:

```bash
sudo git branch --show-current
```

---

## 3. Check Remote Repository

```bash
sudo git remote -v
```

Purpose:
- Displays configured Git remotes.
- Confirms fetch and push locations for `origin`.

---

## 4. Inspect Commit History

```bash
sudo git log --oneline --decorate --graph
```

Purpose:
- `--oneline` provides compact commit output.
- `--decorate` displays branch and tag references.
- `--graph` visually represents commit relationships.

The target commit identified during the challenge was:

```text
f8befc4 add data.txt file
```

---

## 5. Reset Repository History

```bash
sudo git reset --hard f8befc4
```

Purpose:
- Moves `HEAD` to commit `f8befc4`.
- Moves the current branch pointer to the same commit.
- Resets the index and working tree.
- Removes later commits from the current branch history.

---

## 6. Verify the Reset

```bash
sudo git log --oneline --decorate --graph
```

Expected history:

```text
f8befc4 (HEAD -> master) add data.txt file
291e7a2 initial commit
```

Check the working tree:

```bash
sudo git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

## 7. Force Push the Rewritten History

```bash
sudo git push --force origin master
```

Purpose:
- Updates the remote `master` branch even though the rewritten local branch is no longer a fast-forward continuation of the old remote history.

Expected result includes:

```text
master -> master (forced update)
```

---

## 8. Verify Final History

```bash
sudo git log --oneline
```

Expected:

```text
f8befc4 add data.txt file
291e7a2 initial commit
```

---

## 9. Refresh Remote References

```bash
sudo git fetch origin
```

Then verify all references:

```bash
sudo git log --oneline --decorate --all --graph
```

Expected:

```text
f8befc4 (HEAD -> master, origin/master) add data.txt file
291e7a2 initial commit
```

---

## Complete Command Sequence

```bash
cd /usr/src/kodekloudrepos/media

sudo git status
sudo git branch -vv
sudo git remote -v

sudo git log --oneline --decorate --graph

sudo git reset --hard f8befc4

sudo git log --oneline --decorate --graph
sudo git status

sudo git branch --show-current

sudo git push --force origin master

sudo git log --oneline
sudo git fetch origin
sudo git log --oneline --decorate --all --graph
```

---

## Useful Related Commands

Show the latest five commits:

```bash
sudo git log --oneline -5
```

Show the current `HEAD` commit:

```bash
sudo git rev-parse HEAD
```

Show the remote branch commit:

```bash
sudo git rev-parse origin/master
```

Compare local and remote commit hashes:

```bash
sudo git rev-parse master
sudo git rev-parse origin/master
```
