# DevOps Challenge — Day 34
## Git Post-Update Hook for Automated Release Tagging

## Overview

In this challenge, the Nautilus application development team needed to automate release tagging for the Git repository `/opt/media.git`, whose working clone is located under `/usr/src/kodekloudrepos/media` on the Storage Server in Stratos DC.

The task required merging the `feature` branch into `master`, configuring a **server-side `post-update` Git hook**, and ensuring that every push to the `master` branch automatically creates a date-based release tag in the following format:

```text
release-YYYY-MM-DD
```

For example:

```text
release-2026-08-31
```

The challenge also required testing the hook at least once, creating the release tag for the current date, pushing the changes, performing the work as the `natasha` user, and avoiding any unnecessary modification of existing repository or directory permissions.

---

## Task Requirements

The challenge specifically required the following:

1. Connect to the Storage Server as the `natasha` user.
2. Work with the cloned repository:
   ```text
   /usr/src/kodekloudrepos/media
   ```
3. Merge the `feature` branch into the `master` branch.
4. Do **not** push immediately after the merge.
5. Create a `post-update` hook inside:
   ```text
   /opt/media.git/hooks/
   ```
6. Configure the hook so that whenever `master` is updated by a push, a release tag is created automatically using the current date:
   ```text
   release-YYYY-MM-DD
   ```
7. Make the hook executable.
8. Push the merged `master` branch to `/opt/media.git`.
9. Confirm that the hook runs successfully and creates the release tag for the current date.
10. Verify that the generated tag points to the same commit as `master`.
11. Preserve existing repository and directory permissions.

---

## Environment

| Component | Value |
|---|---|
| Server | Storage Server |
| User | `natasha` |
| Bare Git Repository | `/opt/media.git` |
| Working Clone | `/usr/src/kodekloudrepos/media` |
| Source Branch | `feature` |
| Target Branch | `master` |
| Hook Type | `post-update` |
| Release Tag Format | `release-YYYY-MM-DD` |

---

## Implementation

### 1. Connect to the Storage Server

```bash
ssh natasha@ststor01
```

Verify the logged-in user:

```bash
whoami
```

Expected:

```text
natasha
```

---

### 2. Inspect the Repository

```bash
cd /usr/src/kodekloudrepos/media
git status
git branch -a
git remote -v
```

This verifies the current branch state and confirms that the remote points to the bare repository.

---

### 3. Merge `feature` into `master`

```bash
git checkout master
git merge feature
```

After the merge:

```bash
git status
git log --oneline --decorate -5
```

The merge was completed before pushing, as required by the task.

---

### 4. Create the `post-update` Hook

The hook was created in the bare repository:

```bash
cd /opt/media.git/hooks
vi post-update
```

Hook content:

```bash
#!/bin/bash

for ref in "$@"
do
    if [ "$ref" = "refs/heads/master" ]; then
        TAG="release-$(date +%Y-%m-%d)"

        if ! git --git-dir=/opt/media.git rev-parse -q --verify "refs/tags/$TAG" >/dev/null
        then
            git --git-dir=/opt/media.git tag "$TAG" refs/heads/master
        fi
    fi
done

git update-server-info
```

### What this hook does

- Git runs `post-update` after refs are updated in the remote repository.
- The hook receives updated refs as command-line arguments.
- It checks whether `refs/heads/master` was one of the updated refs.
- If `master` was updated, it builds a tag using the current date:
  ```text
  release-YYYY-MM-DD
  ```
- It checks whether the tag already exists.
- If the tag does not exist, it creates the tag against the current `master` commit.
- `git update-server-info` updates auxiliary repository metadata.

---

### 5. Make the Hook Executable

```bash
chmod +x /opt/media.git/hooks/post-update
```

Verification:

```bash
ls -l /opt/media.git/hooks/post-update
```

The executable bit is essential because Git will ignore a hook that is not executable.

---

### 6. Confirm the Expected Release Tag

```bash
date +%Y-%m-%d
echo "release-$(date +%Y-%m-%d)"
```

For the completed challenge, the generated tag was:

```text
release-2026-08-31
```

---

### 7. Push `master`

```bash
cd /usr/src/kodekloudrepos/media
git push origin master
```

The push updated `master` inside `/opt/media.git`, which triggered the `post-update` hook automatically.

Observed push result:

```text
To /opt/media.git
   e082dfd..9d0ebda  master -> master
```

---

## Verification

### Verify the release tag

```bash
git --git-dir=/opt/media.git tag
```

Result:

```text
release-2026-08-31
```

### Verify refs

```bash
git --git-dir=/opt/media.git show-ref --heads --tags
```

The final refs showed both `master` and the release tag pointing to the same commit.

### Compare `master` and the tag directly

```bash
git --git-dir=/opt/media.git rev-parse refs/heads/master
git --git-dir=/opt/media.git rev-parse "refs/tags/release-$(date +%Y-%m-%d)"
```

Both commands returned:

```text
9d0ebda9402c4300681052b215c81e590c98da2d
```

This confirms that:

```text
master
   |
   v
9d0ebda...
   ^
   |
release-2026-08-31
```

The release tag therefore represents the exact commit deployed to `master`.

---

## Final Result

The challenge was completed successfully.

- `feature` was merged into `master`
- `master` was pushed successfully
- A server-side `post-update` hook was configured
- The hook automatically created a date-based release tag
- The release tag `release-2026-08-31` was created
- The tag points to the same commit as `master`
- The hook was made executable and tested
- Existing repository/directory permissions were preserved
- All operations were performed as `natasha`

---

## Key Learning

This challenge demonstrates how **Git server-side hooks** can automate repository actions after developers push changes.

Instead of relying on a developer to manually create a release tag after every push to `master`, the repository itself performs the operation automatically. This is a simple example of the automation principles commonly used in CI/CD workflows.

---

## Day 34 Summary

Day 34 focused on implementing Git automation using a server-side `post-update` hook. The `feature` branch was merged into `master`, after which a hook was installed in the bare repository `/opt/media.git`. The hook detects updates to `master` and automatically creates a release tag in the format `release-YYYY-MM-DD`. After making the hook executable, the merged branch was pushed, which triggered the hook and generated `release-2026-08-31`. The release tag was then verified to point to the same commit as the updated `master` branch.
