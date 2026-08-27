# Day 27 Commands – Git Revert

## Repository

```bash
/usr/src/kodekloudrepos/games
```

---

## 1. Navigate to the Git Repository

```bash
cd /usr/src/kodekloudrepos/games
```

---

## 2. Check Repository Status

```bash
sudo git status
```

---

## 3. View Recent Commit History

```bash
sudo git log --oneline -3
```

---

## 4. Verify the Previous Commit Message

```bash
sudo git log -1 --format="%s" HEAD^
```

Expected:

```text
initial commit
```

---

## 5. Revert the Latest Commit Without Auto-Committing

```bash
sudo git revert --no-commit HEAD
```

---

## 6. Check Revert Changes

```bash
sudo git status
```

---

## 7. Commit the Reverted Changes

```bash
sudo git commit -m "revert games"
```

---

## 8. Verify the Final Commit History

```bash
sudo git log --oneline -3
```

Observed final history:

```text
aab77cd revert games
fa3e970 add data.txt file
61c3e3b initial commit
```

---

## 9. Verify the Exact Latest Commit Message

```bash
sudo git log -1 --format="%s"
```

Expected output:

```text
revert games
```

---

## 10. Optional Detailed History Check

```bash
sudo git log --format="%h %s" -3
```

Observed:

```text
aab77cd revert games
fa3e970 add data.txt file
61c3e3b initial commit
```

---

## Complete Command Sequence

```bash
cd /usr/src/kodekloudrepos/games

sudo git status
sudo git log --oneline -3
sudo git log -1 --format="%s" HEAD^

sudo git revert --no-commit HEAD

sudo git status

sudo git commit -m "revert games"

sudo git log --oneline -3
sudo git log -1 --format="%s"
sudo git log --format="%h %s" -3
```

---

## Core Commands

```bash
sudo git revert --no-commit HEAD
sudo git commit -m "revert games"
```

---

## Result

**Day 27 Git revert challenge completed successfully ✅**
