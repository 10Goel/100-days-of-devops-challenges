# DevOps Day 31 — Commands

## Repository

```bash
cd /usr/src/kodekloudrepos/apps
```

---

## 1. Check Repository Status

```bash
sudo git status
```

---

## 2. Check Current Branch

```bash
sudo git branch --show-current
```

Result:

```text
master
```

---

## 3. List Available Stashes

```bash
sudo git stash list
```

The task required restoring:

```text
stash@{1}
```

---

## 4. Restore the Required Stash

```bash
sudo git stash apply 'stash@{1}'
```

---

## 5. Verify Restored Changes

```bash
sudo git status
```

```bash
sudo git diff
```

---

## 6. Stage the Changes

```bash
sudo git add .
```

Optional verification:

```bash
sudo git status
```

---

## 7. Commit the Restored Changes

```bash
sudo git commit -m "Restore stashed changes"
```

Commit created:

```text
da32f2 Restore stashed changes
```

---

## 8. Verify the Latest Commit

```bash
sudo git log -1 --oneline
```

---

## 9. Reconfirm the Branch

```bash
sudo git branch --show-current
```

Result:

```text
master
```

---

## 10. Push to the Remote Repository

```bash
sudo git push origin master
```

Successful push:

```text
master -> master
```

---

## 11. Final Repository Verification

```bash
sudo git status
```

Expected:

```text
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

---

## 12. Verify Recent Commit History

```bash
sudo git log --oneline --decorate -3
```

Latest commit:

```text
da32f2 (HEAD -> master, origin/master) Restore stashed changes
```

---

## Complete Command Sequence

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

## Useful Optional Commands

Inspect a stash before restoring it:

```bash
sudo git stash show -p 'stash@{1}'
```

Check configured remotes:

```bash
sudo git remote -v
```

Check synchronization with upstream:

```bash
sudo git status
```

---

## ✅ Final Result

The changes from `stash@{1}` were restored successfully, committed, and pushed to `origin/master`.
