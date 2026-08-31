# DevOps Challenge — Day 34 Notes
## Git Hooks and Automated Release Tagging

## 1. What Was the Task?

The goal of Day 34 was to automate release-tag creation whenever changes are pushed to the `master` branch of the bare Git repository:

```text
/opt/media.git
```

The repository had a working clone at:

```text
/usr/src/kodekloudrepos/media
```

The challenge required:

- merging `feature` into `master`
- creating a `post-update` Git hook
- ensuring the hook runs whenever `master` is pushed
- creating a release tag automatically using the current date
- using the format:
  ```text
  release-YYYY-MM-DD
  ```
- testing the hook at least once
- creating today's release tag
- pushing the changes
- performing the task as `natasha`
- avoiding unnecessary permission changes

The important DevOps concept in this task is **Git Hooks**.

---

# 2. What Are Git Hooks?

Git hooks are executable scripts that Git runs automatically when specific Git events occur.

Examples of events include:

- committing code
- merging branches
- pushing changes
- receiving a push on a remote repository
- updating references
- rebasing
- checking out branches

Hooks allow us to attach automation to normal Git operations.

A simple conceptual flow is:

```text
Git Event
   |
   v
Hook Triggered
   |
   v
Custom Script Runs
   |
   v
Automated Action
```

For example:

```text
Developer pushes master
        |
        v
Remote repository receives update
        |
        v
post-update hook runs
        |
        v
release-YYYY-MM-DD tag is created
```

---

# 3. Where Are Git Hooks Stored?

Git hooks are stored inside the repository's `hooks` directory.

For a normal repository:

```text
.git/hooks/
```

Example:

```text
project/.git/hooks/
```

For a bare repository:

```text
repository.git/hooks/
```

In this challenge:

```text
/opt/media.git/hooks/
```

The hook therefore had to be created as:

```text
/opt/media.git/hooks/post-update
```

---

# 4. What Is a Bare Git Repository?

A bare repository contains Git's repository data but does not contain a checked-out working tree.

Typical bare repository:

```text
media.git/
├── HEAD
├── config
├── hooks/
├── objects/
├── refs/
└── ...
```

Bare repositories are frequently used as central Git remotes.

In this challenge:

```text
/opt/media.git
```

was the bare repository.

The actual working copy was:

```text
/usr/src/kodekloudrepos/media
```

The working repository could therefore use commands such as:

```bash
git checkout
git merge
git status
```

while the bare repository primarily received pushes and stored refs.

---

# 5. Client-Side vs Server-Side Git Hooks

Git hooks can broadly be thought of as client-side or server-side hooks.

## Client-Side Hooks

These execute on a developer's local repository.

Examples:

```text
pre-commit
prepare-commit-msg
commit-msg
post-commit
pre-rebase
```

Typical uses:

- code formatting
- linting
- checking commit messages
- preventing invalid commits

---

## Server-Side Hooks

These run on the remote/bare Git repository when pushes are received.

Common examples:

```text
pre-receive
update
post-receive
post-update
```

Typical uses:

- validating pushes
- rejecting unauthorized changes
- triggering deployments
- creating tags
- notifying external systems
- starting CI/CD processes

Day 34 specifically used a **server-side hook**.

---

# 6. What Is the `post-update` Hook?

`post-update` is a server-side Git hook.

It executes **after the remote repository has successfully updated its refs**.

Its general lifecycle is:

```text
git push
   |
   v
remote receives objects
   |
   v
refs are updated
   |
   v
post-update executes
```

The hook receives the names of updated refs as command-line arguments.

For example, when `master` is updated:

```text
refs/heads/master
```

may be passed to the hook.

That is why the script can inspect:

```bash
"$@"
```

where `$@` represents all arguments passed to the script.

---

# 7. Hook Used in the Challenge

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

---

# 8. Understanding the Hook Line by Line

## Shebang

```bash
#!/bin/bash
```

This tells the operating system to execute the script using Bash.

---

## Loop Through Updated Refs

```bash
for ref in "$@"
do
```

`post-update` may receive one or multiple updated refs.

`"$@"` means:

> all arguments passed to the script, preserved individually.

Example:

```text
refs/heads/master
refs/heads/develop
```

The loop checks each updated ref.

---

## Detect the `master` Branch

```bash
if [ "$ref" = "refs/heads/master" ]; then
```

Git internally represents branches as refs.

The `master` branch is:

```text
refs/heads/master
```

This condition prevents the release tag from being created when another branch is pushed.

Example:

```text
feature push
    |
    X
No release tag

master push
    |
    v
Create release tag
```

---

## Generate the Release Tag Name

```bash
TAG="release-$(date +%Y-%m-%d)"
```

`date +%Y-%m-%d` returns:

```text
2026-08-31
```

Therefore:

```text
TAG=release-2026-08-31
```

The tag format directly satisfies the challenge requirement.

---

# 9. Why Check Whether the Tag Already Exists?

The script contains:

```bash
if ! git --git-dir=/opt/media.git rev-parse -q --verify "refs/tags/$TAG" >/dev/null
```

This checks whether today's release tag already exists.

Without this check, running the hook again on the same day could produce an error similar to:

```text
fatal: tag 'release-2026-08-31' already exists
```

The check makes the hook safer and effectively idempotent for the same date.

Conceptually:

```text
Does today's tag exist?
        |
   +----+----+
   |         |
  Yes        No
   |         |
 Skip      Create tag
```

---

# 10. Creating the Tag

```bash
git --git-dir=/opt/media.git tag "$TAG" refs/heads/master
```

This creates the release tag in the bare repository and points it directly at the current `master` commit.

Equivalent concept:

```text
master --------> commit ABC
                  ^
                  |
release-date -----+
```

This is exactly what was verified at the end of the challenge.

---

# 11. What Does `--git-dir` Mean?

In a normal repository Git automatically finds:

```text
.git/
```

However, `/opt/media.git` is itself a bare repository.

Therefore we can explicitly tell Git where the repository database is:

```bash
git --git-dir=/opt/media.git ...
```

Example:

```bash
git --git-dir=/opt/media.git tag
```

This means:

> Run `git tag` using `/opt/media.git` as the Git repository.

This is very useful when administering bare repositories.

---

# 12. Why Is `chmod +x` Necessary?

Git hooks are scripts, but Git only executes hooks that are executable.

Therefore:

```bash
chmod +x /opt/media.git/hooks/post-update
```

was required.

Before:

```text
-rw-r--r-- post-update
```

Git may ignore the hook.

After:

```text
-rwxr-xr-x post-update
```

the hook can execute.

A common Git-hook troubleshooting point is therefore:

```bash
ls -l .git/hooks/
```

or, for a bare repository:

```bash
ls -l /opt/media.git/hooks/
```

---

# 13. Why Was `git update-server-info` Included?

```bash
git update-server-info
```

updates auxiliary information used by some Git transport mechanisms.

Historically, the sample `post-update` hook installed by Git often contains:

```bash
exec git update-server-info
```

It is not responsible for creating the release tag in this task, but retaining this behavior is reasonable when replacing the sample hook.

---

# 14. Why Was the Hook Created Before the Push?

The task explicitly required completing the hook setup before pushing the merged changes.

The correct order was therefore:

```text
feature
   |
   v
merge into master
   |
   v
create post-update hook
   |
   v
make hook executable
   |
   v
push master
   |
   v
hook runs
   |
   v
release tag created
```

If `master` had been pushed before configuring the hook, that push would not have triggered the required release-tag automation.

---

# 15. Why Is the Hook Stored on the Remote Repository?

The hook reacts to pushes received by `/opt/media.git`.

Therefore it must live inside:

```text
/opt/media.git/hooks/
```

and not merely inside:

```text
/usr/src/kodekloudrepos/media/.git/hooks/
```

A local hook would execute only for operations occurring in the working clone and would not represent centralized remote-side automation.

---

# 16. Verification Performed

The current release tag was listed using:

```bash
git --git-dir=/opt/media.git tag
```

Result:

```text
release-2026-08-31
```

Refs were inspected with:

```bash
git --git-dir=/opt/media.git show-ref --heads --tags
```

The current `master` commit was checked using:

```bash
git --git-dir=/opt/media.git rev-parse refs/heads/master
```

The release tag was checked using:

```bash
git --git-dir=/opt/media.git rev-parse "refs/tags/release-$(date +%Y-%m-%d)"
```

Both returned:

```text
9d0ebda9402c4300681052b215c81e590c98da2d
```

Therefore:

```text
refs/heads/master
        |
        v
9d0ebda9402c4300681052b215c81e590c98da2d
        ^
        |
refs/tags/release-2026-08-31
```

The release tag correctly references the current `master` commit.

---

# 17. Important Git Hook Characteristics

### Hooks are ordinary executable programs

A hook can be written in:

- Bash
- Python
- Perl
- Ruby
- another executable scripting language

provided the correct interpreter is available.

---

### Hook filenames matter

Git expects exact names such as:

```text
pre-commit
post-update
post-receive
```

A file named:

```text
post-update.sh
```

would normally not be treated as the `post-update` hook automatically.

---

### Sample hooks do not execute automatically

Git often provides files such as:

```text
post-update.sample
pre-commit.sample
```

These are examples.

Git executes:

```text
post-update
```

not:

```text
post-update.sample
```

---

### Hooks usually need executable permissions

Always verify:

```bash
ls -l hooks/
```

---

### Server-side hooks provide centralized control

A server-side hook runs on the repository receiving pushes.

This makes it useful for enforcing or automating repository-wide behavior.

---

# 18. `post-update` vs `post-receive`

Both are server-side hooks that execute after successful ref updates, but they receive information differently.

## `post-update`

Receives updated ref names as arguments:

```text
refs/heads/master refs/heads/develop
```

Example:

```bash
for ref in "$@"
do
    ...
done
```

---

## `post-receive`

Receives information through standard input in the format:

```text
old-value new-value ref-name
```

Example:

```text
<old-sha> <new-sha> refs/heads/master
```

`post-receive` therefore exposes more information about each update and is commonly used in advanced deployment/CI automation.

For Day 34, the challenge explicitly required `post-update`.

---

# 19. Common Git Hook Use Cases in DevOps

Git hooks can automate many workflows.

Examples include:

```text
Push to master
      |
      +--> create release tag
      |
      +--> trigger build
      |
      +--> notify team
      |
      +--> deploy application
      |
      +--> update documentation
```

Other examples:

- enforce branch policies
- reject invalid commits
- validate commit messages
- run automated tests
- trigger Jenkins jobs
- invoke CI/CD pipelines
- create build metadata
- trigger deployment scripts
- send notifications

---

# 20. Troubleshooting Git Hooks

If a hook does not execute, check:

### 1. Correct hook location

```bash
ls -l /opt/media.git/hooks/
```

### 2. Correct filename

Correct:

```text
post-update
```

Incorrect:

```text
post-update.sh
post-update.txt
post-update.sample
```

### 3. Executable permission

```bash
chmod +x /opt/media.git/hooks/post-update
```

### 4. Correct shebang

```bash
#!/bin/bash
```

### 5. Script syntax

```bash
bash -n /opt/media.git/hooks/post-update
```

### 6. Test manually

```bash
/opt/media.git/hooks/post-update refs/heads/master
```

### 7. Verify generated refs/tags

```bash
git --git-dir=/opt/media.git show-ref
```

---

# 21. Important Permission Lesson

The challenge explicitly warned:

> Do not alter existing repository or directory permissions.

Therefore broad commands such as these should be avoided:

```bash
sudo chmod -R 777 /opt/media.git
sudo chown -R natasha:natasha /opt/media.git
```

They could alter the repository configuration beyond what the task requires.

Only the new hook needed to become executable:

```bash
chmod +x /opt/media.git/hooks/post-update
```

This follows the principle of least change.

---

# 22. Key Takeaways

- Git hooks automate actions around Git events.
- `post-update` is a server-side hook.
- It executes after repository refs have been updated.
- Server-side hooks belong in the bare repository's `hooks/` directory.
- Git passes updated ref names to `post-update`.
- `refs/heads/master` represents the `master` branch internally.
- Hooks must normally be executable.
- `--git-dir` is useful when operating directly on a bare repository.
- Date-based tags can automate simple release/version workflows.
- Checking whether a tag already exists makes the hook safer.
- The hook must exist before the push that is supposed to trigger it.
- Avoid broad permission changes when a task only requires changing the hook file.

---

# Day 34 Learning Summary

Day 34 introduced **server-side Git hooks**, specifically the `post-update` hook. The task demonstrated how a bare Git repository can automatically react to a push without requiring an additional manual release step. After merging `feature` into `master`, a `post-update` script was configured in `/opt/media.git/hooks/`. The script checks whether `master` was updated and, if so, creates a release tag using the server's current date. The hook was made executable, `master` was pushed, and the generated tag was verified to reference the same commit as the updated branch. This provides a practical foundation for understanding how Git events can drive automation in CI/CD and release workflows.
