# DevOps Challenge — Day 34 Commands
## Git `post-update` Hook and Automated Release Tag

## 1. Connect to the Storage Server

```bash
ssh natasha@ststor01
```

Verify the user:

```bash
whoami
```

---

## 2. Move to the Working Repository

```bash
cd /usr/src/kodekloudrepos/media
```

Inspect repository state:

```bash
git status
git branch -a
git remote -v
```

---

## 3. Merge `feature` into `master`

```bash
git checkout master
git merge feature
```

Verify the merge:

```bash
git status
git log --oneline --decorate -5
```

---

## 4. Move to the Bare Repository Hook Directory

```bash
cd /opt/media.git/hooks
```

Inspect existing hooks:

```bash
ls -l
```

---

## 5. Create the `post-update` Hook

```bash
vi post-update
```

Add:

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

Save and exit from `vi`:

```text
Esc
:wq
Enter
```

---

## 6. Make the Hook Executable

```bash
chmod +x /opt/media.git/hooks/post-update
```

Verify:

```bash
ls -l /opt/media.git/hooks/post-update
```

---

## 7. Optional Syntax Check

```bash
bash -n /opt/media.git/hooks/post-update
```

No output means the Bash syntax is valid.

---

## 8. Check Today's Release Tag Name

```bash
date +%Y-%m-%d
```

```bash
echo "release-$(date +%Y-%m-%d)"
```

Example:

```text
release-2026-08-31
```

---

## 9. Return to the Working Repository

```bash
cd /usr/src/kodekloudrepos/media
```

Check status before push:

```bash
git status
git log --oneline --decorate -5
```

---

## 10. Push `master`

```bash
git push origin master
```

Observed successful update:

```text
To /opt/media.git
   e082dfd..9d0ebda  master -> master
```

This push triggers:

```text
/opt/media.git/hooks/post-update
```

---

## 11. Verify the Automatically Created Release Tag

```bash
git --git-dir=/opt/media.git tag
```

Or:

```bash
git --git-dir=/opt/media.git tag --list "release-$(date +%Y-%m-%d)"
```

Expected for this challenge:

```text
release-2026-08-31
```

---

## 12. Verify All Branch and Tag References

```bash
git --git-dir=/opt/media.git show-ref --heads --tags
```

---

## 13. Verify `master` Commit

```bash
git --git-dir=/opt/media.git rev-parse refs/heads/master
```

Result during completion:

```text
9d0ebda9402c4300681052b215c81e590c98da2d
```

---

## 14. Verify Release Tag Commit

```bash
git --git-dir=/opt/media.git rev-parse "refs/tags/release-$(date +%Y-%m-%d)"
```

Result:

```text
9d0ebda9402c4300681052b215c81e590c98da2d
```

Matching hashes confirm that the release tag points to the current `master` commit.

---

## 15. Manually Test the Hook If Required

```bash
/opt/media.git/hooks/post-update refs/heads/master
```

The hook includes an existing-tag check, so re-running it on the same date does not attempt to recreate an already existing tag.

---

## 16. Final Repository Verification

```bash
cd /usr/src/kodekloudrepos/media
git status
git branch
git log --oneline --decorate --all -10
```

Verify hook:

```bash
ls -l /opt/media.git/hooks/post-update
```

Verify release tag again:

```bash
git --git-dir=/opt/media.git tag --list "release-$(date +%Y-%m-%d)"
```

---

# Compact Command Flow

```bash
ssh natasha@ststor01

whoami

cd /usr/src/kodekloudrepos/media
git status
git branch -a
git remote -v

git checkout master
git merge feature

cd /opt/media.git/hooks
vi post-update

chmod +x /opt/media.git/hooks/post-update

date +%Y-%m-%d
echo "release-$(date +%Y-%m-%d)"

cd /usr/src/kodekloudrepos/media
git status
git push origin master

git --git-dir=/opt/media.git tag
git --git-dir=/opt/media.git show-ref --heads --tags

git --git-dir=/opt/media.git rev-parse refs/heads/master
git --git-dir=/opt/media.git rev-parse "refs/tags/release-$(date +%Y-%m-%d)"

ls -l /opt/media.git/hooks/post-update
```

---

# Commands to Avoid

The challenge specifically required preserving existing repository and directory permissions.

Avoid broad permission or ownership changes such as:

```bash
sudo chmod -R 777 /opt/media.git
```

```bash
sudo chown -R natasha:natasha /opt/media.git
```

Only the hook itself needed executable permission:

```bash
chmod +x /opt/media.git/hooks/post-update
```
