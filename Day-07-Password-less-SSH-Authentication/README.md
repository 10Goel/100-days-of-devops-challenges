# Day 07 - Password-less SSH Authentication

## 📌 Challenge Objective

Configure password-less SSH authentication from the **jump host** (`thor`) to all application servers using their respective users.

### Target Connections

| Source | Destination |
|---------|-------------|
| thor@jump-host | tony@stapp01 |
| thor@jump-host | steve@stapp02 |
| thor@jump-host | banner@stapp03 |

---

## Problem Statement

The operations team has multiple automation scripts running from the jump host. These scripts require remote access to application servers without manual password entry.

To enable secure automation, SSH Key-Based Authentication must be configured.

---

## Solution

Generated an SSH key pair on the jump host and copied the public key to each application server using `ssh-copy-id`.

After configuration, all servers accepted SSH connections without prompting for passwords.

---

## Verification

Successfully logged in without password:

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

No password prompt appeared.

---

## Concepts Learned

- SSH Authentication
- Public Key Cryptography
- Private Key vs Public Key
- Password-less SSH Login
- ssh-keygen
- ssh-copy-id
- authorized_keys
- ~/.ssh directory
- DevOps Automation Fundamentals

---

## Result

✅ Password-less SSH configured successfully

✅ Challenge completed successfully
