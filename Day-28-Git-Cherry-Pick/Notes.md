# DevOps Challenge — Day 28 Notes

## Topic: Git Cherry-Pick

Day 28 focused on using **Git cherry-pick** to copy one specific commit from a development branch into another branch without merging all changes from the source branch.

---

## Scenario

Repository:

```text
/opt/beta.git
```

Local clone:

```text
/usr/src/kodekloudrepos/beta
```

Branches:

```text
master
feature
```

The `feature` branch contained work that was still in progress. However, one completed commit with the message:

```text
Update info.txt
```

needed to be added to `master`.

Because only one commit was required, merging the entire `feature` branch would have been inappropriate.

---

## What is `git cherry-pick`?

`git cherry-pick` applies the changes introduced by an existing commit onto the currently checked-out branch.

Syntax:

```bash
git cherry-pick <commit-hash>
```

Example from this task:

```bash
sudo git cherry-pick 8ace519
```

This took the changes from commit `8ace519` on `feature` and applied them to `master`.

---

## Why the Commit Hash Changed

The source commit was:

```text
8ace519 Update info.txt
```

After cherry-picking, the new commit on `master` became:

```text
74f0c3c Update info.txt
```

This is expected.

A Git commit hash is calculated from several pieces of metadata, including:

- File/tree contents
- Parent commit
- Commit message
- Author information
- Committer information
- Timestamp-related metadata

When a commit is cherry-picked onto another branch, its **parent commit changes**, so Git creates a new commit object with a different hash.

The change may be logically the same, but the commit identity is different.

---

## Cherry-Pick vs Merge

### Merge

```bash
git merge feature
```

A merge integrates branch history and can bring multiple commits into the target branch.

This was not appropriate for the challenge because the developer still had unfinished work on `feature`.

### Cherry-Pick

```bash
git cherry-pick <commit>
```

Cherry-pick selects one or more specific commits and applies them independently to the current branch.

For this challenge:

```text
feature:
    ── A ── B ── C
             ↑
       Update info.txt

master:
    ── X ── Y
```

After cherry-picking commit `B`:

```text
master:
    ── X ── Y ── B'
```

`B'` contains the same selected changes as `B`, but it is a new commit with a new hash.

---

## Important Workflow Rule

Always cherry-pick **onto the destination branch**.

Correct sequence:

```bash
sudo git checkout master
sudo git cherry-pick 8ace519
```

If the cherry-pick were run while still on `feature`, the commit would be applied to the wrong branch.

---

## Useful Inspection Commands

### Check repository status

```bash
sudo git status
```

Useful for determining:

- Current branch
- Modified files
- Staged files
- Untracked files
- Whether the working tree is clean

### List local branches

```bash
sudo git branch
```

The `*` indicates the currently checked-out branch.

Example:

```text
  feature
* master
```

### Inspect another branch without switching to it

```bash
sudo git log feature --oneline
```

This is useful because Git allows viewing the history of a named branch directly.

### Search commits by message

```bash
sudo git log feature --oneline --grep="Update info.txt"
```

This is particularly useful when the branch contains many commits.

### Inspect recent history

```bash
sudo git log --oneline -5
```

### Inspect a commit

```bash
sudo git show <commit-hash>
```

Example:

```bash
sudo git show 8ace519
```

---

## Push Operation

After modifying local `master`, the change needed to be sent to the remote repository:

```bash
sudo git push origin master
```

Successful output confirmed:

```text
c775ea5..74f0c3c  master -> master
```

This indicates that remote `master` advanced from `c775ea5` to `74f0c3c`.

---

## Remote Verification

The configured remote was checked with:

```bash
sudo git remote -v
```

Result:

```text
origin  /opt/beta.git (fetch)
origin  /opt/beta.git (push)
```

This confirmed that `origin` pointed to the required bare Git repository.

---

## Working Tree Verification

After the cherry-pick:

```bash
sudo git status
```

reported:

```text
nothing to commit, working tree clean
```

This is an important final validation because it confirms there are no unintended or uncommitted changes remaining in the repository.

---

## What if a Cherry-Pick Conflict Occurs?

A cherry-pick can produce conflicts if the target branch has changes that overlap with the selected commit.

Git will pause the operation.

Check conflicts:

```bash
sudo git status
```

Edit and resolve the conflicting files, then stage them:

```bash
sudo git add <file>
```

Continue:

```bash
sudo git cherry-pick --continue
```

If the cherry-pick should be cancelled:

```bash
sudo git cherry-pick --abort
```

The Day 28 task completed without requiring conflict resolution.

---

## `sudo` Usage

The repository was located under:

```text
/usr/src/kodekloudrepos/beta
```

In the lab environment, elevated permissions may be required for Git operations depending on repository ownership and permissions.

Therefore, commands that modified or inspected the repository were executed with `sudo`, for example:

```bash
sudo git status
sudo git branch
sudo git checkout master
sudo git cherry-pick 8ace519
sudo git push origin master
```

In a normal personal development repository owned by your own user account, Git typically does **not** need `sudo`.

---

## Final Repository State

Local branches:

```text
  feature
* master
```

Recent `master` history:

```text
74f0c3c Update info.txt
c775ea5 Add welcome.txt
2f0dd51 initial commit
```

Remote:

```text
origin  /opt/beta.git
```

The selected change was successfully present on `master` and pushed to `origin/master`.

---

## Key Takeaways

1. Use **merge** when you intend to integrate a branch's work.
2. Use **cherry-pick** when only selected commits should be transferred.
3. Inspect the commit before applying it.
4. Always verify the destination branch before running cherry-pick.
5. A cherry-picked commit normally receives a **new commit hash**.
6. Verify the working tree after the operation.
7. Do not forget to push the updated branch when the task requires remote changes.
8. `git log <branch>` can inspect another branch without checking it out.
9. `git cherry-pick --abort` is useful for safely cancelling a failed cherry-pick.
10. Always verify the final remote branch state in production-style Git tasks.

---

## Day 28 Learning Outcome

This challenge demonstrated how Git can selectively integrate completed work without disturbing unfinished development. Using `git cherry-pick`, a single commit was safely copied from `feature` into `master`, verified locally, and pushed to the remote repository. This workflow is particularly useful for production hotfixes, backports, selective release changes, and situations where only specific commits from a larger development branch are ready for integration.
