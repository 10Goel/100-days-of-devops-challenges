# DevOps Day 22 — Commands

## Challenge

Clone:

```text
/opt/apps.git
```

into:

```text
/usr/src/kodekloudrepos
```

on the Storage Server using the `natasha` user.

---

## 1. Connect to the Storage Server

```bash
ssh natasha@ststor01
```

---

## 2. Confirm Current User

```bash
whoami
```

Expected:

```text
natasha
```

---

## 3. Verify Destination Directory

```bash
ls -ld /usr/src/kodekloudrepos
```

```bash
ls -la /usr/src/kodekloudrepos
```

---

## 4. Verify Source Repository

```bash
ls -ld /opt/apps.git
```

Optional bare repository check:

```bash
git --git-dir=/opt/apps.git rev-parse --is-bare-repository
```

Expected:

```text
true
```

---

## 5. Move to Destination

```bash
cd /usr/src/kodekloudrepos
```

---

## 6. Clone Repository

```bash
git clone /opt/apps.git
```

Expected result:

```text
Cloning into 'apps'...
done.
```

If the repository contains no commits yet, Git may display:

```text
warning: You appear to have cloned an empty repository.
```

This is valid if the source repository is empty.

---

## 7. Verify Repository Directory

```bash
ls -la /usr/src/kodekloudrepos
```

The following directory should exist:

```text
apps
```

---

## 8. Enter the Cloned Repository

```bash
cd /usr/src/kodekloudrepos/apps
```

---

## 9. Check Git Status

```bash
git status
```

---

## 10. Verify Remote Repository

```bash
git remote -v
```

Expected remote path:

```text
/opt/apps.git
```

---

## Quick Command Sequence

```bash
ssh natasha@ststor01
whoami
cd /usr/src/kodekloudrepos
git clone /opt/apps.git
cd apps
git status
git remote -v
```

---

## Useful Alternative Verification

Check whether the cloned directory is a Git working tree:

```bash
git -C /usr/src/kodekloudrepos/apps rev-parse --is-inside-work-tree
```

Expected:

```text
true
```

Check the configured origin URL:

```bash
git -C /usr/src/kodekloudrepos/apps remote get-url origin
```

Expected:

```text
/opt/apps.git
```

---

## Commands Not Required

Avoid unnecessary changes such as:

```bash
chmod
chown
git init
rm -rf
```

These commands are not required for this task and may modify the server state incorrectly.
