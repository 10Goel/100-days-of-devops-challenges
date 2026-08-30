# Day 33 — Git Merge Conflict Resolution Commands

This file contains the commands used during the Day 33 DevOps challenge.

---

## 1. Connect to the Storage Server

```bash
ssh max@ststor01
```

> Enter the password provided by the lab when prompted.

---

## 2. Navigate to the Repository

```bash
cd /home/max/story-blog
```

Verify the location:

```bash
pwd
```

---

## 3. Inspect Repository State

```bash
git status
```

```bash
git branch
```

```bash
git log --oneline --decorate -5
```

```bash
git remote -v
```

```bash
ls -l
```

Inspect the current index:

```bash
cat story-index.txt
```

---

## 4. Attempt the Push

```bash
git push origin master
```

The push is expected to fail if the remote `master` branch contains commits that do not exist locally.

---

## 5. Pull and Merge Remote Changes

```bash
git pull --no-rebase origin master
```

Check the merge state:

```bash
git status
```

Inspect the conflicting file:

```bash
cat story-index.txt
```

---

## 6. Resolve `story-index.txt`

Write the required final content:

```bash
cat > story-index.txt <<'EOF'
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
EOF
```

Verify:

```bash
cat story-index.txt
```

---

## 7. Verify Conflict Markers Are Removed

```bash
grep -nE '<<<<<<<|=======|>>>>>>>' story-index.txt
```

No output is expected.

Verify that the typo no longer exists:

```bash
grep -nE 'Moose|Mooose' story-index.txt
```

No output is expected.

Verify the corrected title:

```bash
grep -n "Mouse" story-index.txt
```

---

## 8. Stage the Resolved File

```bash
git add story-index.txt
```

Check status:

```bash
git status
```

---

## 9. Configure Git Identity if Required

Check existing configuration:

```bash
git config user.name
git config user.email
```

If required:

```bash
git config user.name "max"
git config user.email "max@stratos.xfusioncorp.com"
```

---

## 10. Commit the Resolution

```bash
git commit -m "Resolve story index merge conflict"
```

---

## 11. Push the Final Changes

```bash
git push origin master
```

---

## 12. Final Verification

```bash
git status
```

```bash
cat story-index.txt
```

```bash
git log --oneline --decorate -5
```

Verify local and remote branches point to the same commit:

```bash
git rev-parse master
git rev-parse origin/master
```

---

## Compact Command Flow

```bash
ssh max@ststor01

cd /home/max/story-blog

git status
git branch
git log --oneline --decorate -5
git remote -v
cat story-index.txt

git push origin master
git pull --no-rebase origin master

git status
cat story-index.txt

cat > story-index.txt <<'EOF'
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
EOF

grep -nE '<<<<<<<|=======|>>>>>>>' story-index.txt
grep -nE 'Moose|Mooose' story-index.txt

git add story-index.txt
git status

git commit -m "Resolve story index merge conflict"
git push origin master

git status
cat story-index.txt
git log --oneline --decorate -5
git rev-parse master
git rev-parse origin/master
```

---

## Commands Intentionally Not Used

### Do not force push

```bash
git push --force
```

A force push could overwrite Sarah's changes.

### Do not use sudo with Git

```bash
sudo git ...
```

The repository belongs to the normal user and Git operations should be performed as that user. Using `sudo` could create root-owned files inside the repository and cause permission problems.
