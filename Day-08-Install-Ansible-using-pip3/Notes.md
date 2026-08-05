# Day 8 Notes - Installing Ansible using pip3

# What is Ansible?

Ansible is an open-source Configuration Management and Automation tool used to automate:

- Server Configuration
- Application Deployment
- Software Installation
- Infrastructure Provisioning
- System Administration Tasks

Instead of logging into hundreds of servers individually, Ansible allows administrators to automate operations from a single control machine.

---

# Why Do We Need Ansible?

Imagine managing 100 Linux servers.

Without Ansible:

- SSH into Server 1
- Install package
- SSH into Server 2
- Install package
- SSH into Server 3
- Repeat...

This approach is slow and error-prone.

With Ansible:

Write the task once.

Run it on all servers simultaneously.

Automation saves time and ensures consistency.

---

# Basic Ansible Architecture

```

                 Control Node
                (Jump Host)

                     │
     ┌───────────────┼───────────────┐
     │               │               │

Managed Node 1   Managed Node 2   Managed Node 3

```

The Control Node executes Ansible commands.

Managed Nodes receive and execute the tasks.

---

# Why Was pip3 Used?

There are two common installation methods.

## Method 1

```bash
yum install ansible
```

Pros

- Simple
- Uses operating system repositories

Cons

- May install an older version

---

## Method 2

```bash
pip3 install ansible==4.8.0
```

Pros

- Install exact versions
- Access latest releases
- Independent of OS repositories

Cons

- Requires Python and pip3

The challenge specifically required pip3.

---

# What Does "ansible==4.8.0" Mean?

The double equals operator specifies the exact package version.

Example:

```bash
pip3 install ansible==4.8.0
```

This installs only version 4.8.0.

---

# Verifying Installation

```bash
ansible --version
```

Displays:

- Installed version
- Python version
- Executable location

---

# Locating the Binary

```bash
which ansible
```

Example output

```

/usr/local/bin/ansible

```

This tells us where the executable is stored.

---

# What is PATH?

PATH is an environment variable containing directories that the shell searches for executable programs.

Example

```

/usr/local/bin
/usr/bin
/bin

```

When you type

```bash
ansible
```

Linux searches each directory listed in PATH until it finds the executable.

---

# What Does "Globally Available" Mean?

It does NOT mean installing Ansible for every user.

It means the executable is located in a directory included in the system-wide PATH.

For example

```

/usr/local/bin/ansible

```

Since `/usr/local/bin` is in PATH, every user can simply run:

```bash
ansible --version
```

without specifying the full path.

---

# When Would Extra Configuration Be Needed?

Suppose Ansible is installed in

```

/root/.local/bin

```

Only the root user can access it.

Normal users would receive:

```

ansible: command not found

```

Possible solutions:

- Add the directory to the global PATH
- Create a symbolic link in `/usr/local/bin`

Example:

```bash
ln -s /root/.local/bin/ansible /usr/local/bin/ansible
```

---

# PyPI

PyPI (Python Package Index) is the official repository for Python packages.

pip3 downloads packages directly from PyPI.

---

# Important Commands

Check Python

```bash
python3 --version
```

Check pip

```bash
pip3 --version
```

Install package

```bash
pip3 install ansible==4.8.0
```

Verify installation

```bash
ansible --version
```

Locate executable

```bash
which ansible
```

Display PATH

```bash
echo $PATH
```

---

# Interview Questions

## What is Ansible?

Ansible is an agentless configuration management and automation tool used for provisioning, application deployment, and infrastructure automation over SSH.

---

## Why is Ansible called agentless?

Because no agent needs to be installed on managed nodes.

It communicates using SSH.

---

## What is a Control Node?

The machine where Ansible is installed and from which automation tasks are executed.

---

## What are Managed Nodes?

The servers controlled by Ansible.

---

## Why use pip3 instead of yum?

pip3 allows installation of specific package versions and is independent of operating system repositories.

---

## What does "globally available binary" mean?

The executable is stored in a directory that exists in the system PATH, allowing every user to execute the command without specifying its full path.

---

# Key Takeaways

- Learned the role of Configuration Management.
- Understood the architecture of Ansible.
- Installed Ansible using pip3.
- Verified installation.
- Learned how PATH determines executable availability.
- Understood the meaning of globally accessible binaries.
- Gained exposure to Python package management using pip3.
