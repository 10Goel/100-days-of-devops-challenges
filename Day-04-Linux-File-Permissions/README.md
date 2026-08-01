# 🔐 Day 4 - Linux File Permissions

## 🎯 Objective

Grant executable permissions to the script `/tmp/xfusioncorp.sh` on **Application Server 3** and ensure that all users can execute it.

---

# 📝 Problem Statement

The system administration team deployed a new Bash script for automating backup processes.

However, the script lacked executable permissions.

The task was to:

- Connect to App Server 3.
- Locate the script.
- Grant executable permissions.
- Verify the updated permissions.

---

# 🖥️ Lab Environment

| Server | Hostname | Login User |
|---------|----------|------------|
| Application Server 3 | stapp03 | banner |

---

# 📚 Concepts Learned

- Linux File Permissions
- chmod
- Symbolic Permissions
- Numeric Permissions
- File Ownership
- Execute Permission
- SSH
- sudo

---

# 🛠️ Commands Used

## Connect to the Server

```bash
ssh banner@stapp03
```

---

## Become Root

```bash
sudo su -
```

---

## Check Existing Permissions

```bash
ls -l /tmp/xfusioncorp.sh
```

Output

```text
----------
```

---

## Grant Execute Permissions

Initially I tried:

```bash
chmod a+x /tmp/xfusioncorp.sh
```

This changed the permissions to:

```text
--x--x--x
```

Although technically executable, the lab validator expected the script to have standard executable permissions.

The final successful command was:

```bash
chmod 755 /tmp/xfusioncorp.sh
```

---

## Verify

```bash
ls -l /tmp/xfusioncorp.sh
```

Output

```text
-rwxr-xr-x
```

---

# 🔎 Understanding chmod 755

The numeric permission:

```
755
```

means

```
Owner : 7 = rwx
Group : 5 = r-x
Others: 5 = r-x
```

Result:

```
-rwxr-xr-x
```

Owner has full access.

Everyone else can read and execute.

---

# 🔎 Understanding chmod a+x

```
chmod a+x file
```

adds execute permission to

- Owner
- Group
- Others

without modifying existing read or write permissions.

Example

Before

```
-rw-r--r--
```

After

```
-rwxr-xr-x
```

However, if a file initially has

```
----------
```

then

```
chmod a+x
```

produces

```
--x--x--x
```

because execute permission is simply added.

---

# ⚠️ Challenge Faced

Initially I assumed

```bash
chmod a+x
```

would satisfy the task.

The file originally had no permissions:

```
----------
```

Therefore,

```bash
chmod a+x
```

produced

```
--x--x--x
```

Although this technically granted execute permission, the KodeKloud validator expected the conventional executable permission set:

```
-rwxr-xr-x
```

Using

```bash
chmod 755 /tmp/xfusioncorp.sh
```

resolved the issue successfully.

---

# 💡 Key Takeaways

- chmod modifies Linux file permissions.
- chmod a+x adds execute permission only.
- chmod 755 explicitly sets permissions.
- Numeric permissions are often preferred in production.
- Always verify permissions using:

```bash
ls -l
```

---

# 🚀 Skills Practised

- Linux Administration
- File Permission Management
- chmod
- SSH
- Root Privileges
- Bash Scripts

---

# 📚 References

- Linux chmod Manual
- Linux File Permission Documentation
- GNU Coreutils Documentation

---

## ✅ Status

**Task Completed Successfully**
