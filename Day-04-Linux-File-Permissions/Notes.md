# Linux File Permissions

## What is chmod?

`chmod` stands for **Change Mode**.

It is used to modify file and directory permissions.

---

## Linux Permission Types

| Permission | Symbol | Value |
|------------|--------|------:|
| Read | r | 4 |
| Write | w | 2 |
| Execute | x | 1 |

---

## Permission Categories

| Symbol | Meaning |
|--------|---------|
| u | User (Owner) |
| g | Group |
| o | Others |
| a | All Users |

---

## Numeric Permissions

| Number | Permission |
|---------|------------|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 3 | -wx |
| 2 | -w- |
| 1 | --x |
| 0 | --- |

---

## Common Permission Sets

| Permission | Meaning |
|------------|---------|
| 777 | Everyone has full access |
| 755 | Owner full access, others read & execute |
| 744 | Owner full access, others read only |
| 700 | Owner only |
| 644 | Standard file permission |
| 600 | Private file |

---

## chmod Examples

Grant execute permission:

```bash
chmod +x file.sh
```

Owner only:

```bash
chmod u+x file.sh
```

Everyone:

```bash
chmod a+x file.sh
```

Standard executable permission:

```bash
chmod 755 file.sh
```

Standard text file permission:

```bash
chmod 644 file.txt
```

---

## Understanding 755

```
755
```

becomes

```
Owner  : rwx
Group  : r-x
Others : r-x
```

Result

```
-rwxr-xr-x
```

---

## Interview Notes

### Difference between chmod +x and chmod 755

`chmod +x`

- Adds execute permission.
- Existing permissions remain unchanged.

`chmod 755`

- Sets permissions exactly to `rwxr-xr-x`.
- Existing permissions are overwritten.

---

## Real-World Usage

Scripts are commonly assigned:

```bash
chmod 755 script.sh
```

Configuration files often use:

```bash
chmod 644 config.conf
```

Private SSH keys should use:

```bash
chmod 600 ~/.ssh/id_rsa
```

---

## Lesson Learned

Today's task demonstrated that there is a difference between **adding permissions** and **setting permissions**.

Understanding this distinction is essential for Linux system administration and DevOps engineering.
