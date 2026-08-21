# DevOps Day 24 — Git Branch Creation

## Overview

This challenge focused on Git branch management in an existing project repository on the Storage Server. The requirement was to create a new branch named `xfusioncorp_games` from the `master` branch inside the `/usr/src/kodekloudrepos/games` repository without making any changes to the application code.

## Task Requirements

- Connect to the **Storage Server** in Stratos DC.
- Navigate to the Git repository:
  ```bash
  /usr/src/kodekloudrepos/games
  ```
- Create a new branch named:
  ```bash
  xfusioncorp_games
  ```
- Ensure the new branch is created from:
  ```bash
  master
  ```
- Do not modify, add, delete, or commit any code.

## Implementation

First, the Storage Server was accessed from the jump host.

```bash
ssh natasha@ststor01
```

The target repository was opened:

```bash
cd /usr/src/kodekloudrepos/games
```

The current Git status and available branches were checked:

```bash
sudo git status
sudo git branch
```

The `master` branch was selected to ensure that the new branch would be created from the correct base:

```bash
sudo git checkout master
```

The required branch was then created:

```bash
sudo git checkout -b xfusioncorp_games
```

## Verification

The branch list was checked:

```bash
sudo git branch
```

Expected result:

```text
  master
* xfusioncorp_games
```

The repository status was also verified:

```bash
sudo git status
```

Expected result:

```text
On branch xfusioncorp_games
nothing to commit, working tree clean
```

To confirm that the new branch was created from the same commit as `master`, the commit hashes can be compared:

```bash
sudo git rev-parse master
sudo git rev-parse xfusioncorp_games
```

Both branches should initially point to the same commit.

## Result

The `xfusioncorp_games` branch was successfully created from the `master` branch without modifying the repository contents.

**Status:** ✅ Challenge completed successfully.

## Key Takeaways

- Git branches are lightweight pointers to commits.
- Creating a branch from `master` makes the new branch initially reference the same commit.
- `git checkout -b <branch>` creates and switches to a new branch in one command.
- Repository state should always be verified using `git status` and `git branch`.
- Operational tasks should follow the exact scope requested; no code changes were necessary for this challenge.
