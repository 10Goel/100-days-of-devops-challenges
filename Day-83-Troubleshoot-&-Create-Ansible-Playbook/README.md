# Day 83 — Create an Empty File Using Ansible

![Ansible](https://img.shields.io/badge/Automation-Ansible-EE0000?logo=ansible&logoColor=white) ![Linux](https://img.shields.io/badge/Platform-Linux-333333?logo=linux&logoColor=white) ![Status](https://img.shields.io/badge/KodeKloud-PASSED-2ea44f)

## Overview

Completed **KodeKloud 100 Days of DevOps — Day 83**, configuring an existing Ansible inventory to target **App Server 3 (`stapp03`)** and writing a playbook that creates an empty file at `/tmp/file.txt` on the remote machine.

The main troubleshooting point was an initial SSH host-key verification failure while Ansible used password-based authentication. Trusting the remote host key through a verified interactive SSH connection resolved the host-key obstacle.

## Requirements

| Item | Required value |
|---|---|
| Control node | Jump host (`thor`) |
| Inventory | `/home/thor/ansible/inventory` |
| Target server | App Server 3 — `stapp03` |
| SSH account | `banner` |
| Playbook | `/home/thor/ansible/playbook.yml` |
| Target file | `/tmp/file.txt` |
| File expectation | Exists, regular, empty |
| Validation invocation | `ansible-playbook -i inventory playbook.yml` |

## Workflow

```text
Jump host (thor)
  /home/thor/ansible/
    ├── inventory        -> stapp03, SSH connection variables
    └── playbook.yml     -> file module, state: touch
          │
          └── SSH → App Server 3 (banner@stapp03)
                          └── /tmp/file.txt (zero bytes)
```

## Implementation

### 1. Inspect and update the existing inventory

```bash
cd /home/thor/ansible
ls -la
vi inventory
chmod 600 inventory
ansible-inventory -i inventory --graph
```

An example INI entry (replace placeholders with lab-specific connection details):

```ini
[app_servers]
stapp03 ansible_host=stapp03 ansible_connection=ssh ansible_user=banner ansible_password='REDACTED_LAB_PASSWORD'
```

**Security:** Do not commit the inventory with plaintext credentials. Prefer SSH-key authentication or Ansible Vault outside this temporary lab.

### 2. Create the playbook

```yaml
---
- name: Create an empty file on App Server 3
  hosts: stapp03
  become: false
  gather_facts: false

  tasks:
    - name: Create empty file at /tmp/file.txt
      ansible.builtin.file:
        path: /tmp/file.txt
        state: touch
        mode: '0644'
```

Using `hosts: stapp03` ensures that this play selects the specific inventory host. Privilege escalation is unnecessary for ordinary file creation under `/tmp` by the SSH account.

### 3. Fix SSH trust and verify remote execution

The initial ping failed because the host key was not yet trusted and `sshpass` could not handle the interactive host-key prompt.

```bash
ssh banner@stapp03
# Verify host-key fingerprint against trusted lab details before accepting.
exit
ssh-keygen -F stapp03
ansible -i inventory stapp03 -m ping
```

After the interactive SSH connection, `stapp03` was added to the jump host's `known_hosts`. This is distinct from authenticating with the remote user's SSH password.

### 4. Validate and run

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
ansible -i inventory stapp03 -m stat -a 'path=/tmp/file.txt'
```

The `stat` output should report `exists: true`, `isreg: true`, and `size: 0`.

## Result

**KodeKloud challenge: PASSED** (confirmed following the successful troubleshooting and execution sequence).

The screenshots documented the inventory graph resolving `stapp03` and the SSH host-key failure that was subsequently addressed. The final task success was confirmed; no fabricated play recap counts or exact final log lines are asserted here.

## What I Learned

- How an **INI inventory** maps inventory hostnames to real managed nodes.
- How Ansible uses **SSH authentication** and **host-key trust** as separate mechanisms.
- How `ansible.builtin.file` with `state: touch` creates a missing file without truncating an existing one.
- How to test inventory parsing, connection, playbook syntax, and remote file state independently.
- Why credentials must be excluded from GitHub repositories.
- [`Notes.md`](Notes.md) — detailed Ansible concepts, troubleshooting, and interview questions.

