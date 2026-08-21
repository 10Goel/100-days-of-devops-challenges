# DevOps Day 24 — Commands

## Connect to Storage Server

```bash
ssh natasha@ststor01
```

## Navigate to Repository

```bash
cd /usr/src/kodekloudrepos/games
```

## Confirm Current Location

```bash
pwd
```

Expected:

```text
/usr/src/kodekloudrepos/games
```

## Check Repository Status

```bash
sudo git status
```

## List Existing Branches

```bash
sudo git branch
```

## Switch to `master`

```bash
sudo git checkout master
```

## Create the Required Branch

```bash
sudo git checkout -b xfusioncorp_games
```

## Verify Branch Creation

```bash
sudo git branch
```

Expected:

```text
  master
* xfusioncorp_games
```

## Verify Working Tree

```bash
sudo git status
```

Expected:

```text
On branch xfusioncorp_games
nothing to commit, working tree clean
```

## Verify Branch Base

```bash
sudo git rev-parse master
sudo git rev-parse xfusioncorp_games
```

Both commands should initially return the same commit hash.

## Optional Commit History Check

```bash
sudo git log --oneline --decorate -5
```

## Complete Command Flow

```bash
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/games

sudo git status
sudo git branch

sudo git checkout master
sudo git checkout -b xfusioncorp_games

sudo git branch
sudo git status

sudo git rev-parse master
sudo git rev-parse xfusioncorp_games
```

## Commands Not Required

Do not run these unless explicitly required by a task:

```bash
git add
git commit
git push
```

The Day 24 challenge only required local branch creation.
