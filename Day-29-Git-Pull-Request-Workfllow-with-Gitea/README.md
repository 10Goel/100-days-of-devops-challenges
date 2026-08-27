# DevOps Challenge – Git Pull Request Workflow with Gitea

## 📌 Overview

This challenge focused on implementing a proper **Git Pull Request (PR) workflow** using **Gitea**. Instead of allowing direct changes to the `master` branch, the task required changes to be developed on a separate feature branch, reviewed by another user, approved, and then merged into `master`.

The story **“The Fox and Grapes”** had already been pushed by **Max** to the remote branch `story/fox-and-grapes`. The objective was to verify the repository history, create a Pull Request from the feature branch into `master`, assign **Tom** as a reviewer, approve the changes as Tom, and finally merge the Pull Request.

---

## 🎯 Task Requirements

- SSH into the storage server as **Max**.
- Locate and inspect the cloned Git repository.
- Verify the existing branch and commit history.
- Confirm Max's story was present on `story/fox-and-grapes`.
- Create a Pull Request in **Gitea**:
  - **Title:** `Added fox-and-grapes story`
  - **Source branch:** `story/fox-and-grapes`
  - **Destination branch:** `master`
- Assign **Tom** as the reviewer.
- Log in as Tom.
- Review and approve the Pull Request.
- Merge the approved Pull Request into `master`.

---

## 🏗️ Environment

| Component | Details |
|---|---|
| Git Hosting | Gitea |
| Repository | `sarah/story-blog` |
| Feature Branch | `story/fox-and-grapes` |
| Target Branch | `master` |
| PR Author | Max |
| Reviewer | Tom |
| Workflow | Feature Branch → Pull Request → Review → Approval → Merge |

---

## 🔄 Workflow

```text
Developer Work
     │
     ▼
story/fox-and-grapes
     │
     ▼
Push Feature Branch
     │
     ▼
Create Pull Request
     │
     ▼
Assign Tom as Reviewer
     │
     ▼
Tom Reviews Changes
     │
     ▼
Approve Pull Request
     │
     ▼
Merge into master
```

---

## 🛠️ Implementation

### 1. Connected to the Storage Server

Max logged into the storage server over SSH.

```bash
ssh max@ststor01
```

The repository was then located and inspected.

---

### 2. Verified Git Repository State

The repository status, branches, remotes, and commit history were checked.

```bash
git status
git branch -a
git remote -v
git log --oneline --all --decorate --graph
```

The feature branch was confirmed to contain Max's story.

```bash
git log --oneline story/fox-and-grapes
```

The commits present on the story branch but not yet merged into `master` were checked with:

```bash
git log --oneline master..story/fox-and-grapes
```

---

### 3. Created the Pull Request in Gitea

The Pull Request was created with the following configuration:

```text
Title: Added fox-and-grapes story
Source: story/fox-and-grapes
Target: master
```

This ensured the feature branch changes would be reviewed before entering the main branch.

---

### 4. Assigned Tom as Reviewer

Tom was added as the reviewer from the Pull Request's **Reviewers** section.

This introduced a peer-review step into the workflow and prevented immediate merging without another developer's approval.

---

### 5. Reviewed and Approved the Pull Request

After logging into Gitea as **Tom**, the changed files were reviewed and the Pull Request was approved.

The approval confirmed that the proposed changes were acceptable for integration into `master`.

---

### 6. Merged the Pull Request

After approval, Tom merged the Pull Request into `master`.

The final Gitea status confirmed:

```text
Pull request successfully merged and closed.
```

---

## ✅ Verification

The successful merge was confirmed directly in the Gitea Pull Request history.

Optional CLI verification can also be performed:

```bash
git fetch origin
git log --oneline --decorate --graph origin/master -10
```

This confirms that the story and merge commit are now part of the remote `master` branch history.

---

## 🔐 Why Pull Requests Matter

Pull Requests provide a controlled method of integrating code into important branches such as `master` or `main`.

Benefits include:

- **Peer review** before changes are accepted.
- **Traceability** of who proposed, reviewed, and merged changes.
- **Reduced risk** of accidental direct modifications.
- **Better collaboration** between multiple developers.
- **Audit history** through review comments and merge records.
- **Higher code quality** through approval workflows.

---

## 💡 Key Learning

This challenge demonstrated that a professional Git workflow should avoid direct modifications to the primary branch whenever possible. Development is performed on a feature branch, pushed to the remote repository, reviewed through a Pull Request, approved by another team member, and only then merged into the protected branch.

The completed workflow was:

```text
Max → story/fox-and-grapes → Pull Request → Tom Review → Approval → Merge → master
```

---

## 🏁 Result

✅ Repository inspected successfully  
✅ Feature branch verified  
✅ Pull Request created  
✅ Tom assigned as reviewer  
✅ Pull Request reviewed and approved  
✅ Changes merged successfully into `master`  
✅ Pull Request closed successfully

---

## 📚 Skills Practiced

- Git branch management
- Git history inspection
- Remote repository workflows
- Gitea
- Pull Requests
- Peer review
- Branch-based development
- Merge workflows
- DevOps collaboration practices
