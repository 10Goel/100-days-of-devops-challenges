# Commands – Git Pull Request Workflow with Gitea

## SSH Login

Connect to the storage server as Max:

```bash
ssh max@ststor01
```

> Enter Max's password when prompted.

---

## Locate the Repository

Check the current location:

```bash
pwd
```

List files and directories:

```bash
ls -la
```

If the repository location is unknown, search for Git repositories under Max's home directory:

```bash
find ~ -maxdepth 3 -type d -name ".git" 2>/dev/null
```

Then change into the repository directory:

```bash
cd /path/to/repository
```

---

## Inspect Repository Status

```bash
git status
```

---

## Display Local and Remote Branches

```bash
git branch -a
```

---

## Display Git Remotes

```bash
git remote -v
```

---

## View Complete Git History

```bash
git log --oneline --all --decorate --graph
```

---

## Inspect the Feature Branch

```bash
git log --oneline story/fox-and-grapes
```

---

## Show Commits Not Yet in `master`

```bash
git log --oneline master..story/fox-and-grapes
```

---

## Optional: Inspect Remote Feature Branch

```bash
git log --oneline origin/story/fox-and-grapes
```

---

## Gitea Pull Request Configuration

The Pull Request was created in the Gitea UI using:

```text
Title: Added fox-and-grapes story
Pull from: story/fox-and-grapes
Merge into: master
Reviewer: tom
```

The Gitea workflow was:

```text
Create Pull Request
→ Add tom as reviewer
→ Log in as tom
→ Open Files Changed
→ Review changes
→ Approve
→ Merge Pull Request
```

---

## Post-Merge Verification

Fetch the latest remote repository state:

```bash
git fetch origin
```

Display the latest remote `master` history:

```bash
git log --oneline --decorate --graph origin/master -10
```

---

## Check Whether Feature Commits Are Still Outside `master`

```bash
git log --oneline origin/master..origin/story/fox-and-grapes
```

If the branch has been completely merged, no unique unmerged commits should remain.

---

## Show the Latest Commit on Remote `master`

```bash
git log -1 --oneline origin/master
```

---

## Show Detailed Latest Commit Information

```bash
git show --stat origin/master
```

---

## Compare Branch Content

```bash
git diff master..story/fox-and-grapes
```

For remote branches:

```bash
git diff origin/master..origin/story/fox-and-grapes
```

---

## Useful Git Commands for Similar Tasks

### Show current branch

```bash
git branch --show-current
```

### Fetch all updated remote references

```bash
git fetch --all
```

### Show remote branch information

```bash
git remote show origin
```

### Show recent history

```bash
git log --oneline -10
```

### Show branch graph

```bash
git log --oneline --graph --decorate --all
```

### Check whether a branch is merged

```bash
git branch --merged
```

### Check remote branches already merged into `origin/master`

```bash
git branch -r --merged origin/master
```

---

## Important Note About `sudo`

No `sudo` command was required for the Git operations in this challenge.

Avoid:

```bash
sudo git status
sudo git pull
sudo git commit
```

unless there is a specific administrative requirement.

Running normal repository operations with `sudo` can create root-owned Git files and lead to permission issues later.

---

## Final Workflow Summary

```text
ssh max@ststor01
        ↓
Locate repository
        ↓
git status
git branch -a
git remote -v
git log --oneline --all --decorate --graph
        ↓
Verify story/fox-and-grapes
        ↓
Create PR in Gitea
        ↓
Assign Tom
        ↓
Tom reviews and approves
        ↓
Merge into master
        ↓
git fetch origin
        ↓
Verify origin/master
```
