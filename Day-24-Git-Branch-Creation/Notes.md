# DevOps Day 24 — Notes

## Topic

Git Branch Creation and Branch Management

## Objective

Create a new Git branch named `xfusioncorp_games` from the `master` branch in the repository:

```bash
/usr/src/kodekloudrepos/games
```

No application code changes were required.

## What Is a Git Branch?

A Git branch is a movable pointer to a commit in the repository history. Branches allow developers to work on features, fixes, or experiments independently without immediately affecting another branch such as `master`.

For example:

```text
A --- B --- C   master
          \
           C    xfusioncorp_games
```

Immediately after creating the new branch from `master`, both branch names point to the same commit.

## Creating a Branch

A branch can be created using:

```bash
git branch <branch-name>
```

This only creates the branch.

To create and immediately switch to it:

```bash
git checkout -b <branch-name>
```

For this challenge:

```bash
sudo git checkout -b xfusioncorp_games
```

Modern Git also supports:

```bash
git switch -c xfusioncorp_games
```

## Why Switch to `master` First?

The requirement explicitly stated that `xfusioncorp_games` must be created from the `master` branch.

Therefore, it is good practice to explicitly switch to `master` before creating the new branch:

```bash
sudo git checkout master
```

Then:

```bash
sudo git checkout -b xfusioncorp_games
```

This avoids accidentally creating the branch from another existing branch.

## Important Git Commands

### Check Repository Status

```bash
git status
```

Shows:

- Current branch
- Modified files
- Staged files
- Untracked files

### List Local Branches

```bash
git branch
```

The active branch is marked with `*`.

Example:

```text
  master
* xfusioncorp_games
```

### Switch Branches

```bash
git checkout master
```

### Create and Switch to a New Branch

```bash
git checkout -b xfusioncorp_games
```

### Check Branch Commit

```bash
git rev-parse master
git rev-parse xfusioncorp_games
```

If both commit hashes are identical immediately after branch creation, the new branch has been created from the expected commit.

## Why No Commit Was Required

The challenge only required creating a branch. No file changes were requested.

Therefore, the following commands were unnecessary:

```bash
git add
git commit
git push
```

Running additional commands that modify repository history can cause validation failures when a task specifically requests only branch creation.

## Verification Checklist

- Repository path is correct.
- `master` exists.
- `xfusioncorp_games` exists.
- New branch was created from `master`.
- Working tree remains clean.
- No code was modified.
- No commit was created.

## Learning Outcome

This challenge demonstrates one of the most common Git workflows used by DevOps and software engineering teams: creating an isolated branch from a known stable branch before implementing new work.
