# Day 32 — Git Rebase Notes

## 1. What Was the Goal?

The developer had ongoing work on the `feature` branch while new commits had already been added to `master`.

The requirement was to place the feature work on top of the latest `master` history **without losing any feature changes and without creating a merge commit**.

The correct Git operation for this requirement is:

```bash
git rebase master
```

when currently checked out on `feature`.

---

## 2. What Does Rebase Do?

Suppose the history initially looks like this:

```text
A---B---C  master
     \
      D---E  feature
```

Running:

```bash
git checkout feature
git rebase master
```

causes Git to temporarily remove the feature-only commits, move the branch base to the latest `master`, and replay those commits:

```text
A---B---C---D'---E'  feature
        ^
      master
```

The logical feature changes remain, but their commit hashes change because Git creates new commit objects during the replay.

---

## 3. Rebase vs Merge

### Merge

A merge may produce history such as:

```text
A---B---C------M
     \        /
      D------E
```

where `M` is a merge commit.

### Rebase

A rebase produces:

```text
A---B---C---D'---E'
```

This is a linear history.

For Day 32, the task explicitly required the second approach.

---

## 4. Why Update `master` First?

Before rebasing, `master` should represent the latest remote state.

The workflow used was:

```bash
sudo git fetch origin
sudo git checkout master
sudo git pull --ff-only origin master
```

This ensured that `feature` was rebased onto the latest intended base rather than an outdated local copy of `master`.

---

## 5. Why Use `git pull --ff-only`?

A normal `git pull` may perform a merge if the local and remote branches have diverged.

Using:

```bash
sudo git pull --ff-only origin master
```

instructs Git to update only when a fast-forward is possible.

This is useful in a task where avoiding unintended merge commits is important.

---

## 6. Why Did the Push Require Force?

Before the rebase, the remote `feature` branch pointed to the original feature commit history.

After rebasing, Git created a new feature commit with a different hash.

Therefore, Git could not update the remote branch using an ordinary fast-forward push.

A normal push was attempted:

```bash
sudo git push origin feature
```

The final update was performed using:

```bash
sudo git push --force-with-lease origin feature
```

The terminal confirmed a forced update of the remote feature branch.

---

## 7. Why `--force-with-lease` Instead of `--force`?

Both options can rewrite the remote branch, but they differ in safety.

### `--force`

```bash
git push --force
```

Overwrites the remote branch regardless of whether someone else updated it.

### `--force-with-lease`

```bash
git push --force-with-lease
```

Checks that the remote branch still points where Git expects before overwriting it.

Therefore, `--force-with-lease` is the safer choice for rebased branches.

---

## 8. Verified Final State

The final log showed:

```text
* bbdfda0 (HEAD -> feature, origin/feature) Add new feature
* b995b83 (origin/master, origin/HEAD, master) Update info.txt
* 7c669da initial commit
```

This proves that:

1. `master` contains `Update info.txt`.
2. `feature` contains `Add new feature`.
3. The feature commit is directly on top of the master commit.
4. `HEAD` points to `feature`.
5. `origin/feature` points to the same rebased commit.
6. No merge commit exists between `master` and `feature`.

---

## 9. Final Remote Verification

The remote branches were checked using:

```bash
sudo git ls-remote --heads origin
```

Both required branches were present:

```text
refs/heads/feature
refs/heads/master
```

The final remote `feature` commit was:

```text
bbdfda006673b496b092b7c75f42bef36c4c6b84
```

The remote `master` commit was:

```text
b995b8305cc0683db98437300eeb5b1115b9b39a
```

---

## 10. Handling Rebase Conflicts

A rebase can stop if the same lines were changed differently on both branches.

Typical workflow:

```bash
git status
```

Resolve the conflict manually, then:

```bash
git add <file>
git rebase --continue
```

If more conflicts appear, repeat the process.

To cancel the entire rebase:

```bash
git rebase --abort
```

This restores the branch to its pre-rebase state.

---

## 11. Important Safety Checks

Before rebasing, verify that the working tree is clean:

```bash
git status
```

If uncommitted work exists, it can be temporarily saved:

```bash
git stash push -u -m "backup-before-rebase"
```

After the rebase:

```bash
git stash pop
```

This prevents accidental loss or interference from local uncommitted changes.

---

## 12. Key Takeaways

- Rebase is useful for maintaining a clean, linear Git history.
- `git rebase master` while on `feature` replays feature commits on top of `master`.
- Rebase changes commit hashes.
- Published rebased branches can require a force push.
- `git push --force-with-lease` is safer than plain `--force`.
- `git pull --ff-only` helps avoid accidental merge commits.
- `git log --oneline --graph --decorate --all` is extremely useful for validating branch topology.
- Always inspect `git status` before and after history-changing operations.

---

## Day 32 Summary

Day 32 demonstrated how to safely integrate the latest `master` changes into an in-progress `feature` branch using **Git rebase** rather than merge. The repository was inspected, remote updates were fetched, `master` was fast-forwarded, and `feature` was rebased onto it. The resulting history was verified as linear with the feature commit directly above the master commit. Because rebasing rewrote the feature commit history, the updated branch was pushed using `git push --force-with-lease`, and the final local and remote branch references were verified successfully.
