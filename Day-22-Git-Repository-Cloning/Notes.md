# DevOps Day 22 — Notes

## Topic: Cloning a Git Repository from a Local Path

### Challenge Summary

The task required cloning an existing Git repository located on the Storage Server at:

```bash
/opt/apps.git
```

into:

```bash
/usr/src/kodekloudrepos
```

The task had to be performed as the `natasha` user without altering permissions or making unauthorized changes.

---

## 1. What Does `git clone` Do?

`git clone` creates a new copy of an existing Git repository.

General syntax:

```bash
git clone <repository>
```

For this challenge:

```bash
git clone /opt/apps.git
```

Since the command was executed inside:

```bash
/usr/src/kodekloudrepos
```

Git created:

```bash
/usr/src/kodekloudrepos/apps
```

---

## 2. Why Was the Directory Named `apps`?

Git normally derives the destination directory name from the repository name.

Source:

```text
apps.git
```

The `.git` suffix is removed, producing:

```text
apps
```

Therefore:

```bash
git clone /opt/apps.git
```

creates:

```text
apps/
```

in the current working directory.

---

## 3. What Is a Bare Git Repository?

A bare repository contains Git's repository data but normally does not contain a checked-out working tree.

A typical bare repository may look similar to:

```text
HEAD
branches/
config
description
hooks/
info/
objects/
refs/
```

Bare repositories commonly use the `.git` suffix and are often used as shared or central repositories.

The source repository in this task:

```bash
/opt/apps.git
```

is expected to be a bare repository.

To check:

```bash
git --git-dir=/opt/apps.git rev-parse --is-bare-repository
```

Expected:

```text
true
```

---

## 4. Local Clone vs Remote Clone

Git repositories do not always need to be cloned over HTTP or SSH.

Examples:

### Local repository

```bash
git clone /opt/apps.git
```

### SSH repository

```bash
git clone user@server:/path/project.git
```

### HTTPS repository

```bash
git clone https://example.com/project.git
```

In this challenge, both the source repository and destination were on the same Storage Server, so a local filesystem path was used.

---

## 5. Why Use the `natasha` User?

The task explicitly required the clone operation to be performed as `natasha`.

This matters because files created by the clone operation are normally owned by the user executing the command.

Checking the active user before performing administrative work is a useful habit:

```bash
whoami
```

---

## 6. Verifying the Clone

After cloning, verify that the repository exists:

```bash
ls -la /usr/src/kodekloudrepos
```

Then enter the repository:

```bash
cd /usr/src/kodekloudrepos/apps
```

Check its Git state:

```bash
git status
```

Inspect the configured remote:

```bash
git remote -v
```

The `origin` remote should point to:

```text
/opt/apps.git
```

---

## 7. What Is `origin`?

When a repository is cloned, Git normally creates a remote called:

```text
origin
```

It represents the repository from which the clone was created.

For this challenge:

```text
origin → /opt/apps.git
```

You can inspect it using:

```bash
git remote -v
```

---

## 8. Why We Did Not Use `git init`

`git init` creates a new Git repository.

The task did not require creating a new repository. It required copying an already existing repository.

Therefore the correct command was:

```bash
git clone /opt/apps.git
```

and not:

```bash
git init
```

---

## 9. Why Permissions Should Not Be Changed

The challenge specifically warned against modifying existing directories or the repository unnecessarily.

Commands such as:

```bash
chmod
chown
```

should therefore not be used unless the task explicitly requires them.

This follows the DevOps principle of minimizing system changes and preserving the intended server configuration.

---

## 10. Important Verification Commands

### Check current user

```bash
whoami
```

### Check source repository

```bash
ls -ld /opt/apps.git
```

### Confirm bare repository

```bash
git --git-dir=/opt/apps.git rev-parse --is-bare-repository
```

### Check destination

```bash
ls -ld /usr/src/kodekloudrepos
```

### Verify cloned repository

```bash
git -C /usr/src/kodekloudrepos/apps status
```

### Verify remote

```bash
git -C /usr/src/kodekloudrepos/apps remote -v
```

---

## Key Takeaways

- `git clone` copies an existing Git repository.
- Git repositories can be cloned from local filesystem paths.
- Bare repositories are commonly used as central/shared Git repositories.
- `origin` is automatically configured when cloning.
- Always execute commands using the required account.
- Do not change file permissions or ownership unless required.
- Verification after completing an infrastructure task is essential.
