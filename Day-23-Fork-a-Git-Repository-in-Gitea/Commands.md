# DevOps Day 23 — Commands Reference

## 📌 Important Note

This challenge was completed primarily through the **Gitea web interface**, so no shell command was required to perform the actual fork.

The following Git commands are useful for understanding and working with the fork after it has been created.

---

## 1. Clone the Forked Repository

```bash
git clone <fork-repository-url>
```

Example structure:

```bash
git clone http://<gitea-host>/jon/story-blog.git
```

---

## 2. Enter the Repository

```bash
cd story-blog
```

---

## 3. Check Configured Remotes

```bash
git remote -v
```

Expected conceptually:

```text
origin  <jon-fork-url> (fetch)
origin  <jon-fork-url> (push)
```

---

## 4. Add the Original Repository as Upstream

```bash
git remote add upstream <original-repository-url>
```

Example structure:

```bash
git remote add upstream http://<gitea-host>/sarah/story-blog.git
```

---

## 5. Verify Origin and Upstream

```bash
git remote -v
```

A typical configuration becomes:

```text
origin    <jon-fork-url>     (fetch)
origin    <jon-fork-url>     (push)
upstream  <sarah-repo-url>   (fetch)
upstream  <sarah-repo-url>   (push)
```

---

## 6. Fetch Changes from Upstream

```bash
git fetch upstream
```

---

## 7. Check Available Branches

```bash
git branch -a
```

---

## 8. Synchronize the Local Main Branch

If the repository uses `master`:

```bash
git checkout master
git merge upstream/master
```

If the repository uses `main`:

```bash
git checkout main
git merge upstream/main
```

---

## 9. Push Updated Branch to the Fork

For a `master` branch:

```bash
git push origin master
```

For a `main` branch:

```bash
git push origin main
```

---

## 10. Create a Feature Branch

```bash
git checkout -b feature/my-change
```

---

## 11. Check Repository Status

```bash
git status
```

---

## 12. Stage Changes

```bash
git add .
```

---

## 13. Commit Changes

```bash
git commit -m "Add project changes"
```

---

## 14. Push Feature Branch

```bash
git push -u origin feature/my-change
```

---

## 🧾 Gitea UI Operation Used in the Challenge

```text
Gitea UI
→ Login as jon
→ Open sarah/story-blog
→ Fork
→ Owner: jon
→ Repository Name: story-blog
→ Branch to be cloned to fork: All branches
→ Fork Repository
```

---

## ✅ Final Verification

The target repository should display:

```text
jon/story-blog
forked from sarah/story-blog
```
