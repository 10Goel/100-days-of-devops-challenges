# Day 32 — Commands Reference

## Repository Location

```bash
cd /usr/src/kodekloudrepos/apps
```

---

## Initial Inspection

```bash
sudo git status
sudo git branch -a
sudo git remote -v
```

### Purpose

- `git status` — verifies the current branch and working-tree state.
- `git branch -a` — displays local and remote-tracking branches.
- `git remote -v` — confirms the configured remote repository.

---

## Fetch Latest Remote References

```bash
sudo git fetch origin
```

This downloads the latest remote metadata and updates remote-tracking references without modifying the working branch.

---

## Update the `master` Branch

```bash
sudo git checkout master
sudo git pull --ff-only origin master
sudo git status
```

### Why `--ff-only`?

It allows the pull only when Git can fast-forward the branch. This prevents the update operation from creating an unexpected merge commit.

---

## Switch to the Feature Branch

```bash
sudo git checkout feature
sudo git status
```

---

## Rebase `feature` onto `master`

```bash
sudo git rebase master
```

This is the main command required by the challenge.

It takes the commit(s) unique to `feature` and replays them on top of the latest `master`.

---

## Check for Stashes

```bash
sudo git stash list
```

This was used as a safety check to verify whether any previously stashed work needed restoration.

If a stash had existed, it could be restored with:

```bash
sudo git stash pop
```

---

## Verify Rebased History

```bash
sudo git log --oneline --graph --decorate --all
```

Verified result:

```text
* bbdfda0 (HEAD -> feature, origin/feature) Add new feature
* b995b83 (origin/master, origin/HEAD, master) Update info.txt
* 7c669da initial commit
```

---

## Push the Feature Branch

First attempt:

```bash
sudo git push origin feature
```

After the non-fast-forward condition caused by the rewritten rebase history, the safe force push was used:

```bash
sudo git push --force-with-lease origin feature
```

---

## Final Verification

```bash
sudo git status
sudo git log --oneline --graph --decorate --all
sudo git ls-remote --heads origin
```

Verified remote references included:

```text
refs/heads/feature
refs/heads/master
```

---

## Complete Command Sequence

```bash
cd /usr/src/kodekloudrepos/apps

sudo git status
sudo git branch -a
sudo git remote -v

sudo git fetch origin

sudo git checkout master
sudo git pull --ff-only origin master
sudo git status

sudo git checkout feature
sudo git status

sudo git rebase master

sudo git stash list
sudo git log --oneline --graph --decorate --all
sudo git status

sudo git push origin feature
sudo git push --force-with-lease origin feature

sudo git status
sudo git log --oneline --graph --decorate --all
sudo git ls-remote --heads origin
```

---

## Conflict Resolution Commands

If a rebase conflict occurs in a similar task:

```bash
sudo git status
```

Edit and resolve the conflicting file, then stage it:

```bash
sudo git add <file>
```

Continue the rebase:

```bash
sudo git rebase --continue
```

Repeat until the rebase completes.

To abandon the rebase and return to the original state:

```bash
sudo git rebase --abort
```

---

## Important Commands to Avoid for This Task

Do not solve this challenge with:

```bash
sudo git merge master
```

That approach can create a merge commit and does not satisfy the requirement to rebase the feature branch.

Also avoid using plain force push when `--force-with-lease` is sufficient:

```bash
sudo git push --force origin feature
```

Prefer:

```bash
sudo git push --force-with-lease origin feature
```
