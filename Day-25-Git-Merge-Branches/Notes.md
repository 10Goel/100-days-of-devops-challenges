# Day 25 - Notes

## Objective

The main objective of this challenge was to practice a complete Git branching workflow in an existing repository. The workflow included creating a feature branch, adding a new file, committing the change, merging the branch back into the main branch, and synchronizing both branches with the remote repository.

## Repository Details

| Item | Value |
|---|---|
| Repository Path | `/usr/src/kodekloudrepos/news` |
| Remote Repository | `/opt/news.git` |
| Main Branch | `master` |
| New Branch | `nautilus` |
| Source File | `/tmp/index.html` |
| Destination File | `index.html` |

## Git Safe Directory Issue

While accessing the repository as the `natasha` user, Git reported:

```text
fatal: detected dubious ownership in repository
```

This happens when Git detects that the current user does not own the repository directory. It is a security feature designed to prevent users from unknowingly running Git operations inside repositories owned by another user.

The repository was explicitly marked as trusted with:

```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/news
```

This allowed Git commands such as `git status`, `git branch`, and `git remote -v` to run successfully.

## Permission Issue

Even after adding the safe-directory exception, normal Git write operations failed with permission errors such as:

```text
Permission denied
```

This occurred because creating branches, lock files, commits, and updating Git references requires write access to the `.git` directory.

The required Git write operations were therefore performed with `sudo`.

## Branch Creation

The new branch was created from `master`:

```bash
sudo git checkout master
sudo git checkout -b nautilus
```

At this point, `nautilus` initially pointed to the same commit as `master`.

## Adding the File

The required HTML file was copied from:

```text
/tmp/index.html
```

into the repository root:

```bash
sudo cp /tmp/index.html .
```

The file was then staged:

```bash
sudo git add index.html
```

and committed:

```bash
sudo git commit -m "Add index.html"
```

This commit existed on the `nautilus` branch before the merge.

## Merge Behavior

After returning to `master`:

```bash
sudo git checkout master
```

the branch was merged:

```bash
sudo git merge nautilus
```

Git performed a **fast-forward merge**.

### What is a Fast-Forward Merge?

A fast-forward merge occurs when the target branch has not received any new commits since the feature branch was created.

Before merge:

```text
master
  |
  A
   \
    B  <- nautilus
```

After merge:

```text
A --- B
      ^
      |
 master
 nautilus
```

Git simply moves the `master` branch pointer forward to the commit already present on `nautilus`.

No separate merge commit is required.

## Push Requirement

The task specifically required both branches to be pushed to the origin.

The commands used were:

```bash
sudo git push origin nautilus
sudo git push origin master
```

Pushing only `master` would not have fully satisfied the task because the remote repository also needed the `nautilus` branch.

## Useful Verification Commands

### Check repository status

```bash
sudo git status
```

### Display local branches

```bash
sudo git branch
```

### Display commit history

```bash
sudo git log --oneline --decorate --graph --all
```

### Verify remote branches

```bash
sudo git ls-remote --heads origin
```

Expected remote branches:

```text
refs/heads/master
refs/heads/nautilus
```

## Important Lessons

1. Git's safe-directory feature and filesystem permissions are separate concepts.
2. A repository can be marked safe while the current user still lacks permission to modify it.
3. Feature branches should be created from the correct base branch.
4. Files should be staged before committing.
5. A fast-forward merge is valid and does not always create a new merge commit.
6. When a task explicitly asks for multiple branches to be pushed, each required branch must exist on the remote.
7. Verification commands should always be used before considering a Git task complete.

## Final Outcome

The challenge was completed successfully after:

- trusting the repository with `safe.directory`,
- using the necessary privileges for Git write operations,
- creating and committing changes on `nautilus`,
- merging into `master`,
- and pushing both branches to the remote repository.
