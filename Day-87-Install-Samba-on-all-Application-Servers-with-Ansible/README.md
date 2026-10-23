# Day 87 — Install Samba on All Application Servers with Ansible

![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-yum%20module-blue)

## Overview

Automate installation of the `samba` RPM package across all three Nautilus application servers using an Ansible playbook executed by `thor` on the jump host. The task requires the **Ansible yum module**, not an ad-hoc SSH loop or manual package installation.

## Task requirements

| Item | Value |
|---|---|
| Control node | Jump Host (`thor`) |
| Inventory | `/home/thor/playbook/inventory` |
| Playbook | `/home/thor/playbook/playbook.yml` |
| Managed hosts | `stapp01`, `stapp02`, `stapp03` |
| SSH users | `tony`, `steve`, `banner`, respectively |
| Package | `samba` |
| Module | `ansible.builtin.yum` |
| Execution | `ansible-playbook -i inventory playbook.yml` |
| Challenge outcome | **PASSED** (reported after submission) |

## Architecture

```text
                  Jump host / thor
       /home/thor/playbook/inventory
       /home/thor/playbook/playbook.yml
                         |
             Ansible over SSH + sudo
            /            |           \
     stapp01          stapp02        stapp03
    user: tony       user: steve    user: banner
       |                |              |
       +------------ samba package -----+
```

## 1. Inventory

Use an INI inventory like the following at `/home/thor/playbook/inventory`. **Placeholders are intentional**; never commit real lab credentials.

```ini
[app_servers]
stapp01 ansible_user=tony ansible_password='APP1_SSH_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD'
stapp02 ansible_user=steve ansible_password='APP2_SSH_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD'
stapp03 ansible_user=banner ansible_password='APP3_SSH_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD'
```

The connection user and privilege-escalation credentials are separate concerns; values may happen to match in a lab but should not be assumed to match.

Restrict permissions:

```bash
chmod 600 /home/thor/playbook/inventory
```

## 2. Playbook

Create `/home/thor/playbook/playbook.yml`:

```yaml
---
- name: Install Samba across Nautilus application servers
  hosts: app_servers
  become: true
  gather_facts: true

  tasks:
    - name: Ensure Samba package is installed
      ansible.builtin.yum:
        name: samba
        state: present
```

`become: true` grants the package manager sufficient privileges, while `state: present` ensures installation without forcing upgrades of existing packages.

## 3. Execute

From the jump host:

```bash
cd /home/thor/playbook
ansible-inventory -i inventory --graph
ansible -i inventory app_servers -m ping
ansible -i inventory app_servers -b -m command -a 'id -u'
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Before password-based SSH, confirm host keys against trusted lab information and accept them interactively if necessary. Do not disable verification simply to bypass an error.

## 4. Verify installation

```bash
ansible -i inventory app_servers -m command -a 'rpm -q samba'
ansible-playbook -i inventory playbook.yml
```

Successful execution should show each server with `failed=0` and `unreachable=0`. `rpm -q samba` should print an installed package version for each host. On rerun, the playbook normally reports `changed=0` if Samba is already installed and no other changes are needed.

> **Evidence note:** The user's completed challenge was confirmed as passed. Example command output is illustrative, not a fabricated transcript of actual execution.

## What this exercise teaches

- Inventory groups allow one playbook to target multiple servers.
- Package installation typically requires privilege escalation.
- Ansible modules describe desired state rather than imperative shell commands.
- `state: present` offers repeatable, idempotent package management.
- SSH authentication and sudo authentication are separate mechanisms.
- A successful play recap must be complemented with package verification.

## Security

No real passwords, SSH private keys, or confidential infrastructure tokens are included here. For production, use Ansible Vault or another approved secret manager and prefer suitable SSH-key authentication.
