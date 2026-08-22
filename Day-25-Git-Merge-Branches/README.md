# Day 25 - Git Branching, Commit, Merge and Push

## Overview

This challenge focused on practical Git branch management inside the Nautilus application repository. The task required creating a new branch from `master`, copying an existing HTML file into the repository, committing the change on the new branch, merging the branch back into `master`, and finally pushing both branches to the remote repository.

## Task Requirements

- Work with the repository located at `/usr/src/kodekloudrepos/news`.
- Create a new branch named `nautilus` from `master`.
- Copy `/tmp/index.html` into the repository.
- Add and commit the file on the `nautilus` branch.
- Merge the `nautilus` branch back into `master`.
- Push both `master` and `nautilus` to the remote repository.

## Environment

- **Server:** Storage Server
- **Repository:** `/usr/src/kodekloudrepos/news`
- **Remote Repository:** `/opt/news.git`
- **Source File:** `/tmp/index.html`
- **Feature Branch:** `nautilus`
- **Base Branch:** `master`

## Implementation

### 1. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/news
```

Git initially reported a dubious ownership issue. The repository was added to Git's trusted safe-directory list:

```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/news
```

### 2. Switch to the Master Branch

```bash
sudo git checkout master
```

### 3. Create the Nautilus Branch

```bash
sudo git checkout -b nautilus
```

### 4. Copy the Required File

```bash
sudo cp /tmp/index.html .
```

### 5. Stage and Commit the File

```bash
sudo git add index.html
sudo git commit -m "Add index.html"
```

### 6. Merge Nautilus into Master

```bash
sudo git checkout master
sudo git merge nautilus
```

The merge completed as a **fast-forward merge**, because no new commits had been made on `master` after the `nautilus` branch was created.

### 7. Push Both Branches

```bash
sudo git push origin nautilus
sudo git push origin master
```

## Verification

The repository and remote branches were verified using:

```bash
sudo git status
sudo git branch
sudo git log --oneline --decorate --graph --all
sudo git ls-remote --heads origin
```

The remote repository contained both required branches:

```text
refs/heads/master
refs/heads/nautilus
```

## Result

The Day 25 challenge was completed successfully.

The final repository state satisfied all requirements:

- `nautilus` was created from `master`.
- `index.html` was copied into the repository.
- The file was committed on `nautilus`.
- `nautilus` was merged back into `master`.
- Both branches were pushed to the origin repository.

## Key Concepts Practiced

- Git branch creation
- Branch switching
- Git staging and commits
- Fast-forward merges
- Remote branch pushes
- Repository ownership and Git safe-directory configuration
- Git verification commands

---

**Challenge Status:** Completed Successfully
