# Day 21 – Notes

## 1. Git

Git is a distributed version-control system used to track changes in source code and collaborate on software projects.

Check whether Git is installed:

```bash
git --version
```

---

## 2. Installing Git with YUM

On RPM-based systems that use `yum`, Git can be installed with:

```bash
sudo yum install git -y
```

The `-y` option automatically confirms the installation.

---

## 3. Normal vs Bare Git Repository

### Normal Repository

A normal Git repository contains both:

- Git metadata in `.git/`
- A working directory containing project files

Example:

```text
project/
├── .git/
├── app.py
└── README.md
```

It is normally created with:

```bash
git init
```

### Bare Repository

A bare repository contains Git repository data but no working directory.

Example:

```text
ecommerce.git/
├── HEAD
├── config
├── hooks/
├── info/
├── objects/
└── refs/
```

It is created with:

```bash
git init --bare /opt/ecommerce.git
```

---

## 4. Why Use a Bare Repository?

Bare repositories are commonly used as **central/shared repositories** on servers.

Developers can use such a repository as a remote:

```bash
git remote add origin <repository>
```

and then push their changes:

```bash
git push origin main
```

A bare repository is appropriate because the server-side repository does not need to maintain a checked-out working copy.

---

## 5. Repository Verification

The following command checks whether the specified Git directory is bare:

```bash
sudo git --git-dir=/opt/ecommerce.git rev-parse --is-bare-repository
```

Expected result:

```text
true
```

This confirms that `/opt/ecommerce.git` is a bare Git repository.

---

## 6. Important Difference

Do not confuse:

```bash
git init /opt/ecommerce.git
```

with:

```bash
git init --bare /opt/ecommerce.git
```

The first creates a normal repository with a working tree, while the second creates the server-side bare repository required by this challenge.

---

## 7. Linux Permissions

Because `/opt` is normally owned by `root`, administrative privileges may be required:

```bash
sudo git init --bare /opt/ecommerce.git
```

The repository can be inspected with:

```bash
sudo ls -la /opt/ecommerce.git
```

---

## 8. Day 21 Takeaways

- Git can be installed using the system package manager.
- A bare repository is designed for centralized/server-side Git usage.
- `--bare` is the important option when creating such a repository.
- `rev-parse --is-bare-repository` provides a reliable verification.
- Linux permissions matter when creating repositories under protected directories such as `/opt`.

## ✅ Completion

The required repository was successfully created at:

```text
/opt/ecommerce.git
```

Day 21 challenge: **Completed successfully.**
