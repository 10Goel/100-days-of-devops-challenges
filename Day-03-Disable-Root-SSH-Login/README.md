# 🔐 Day 3 - Disable Direct SSH Root Login

## 🎯 Objective

Disable direct SSH root login on all Application Servers by updating the SSH daemon configuration.

---

# 📝 Problem Statement

Following security audits, direct SSH login for the **root** user had to be disabled on all application servers.

The task involved:

- Connecting to each application server using SSH.
- Editing the SSH daemon configuration.
- Disabling root login.
- Restarting the SSH service.
- Verifying the configuration.

---

# 🖥️ Lab Environment

| Server | Hostname | Login User |
|---------|----------|------------|
| Application Server 1 | stapp01 | tony |
| Application Server 2 | stapp02 | steve |
| Application Server 3 | stapp03 | banner |

---

# 📚 Concepts Learned

- SSH (Secure Shell)
- SSH Daemon (sshd)
- SSH Server Configuration
- Root Login Security
- Linux Service Management
- systemctl
- Configuration Files
- Cybersecurity Best Practices

---

# 🛠️ Commands Used

## Connect to each application server

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

---

## Become root

```bash
sudo su -
```

---

## Edit SSH configuration

```bash
vi /etc/ssh/sshd_config
```

Update:

```text
PermitRootLogin no
```

---

## Restart SSH Service

```bash
systemctl restart sshd
```

---

## Verify

```bash
grep "^PermitRootLogin" /etc/ssh/sshd_config
```

Expected Output

```text
PermitRootLogin no
```

---

# 🔎 Understanding the Configuration

The SSH daemon configuration file is located at:

```text
/etc/ssh/sshd_config
```

The directive

```text
PermitRootLogin no
```

prevents users from logging into the server directly as the **root** user.

Instead, administrators should:

1. Login using a normal user account.
2. Elevate privileges using:

```bash
sudo su -
```

This improves accountability and reduces security risks.

---

# 🔒 Why Disabling Root Login is Important

Disabling direct root login:

- Prevents brute-force attacks targeting the root account.
- Improves system security.
- Encourages the principle of least privilege.
- Provides better auditing since administrators log in using individual accounts before elevating privileges.

---

# ⚠️ Challenge Faced

Initially, I modified the SSH configuration correctly but forgot to restart the SSH daemon.

As a result, the new configuration was **not applied**, and the task validation failed on Application Server 2.

After restarting the SSH service using:

```bash
systemctl restart sshd
```

the updated configuration was loaded successfully, and the task passed.

This reinforced the importance of restarting or reloading services after modifying their configuration files.

---

# 💡 Key Takeaways

- SSH configuration is managed through `/etc/ssh/sshd_config`.
- Editing a configuration file alone is not sufficient.
- Services must be restarted or reloaded for changes to take effect.
- Direct root login is considered a security risk.
- Using `sudo` provides better accountability and follows Linux security best practices.

---

# 🚀 Skills Practised

- SSH
- Linux System Administration
- SSH Security
- Service Management
- Configuration Management
- Cybersecurity Fundamentals

---

# 📚 References

- Linux `sshd_config` Manual
- Linux `systemctl` Manual
- OpenSSH Documentation

---

## ✅ Status

**Task Completed Successfully**
