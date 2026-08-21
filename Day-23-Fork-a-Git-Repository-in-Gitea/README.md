# DevOps Day 23 — Fork a Git Repository in Gitea

## 📌 Challenge Overview

In this challenge, a Git server is being used by the Nautilus project team. A new developer, **Jon**, joined the team and needed to begin work on an existing project without modifying the original repository directly.

The task was to log in to the **Gitea** web interface and fork the existing repository:

- **Source repository:** `sarah/story-blog`
- **Fork owner:** `jon`
- **Forked repository:** `jon/story-blog`

The repository was successfully forked under Jon's account.

---

## 🎯 Objective

Fork the existing Gitea repository `sarah/story-blog` into the `jon` user account while preserving the repository content and history.

---

## 🛠️ Environment

| Component | Details |
|---|---|
| Git platform | Gitea |
| Source owner | `sarah` |
| Source repository | `story-blog` |
| Target user | `jon` |
| Target repository | `story-blog` |
| Operation | Repository Fork |

---

## ✅ Task Steps

1. Open the **Gitea UI** from the lab environment.
2. Log in using the provided Jon account credentials.
3. Navigate to the repository:
   ```text
   sarah/story-blog
   ```
4. Select the **Fork** option.
5. Set the fork owner to:
   ```text
   jon
   ```
6. Keep the repository name as:
   ```text
   story-blog
   ```
7. Keep **All branches** selected.
8. Click **Fork Repository**.
9. Verify that the newly created repository appears as:
   ```text
   jon/story-blog
   ```
10. Confirm that Gitea displays:
    ```text
    forked from sarah/story-blog
    ```

---

## 🔍 Verification

After the fork was created, the repository page confirmed:

```text
jon/story-blog
forked from sarah/story-blog
```

The fork contained the existing project files and commit history, including files such as:

```text
frogs-and-ox.txt
lion-and-mouse.txt
```

This confirmed that the repository was successfully copied into Jon's namespace as a Git fork.

---

## 🧠 What I Learned

This challenge demonstrated how repository forking works in a Git hosting platform such as Gitea.

A fork creates a server-side copy of an existing repository under another user's namespace. The new developer can independently create branches, make commits, and test changes without directly modifying the upstream repository.

The original repository remains the upstream source, while the fork can later be used to contribute changes back through pull requests.

---

## 🔄 Fork vs Clone

| Fork | Clone |
|---|---|
| Creates a copy on the Git hosting server | Creates a local working copy |
| Usually belongs to another user/account | Exists on the local machine |
| Useful for collaboration and pull requests | Useful for local development |
| Maintains relationship with upstream repository | Does not automatically create a server-side fork |

---

## 📚 Key Git/Gitea Concepts

- Repository forking
- Upstream repository
- User namespaces
- Git collaboration workflow
- Branch isolation
- Pull-request based development
- Gitea repository management

---

## 🏁 Result

**Status: Successfully Completed ✅**

The repository `sarah/story-blog` was successfully forked into the `jon` account as:

```text
jon/story-blog
```

The fork retained the source repository's files and commit history and remained linked to the original repository as its upstream source.
