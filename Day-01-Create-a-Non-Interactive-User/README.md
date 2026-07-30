# Day 01 - Create a Non-Interactive User

## 🎯 Objective

Create a Linux user with a non-interactive login shell on the target application server.

---

## 📚 Concepts Learned

- SSH (Secure Shell)
- Jump Host
- Application Server
- Linux User Management
- Root User
- sudo
- Non-Interactive Login Shell

---

## 🛠️ Commands Used

### Connect to the target server

```bash
ssh tony@stapp01
```

### Become root

```bash
sudo su -
```

### Create the user

```bash
useradd -s /usr/sbin/nologin john
```

---

## ✅ Verification

```bash
id john
```

or

```bash
grep john /etc/passwd
```

---

## 🧠 What I Learned

- Every Linux server has its own user database.
- The Jump Host is only a gateway to other servers.
- SSH allows secure remote login.
- The `nologin` shell prevents interactive logins.
- Always ensure you're connected to the correct server before making changes.

---

## 🚀 Skills Practised

- SSH
- Linux User Management
- sudo
- System Administration
