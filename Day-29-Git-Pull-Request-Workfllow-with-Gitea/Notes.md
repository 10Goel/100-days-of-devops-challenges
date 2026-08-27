# Notes – Git Pull Request Workflow with Gitea

## 1. Pull Request Concept

A **Pull Request (PR)** is a request to merge changes from one Git branch into another.

In this challenge:

```text
Source Branch: story/fox-and-grapes
Destination Branch: master
```

This means the commits from `story/fox-and-grapes` were proposed for inclusion in `master`.

---

## 2. Why Use a Feature Branch?

Instead of developing directly on `master`, changes should normally be made on a dedicated branch.

Example:

```text
master
  │
  └── story/fox-and-grapes
```

Advantages:

- Keeps the main branch stable.
- Isolates development work.
- Makes review easier.
- Supports multiple developers working simultaneously.
- Makes rollback and troubleshooting simpler.

---

## 3. Source Branch vs Destination Branch

When creating a Pull Request, understanding the direction is critical.

```text
story/fox-and-grapes  →  master
```

- **Source / pull-from branch:** contains the new changes.
- **Destination / merge-into branch:** receives the approved changes.

Reversing these branches would create the wrong Pull Request.

---

## 4. Repository Verification

Before creating a PR, inspect the repository.

### Check working tree

```bash
git status
```

Shows whether the current working tree has modified, staged, or untracked files.

### Show branches

```bash
git branch -a
```

Displays local and remote-tracking branches.

### Show configured remotes

```bash
git remote -v
```

Displays fetch and push URLs for the configured Git remotes.

### Inspect complete history

```bash
git log --oneline --all --decorate --graph
```

Useful options:

- `--oneline` → compact commit representation.
- `--all` → include all branches.
- `--decorate` → show branch and tag references.
- `--graph` → display branch topology visually.

---

## 5. Comparing Branches

A useful command for checking changes not yet present in `master` is:

```bash
git log --oneline master..story/fox-and-grapes
```

Meaning:

> Show commits reachable from `story/fox-and-grapes` that are not reachable from `master`.

This is useful before opening a Pull Request.

---

## 6. Reviewer Role

A reviewer checks the proposed changes before they are merged.

In this challenge:

```text
PR Author: Max
Reviewer: Tom
```

Tom's responsibilities were:

1. Open the Pull Request.
2. Inspect the changed file(s).
3. Confirm the changes were acceptable.
4. Approve the Pull Request.
5. Merge the approved changes.

---

## 7. Approval vs Merge

Approval and merge are two separate actions.

### Approval

Approval means:

> The reviewer has examined the change and accepts it.

It does **not** itself modify the `master` branch.

### Merge

Merge means:

> Integrate the approved changes into the destination branch.

Therefore:

```text
Review → Approve → Merge
```

is the correct sequence.

---

## 8. Merge Commit

The challenge used a merge operation that created a merge commit.

A merge commit preserves the relationship between the feature branch and the destination branch.

Typical history:

```text
*   Merge pull request 'Added fox-and-grapes story'
|\
| * Added fox-and-grapes story
|/
* Previous master commit
```

This provides clear historical evidence that the change entered `master` through a Pull Request.

---

## 9. Gitea

**Gitea** is a lightweight, self-hosted Git service similar in purpose to GitHub, GitLab, and Bitbucket.

It provides features such as:

- Repository hosting
- Pull Requests
- Issues
- Code review
- User management
- Branch management
- Releases
- Actions / CI workflows
- Access control

---

## 10. Why `sudo` Was Not Needed

Git commands in this task were executed as the repository-owning user.

Using:

```bash
sudo git ...
```

was unnecessary and could create files owned by `root`, which may later cause permission problems.

Use `sudo` only when administrative permissions are genuinely required, such as modifying system configuration or managing protected system services.

---

## 11. Remote Verification After Merge

After merging in Gitea, the local repository may not automatically know about the latest remote commit.

Use:

```bash
git fetch origin
```

Then verify:

```bash
git log --oneline --decorate --graph origin/master -10
```

`git fetch` downloads updated remote references without modifying the current working branch.

---

## 12. Pull Request Best Practices

A professional PR workflow generally includes:

- Create a dedicated branch.
- Keep changes focused.
- Use meaningful commit messages.
- Push the branch to the remote repository.
- Open a Pull Request.
- Provide a clear title and description.
- Request review.
- Address reviewer feedback.
- Obtain approval.
- Merge only after validation.
- Delete the feature branch when it is no longer needed.

---

## 13. Branch Protection

The challenge illustrates the idea behind **branch protection**.

A protected main branch can be configured to require:

- Pull Requests before merging.
- One or more reviewer approvals.
- Successful CI/CD checks.
- Signed commits.
- Up-to-date branches.
- Restrictions on direct pushes.

Branch protection reduces the risk of unreviewed or broken code reaching critical branches.

---

## 14. Important Difference: `git fetch` vs `git pull`

### `git fetch`

```bash
git fetch origin
```

Downloads remote changes but does not merge them into the current branch.

### `git pull`

```bash
git pull origin master
```

Fetches remote changes and then integrates them into the current branch.

For simple post-merge verification, `git fetch` is safer because it does not alter the current local branch.

---

## 15. Key Takeaway

The core workflow practiced in this task was:

```text
Create Change
     ↓
Feature Branch
     ↓
Push to Remote
     ↓
Open Pull Request
     ↓
Peer Review
     ↓
Approval
     ↓
Merge
     ↓
master
```

This workflow is fundamental to collaborative software development and modern DevOps practices.
