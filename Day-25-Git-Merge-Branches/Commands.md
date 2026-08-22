# Day 25 - Commands

## Connect to the Storage Server

```bash
ssh natasha@ststor01
```

## Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/news
```

## Fix Git Dubious Ownership Warning

```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/news
```

## Inspect Repository

```bash
git status
git branch
git remote -v
```

## Switch to Master

```bash
sudo git checkout master
```

## Create the Nautilus Branch

```bash
sudo git checkout -b nautilus
```

## Copy the Required File

```bash
sudo cp /tmp/index.html .
```

## Verify the File

```bash
ls -l index.html
```

## Check Git Status

```bash
sudo git status
```

## Stage the File

```bash
sudo git add index.html
```

## Commit the File

```bash
sudo git commit -m "Add index.html"
```

## View Commit History

```bash
sudo git log --oneline --decorate --graph --all
```

## Switch Back to Master

```bash
sudo git checkout master
```

## Merge Nautilus into Master

```bash
sudo git merge nautilus
```

## Push Nautilus to Origin

```bash
sudo git push origin nautilus
```

## Push Master to Origin

```bash
sudo git push origin master
```

## Final Verification

```bash
sudo git status
sudo git branch
sudo git log --oneline --decorate --graph --all
sudo git ls-remote --heads origin
```

## Complete Command Flow

```bash
ssh natasha@ststor01

cd /usr/src/kodekloudrepos/news

git config --global --add safe.directory /usr/src/kodekloudrepos/news

git status
git branch
git remote -v

sudo git checkout master
sudo git checkout -b nautilus

sudo cp /tmp/index.html .

sudo git add index.html
sudo git commit -m "Add index.html"

sudo git checkout master
sudo git merge nautilus

sudo git push origin nautilus
sudo git push origin master

sudo git status
sudo git branch
sudo git log --oneline --decorate --graph --all
sudo git ls-remote --heads origin
```

## Expected Final Remote Branches

```text
refs/heads/master
refs/heads/nautilus
```
