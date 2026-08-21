# DevOps Day 23 — Notes

## 📖 Topic: Forking a Git Repository in Gitea

### What is a Git Fork?

A **fork** is a server-side copy of another user's repository created under your own account or namespace.

For this challenge:

```text
Original repository:
sarah/story-blog

Forked repository:
jon/story-blog
```

The new repository belongs to Jon while retaining a relationship with Sarah's original repository.

---

## Why Fork a Repository?

Forking is commonly used when a developer wants to contribute to a project but should not directly modify the original repository.

A typical workflow is:

```text
Original Repository
       ↓
      Fork
       ↓
Developer Repository
       ↓
Local Clone
       ↓
Feature Branch
       ↓
Commit + Push
       ↓
Pull Request
       ↓
Original Repository
```

---

## Fork vs Branch

A **branch** exists inside the same repository.

A **fork** creates another repository, usually under another user's account.

Example:

```text
Repository:
sarah/story-blog

Branch:
sarah/story-blog → feature/new-story

Fork:
sarah/story-blog → jon/story-blog
```

---

## Fork vs Clone

A fork and a clone solve different problems.

### Fork

Created on the Git hosting platform:

```text
sarah/story-blog
        ↓
      Fork
        ↓
jon/story-blog
```

### Clone

Creates a local copy:

```text
jon/story-blog
      ↓
 git clone
      ↓
Local machine
```

In many collaboration workflows, developers first fork the repository and then clone their fork.

---

## Origin and Upstream

When a developer clones their own fork, Git normally creates:

```text
origin
```

`origin` points to the developer's fork.

Example:

```text
origin → jon/story-blog
```

The original repository can then be added manually as:

```text
upstream
```

Example:

```text
upstream → sarah/story-blog
```

This gives the developer two important remotes:

```text
origin   = personal fork
upstream = original repository
```

---

## Keeping a Fork Updated

The upstream repository may continue receiving new commits.

To synchronize a fork:

```bash
git fetch upstream
git checkout master
git merge upstream/master
git push origin master
```

This brings changes from the original project into the developer's fork.

---

## Gitea Forking Process

The challenge was completed using the Gitea web interface.

### Required configuration

```text
Owner: jon
Fork From: sarah/story-blog
Repository Name: story-blog
Branch to be cloned to fork: All branches
```

After selecting **Fork Repository**, Gitea created:

```text
jon/story-blog
```

and displayed:

```text
forked from sarah/story-blog
```

---

## Repository History

A fork normally keeps the existing Git history from the source repository.

That means commits, branches selected during the fork, and repository files are carried into the new repository.

In this challenge, the fork showed the existing commit history and project files, confirming that the repository was copied successfully.

---

## Why This Matters in DevOps

Forking supports collaborative development by allowing engineers to work independently while preserving a controlled integration path.

It is useful for:

- feature development
- code reviews
- pull requests
- open-source contributions
- experimentation
- protecting upstream repositories
- team-based Git workflows

---

## Key Takeaways

- A fork is created on the Git hosting server.
- A clone is created on a local machine.
- The fork belongs to the user who creates it.
- The original repository can be treated as `upstream`.
- The developer's fork is usually `origin` after cloning.
- Forks allow independent development without directly changing the source repository.
- Changes can later be proposed back to the original project through pull requests.
- Gitea provides a workflow similar to GitHub and other Git hosting platforms.

---

## Challenge Result

The task was completed successfully by creating the repository:

```text
jon/story-blog
```

from:

```text
sarah/story-blog
```

This completed the DevOps Day 23 Git repository forking challenge.
