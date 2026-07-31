# Notes

## What is SSH?

SSH (Secure Shell) is a secure network protocol used to remotely access Linux systems.

---

## What is sshd?

`sshd` is the SSH server daemon that listens for incoming SSH connections.

---

## What is `/etc/ssh/sshd_config`?

It is the main configuration file for the SSH server.

Changes made here control:

- Authentication
- Port Number
- Root Login
- Password Authentication
- Key Authentication
- Timeouts

---

## What is `PermitRootLogin`?

This directive determines whether the root user can log in directly through SSH.

Common values:

```
yes
no
prohibit-password
forced-commands-only
```

For production servers:

```
PermitRootLogin no
```

is recommended.

---

## Important Lesson

Whenever a configuration file is modified:

1. Save the file.
2. Restart or reload the corresponding service.
3. Verify the changes.

This is a common Linux system administration workflow.
