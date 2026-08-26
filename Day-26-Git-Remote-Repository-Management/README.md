# DevOps Challenge - Day 26: Git Remote Repository Management

## Overview

In this challenge, the xFusionCorp development team had already cloned the `apps.git` repository under `/usr/src/kodekloudrepos/apps` on the Storage Server. A new Git remote repository was introduced, and the local repository had to be updated accordingly.

The task required adding a new remote named `dev_apps`, copying a provided `index.html` file into the repository, committing the change to the `master` branch, and pushing the updated branch to the newly configured remote repository.

---

## Task Requirements

1. Work with the repository located at:

   ```text
   /usr/src/kodekloudrepos/apps
   ```

2. Add a new Git remote named:

   ```text
   dev_apps
   ```

3. Point the new remote to:

   ```text
   /opt/xfusioncorp_apps.git
   ```

4. Copy the following file into the Git repository:

   ```text
   /tmp/index.html
   ```

5. Add and commit the file to the `master` branch.

6. Push the `master` branch to the new `dev_apps` remote.

---

## Environment

- **Server:** Storage Server
- **Repository:** `/usr/src/kodekloudrepos/apps`
- **Source File:** `/tmp/index.html`
- **New Remote Name:** `dev_apps`
- **New Remote Repository:** `/opt/xfusioncorp_apps.git`
- **Branch:** `master`

---

## Implementation

### 1. Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/apps
```

### 2. Inspect Existing Git Remotes

```bash
sudo git remote -v
```

The repository already contained the original `origin` remote pointing to `/opt/apps.git`.

### 3. Add the New Remote

```bash
sudo git remote add dev_apps /opt/xfusioncorp_apps.git
```

Verify the new remote:

```bash
sudo git remote -v
```

Expected configuration:

```text
dev_apps  /opt/xfusioncorp_apps.git (fetch)
dev_apps  /opt/xfusioncorp_apps.git (push)
origin    /opt/apps.git (fetch)
origin    /opt/apps.git (push)
```

### 4. Copy `index.html` into the Repository

```bash
sudo cp /tmp/index.html /usr/src/kodekloudrepos/apps/
```

Check the repository status:

```bash
sudo git status
```

### 5. Stage the File

```bash
sudo git add index.html
```

### 6. Confirm the Current Branch

```bash
sudo git branch --show-current
```

Output:

```text
master
```

### 7. Commit the Change

```bash
sudo git commit -m "Add index.html"
```

The successful commit created:

```text
f534023 Add index.html
```

### 8. Push the `master` Branch to the New Remote

```bash
sudo git push dev_apps master
```

The push completed successfully and created the remote `master` branch in `/opt/xfusioncorp_apps.git`.

---

## Verification

### Check Working Tree

```bash
sudo git status
```

Result:

```text
On branch master
nothing to commit, working tree clean
```

### Verify Remote Configuration

```bash
sudo git remote -v
```

Confirmed:

```text
dev_apps  /opt/xfusioncorp_apps.git (fetch)
dev_apps  /opt/xfusioncorp_apps.git (push)
origin    /opt/apps.git (fetch)
origin    /opt/apps.git (push)
```

### Verify Local Commit History

```bash
sudo git log --oneline -3
```

Confirmed:

```text
f534023 (HEAD -> master, dev_apps/master) Add index.html
777dd25 (origin/master) initial commit
```

### Verify Commit Directly in the New Bare Repository

```bash
sudo git --git-dir=/opt/xfusioncorp_apps.git log --oneline -3
```

Confirmed:

```text
f534023 (HEAD -> master) Add index.html
777dd25 initial commit
```

---

## Final Result

The Day 26 challenge was completed successfully.

- New Git remote `dev_apps` was configured.
- `dev_apps` correctly points to `/opt/xfusioncorp_apps.git`.
- `/tmp/index.html` was copied into the local repository.
- The file was staged and committed to the `master` branch.
- The `master` branch was successfully pushed to the new remote.
- The remote bare repository was verified to contain the new commit.
- The working tree remained clean after the operation.

---

## Key Concepts Practiced

- Git remote management
- Adding multiple remotes to a repository
- Working with bare Git repositories
- Git staging and commits
- Branch verification
- Pushing branches to specific remotes
- Git repository validation
- Linux file operations with elevated permissions

---

## Challenge Status

**Completed Successfully ✅**
