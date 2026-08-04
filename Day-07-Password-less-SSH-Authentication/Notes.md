# Day 07 Notes - Password-less SSH Authentication

# Introduction

SSH (Secure Shell) is a secure protocol used to remotely access Linux servers over encrypted communication.

Normally, SSH authentication requires a username and password.

Example:

```
thor -------- Password --------> stapp01
```

Although secure, password authentication is not suitable for automation because scripts cannot manually enter passwords.

To solve this problem, Linux provides **SSH Key-Based Authentication**.

---

# What is Password-less SSH?

Password-less SSH uses a pair of cryptographic keys instead of passwords.

The two keys are:

- Private Key
- Public Key

Authentication succeeds when the server verifies that the client owns the matching private key.

---

# SSH Key Pair

Running

```bash
ssh-keygen
```

creates two files.

```
~/.ssh/

id_rsa
id_rsa.pub
```

## id_rsa

Private Key

Characteristics:

- Secret
- Never copied to servers
- Never shared
- Used for authentication

Think of it as your house key.

---

## id_rsa.pub

Public Key

Characteristics:

- Safe to share
- Installed on remote servers
- Stored in

```
~/.ssh/authorized_keys
```

Think of it as the lock installed on the server.

---

# Authentication Process

Without SSH Keys

```
Client
   |
Password
   |
Server
```

Every login requires typing a password.

---

With SSH Keys

```
Client
│
├── Private Key
│
▼

Server
│
└── authorized_keys
```

The server checks whether the client's private key matches the installed public key.

If it matches:

Access granted.

No password required.

---

# Directory Structure

```
~/.ssh/

id_rsa
id_rsa.pub
authorized_keys
known_hosts
config
```

### id_rsa

Private authentication key.

---

### id_rsa.pub

Public key copied to servers.

---

### authorized_keys

Stores all public keys allowed to log in.

Located on remote servers.

---

### known_hosts

Stores fingerprints of previously connected servers.

Prevents Man-in-the-Middle attacks.

---

### config

Optional configuration file for SSH.

Can define aliases and connection settings.

---

# ssh-keygen

Generates an SSH key pair.

Example

```bash
ssh-keygen -t rsa -b 4096
```

Options

```
-t
```

Key algorithm.

Examples:

- rsa
- ed25519
- ecdsa

---

```
-b
```

Key size.

Example:

```
4096
```

Larger keys provide stronger security but require slightly more computation.

---

# ssh-copy-id

Copies the public key to a remote server.

Example

```bash
ssh-copy-id tony@stapp01
```

Internally this command:

- Creates `.ssh` directory if missing
- Creates `authorized_keys`
- Appends your public key
- Sets proper permissions

---

# Login After Configuration

Before

```bash
ssh tony@stapp01
```

Prompt

```
Password:
```

After configuration

```bash
ssh tony@stapp01
```

Direct login

```
Welcome
```

No password required.

---

# Benefits of Password-less SSH

- Faster server login
- Better security
- Eliminates password typing
- Required for automation
- Enables CI/CD pipelines
- Supports configuration management tools
- Essential for DevOps practices

---

# Real-World DevOps Use Cases

Password-less SSH is commonly used by:

- Jenkins
- GitHub Actions
- GitLab CI
- Ansible
- Terraform
- Kubernetes administration
- Shell automation scripts
- Backup servers
- Monitoring systems

---

# Best Practices

- Never share the private key (`id_rsa`).
- Protect the `.ssh` directory with appropriate permissions.
- Use strong key sizes or modern algorithms like `ed25519`.
- Use passphrases for sensitive environments.
- Rotate SSH keys periodically.
- Remove unused public keys from `authorized_keys`.

---

# Interview Questions

### What is SSH?

SSH is a secure protocol used to remotely connect to Linux systems over encrypted communication.

---

### What is Password-less SSH?

It is SSH authentication using cryptographic key pairs instead of passwords.

---

### Difference between Public Key and Private Key?

| Public Key | Private Key |
|------------|-------------|
| Shared | Secret |
| Installed on servers | Stored locally |
| Safe to distribute | Never share |
| Used for verification | Used for authentication |

---

### Which command generates SSH keys?

```bash
ssh-keygen
```

---

### Which command copies the public key?

```bash
ssh-copy-id
```

---

### Where are SSH keys stored?

```
~/.ssh/
```

---

### Where are allowed public keys stored on a server?

```
~/.ssh/authorized_keys
```

---

### Why is Password-less SSH important in DevOps?

It enables secure, automated communication between systems without requiring manual password entry, making it essential for CI/CD pipelines, configuration management, remote deployments, and infrastructure automation.
