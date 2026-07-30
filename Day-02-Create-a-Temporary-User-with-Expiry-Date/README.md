# Day 02 - Create a Temporary User with Expiry Date

> **Platform:** KodeKloud Engineer  
> **Category:** Linux User Management  
> **Difficulty:** Beginner

---

# 📌 Challenge Overview

As part of the temporary assignment to the **Nautilus** project, a developer named **jim** required access to **Application Server 2** for a limited duration.

The objective was to create a temporary Linux user account and configure an account expiration date so that the account would automatically become inactive after the project period.

---

# 🎯 Objective

- Create a Linux user named **jim**
- Configure the account expiration date as **28 March 2027**
- Verify that the account expiry has been successfully configured

---

# 🖥️ Environment

| Property | Value |
|----------|-------|
| Platform | KodeKloud Engineer |
| Operating System | Linux |
| Server | Application Server 2 |
| User Created | jim |
| Expiry Date | 2027-03-28 |

---

# 🛠️ Commands Used

### Connect to the Server

```bash
ssh steve@stapp02
```

### Switch to Root User

```bash
sudo su -
```

### Create the User

```bash
useradd -e 2027-03-28 jim
```

### Verify the Expiry Date

```bash
chage -l jim
```

---

# 🔍 Command Explanation

## useradd

Creates a new Linux user account.

### -e

Specifies the account expiration date.

### 2027-03-28

The account will automatically expire on **28 March 2027**.

### jim

Name of the user account.

---

# ✅ Verification

```bash
chage -l jim
```

Example Output

```text
Last password change                                    : never
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Mar 28, 2027
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

---

# 💡 Key Learnings

- Learned how to create Linux users.
- Learned how to configure temporary user accounts.
- Understood the purpose of the `-e` option.
- Learned how to verify account expiry using `chage`.
- Understood why organizations use temporary accounts.

---

# 🌍 Real-World Use Cases

Temporary user accounts are commonly used for:

- Contractors
- Interns
- Freelancers
- Third-party Vendors
- Auditors
- Temporary Developers
- Client Access

Instead of manually deleting these accounts after the project ends, Linux automatically disables them after the configured expiry date.

---

# 📚 Commands Learned

| Command | Purpose |
|----------|---------|
| `useradd` | Create a new user |
| `useradd -e` | Create a user with an expiry date |
| `chage -l` | Display password and account ageing information |

---

# 🧠 Skills Practiced

- Linux User Management
- Linux Administration
- User Lifecycle Management
- Account Expiration
- Linux Command Line
- User Verification

---

# 📖 References

- `man useradd`
- `man chage`

---

# 🚀 Status

**Challenge Completed Successfully ✅**
