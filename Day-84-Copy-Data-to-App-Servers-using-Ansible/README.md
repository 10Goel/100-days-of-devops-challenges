# Day 84 — Distribute a File to Multiple Servers with Ansible

**KodeKloud 100 Days of DevOps · Day 84 · Challenge PASSED**

## Objective

Copy the existing `/usr/src/sysops/index.html` file from the **jump host** (Ansible control node) into `/opt/sysops/` on **all three application servers**, using an INI inventory and a reusable Ansible playbook.

| Requirement | Exact value |
|---|---|
| Inventory | `/home/thor/ansible/inventory` |
| Playbook | `/home/thor/ansible/playbook.yml` |
| Source (control node) | `/usr/src/sysops/index.html` |
| Target files | `/opt/sysops/index.html` on all application servers |
| Managed hosts | `stapp01`, `stapp02`, `stapp03` |
| Validation invocation | `ansible-playbook -i inventory playbook.yml` |

## Architecture

```text
Jump host (thor): /usr/src/sysops/index.html
                   |  Ansible over SSH
          +--------+--------+
          |        |        |
       stapp01   stapp02   stapp03
        tony      steve    banner
          |        |        |
       /opt/sysops/index.html on each server
```

The `copy` module reads the source file on the Ansible **control node** and transfers it to the managed hosts. A preceding `file` task creates the destination directory when missing.

## Inventory configuration

Create `/home/thor/ansible/inventory` (passwords are placeholders, never publish a populated inventory):

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password='APP1_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD'
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='APP2_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD'
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password='APP3_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD'
```

SSH and sudo passwords are not necessarily the same. Use credentials from the current authorized lab, then secure the inventory:

```bash
chmod 600 /home/thor/ansible/inventory
ansible-inventory -i inventory --graph
```

## Playbook

Create `/home/thor/ansible/playbook.yml`:

```yaml
---
- name: Distribute sysops HTML file to all application servers
  hosts: app_servers
  become: true
  gather_facts: false

  tasks:
    - name: Ensure destination directory exists
      ansible.builtin.file:
        path: /opt/sysops
        state: directory
        mode: '0755'

    - name: Copy HTML file from jump host
      ansible.builtin.copy:
        src: /usr/src/sysops/index.html
        dest: /opt/sysops/index.html
        mode: '0644'
```

## Execute

```bash
cd /home/thor/ansible
ansible -i inventory app_servers -m ping
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The last command is the exact validation command required by the task; it needs no additional CLI connection flags. A successful recap has `failed=0` and `unreachable=0` for **each** server. Exact `changed` counts depend on existing state.

## Verification

```bash
ansible -i inventory app_servers -b -m stat -a 'path=/opt/sysops/index.html'
sha256sum /usr/src/sysops/index.html
ansible -i inventory app_servers -b -m command -a 'sha256sum /opt/sysops/index.html'
```

Confirm file existence, regular-file type, and matching SHA-256 hashes across the controller and all three servers. Optionally rerun the playbook to observe idempotency.

## Troubleshooting overview

| Symptom | Fix or investigation |
|---|---|
| `UNREACHABLE` | Check SSH connectivity, usernames, and credentials |
| Host-key error with `sshpass` | Verify the host fingerprint and accept it interactively from the jump host |
| Sudo prompt / missing password | Configure authorized `ansible_become_password` |
| Permission denied under `/opt` | Check `become: true` and sudo policy |
| Source file not found | Verify `/usr/src/sysops/index.html` exists on the jump host |
| Missing server | Inspect `ansible-inventory --graph` |

## Learning outcomes

Multi-host inventories, SSH authentication versus host-key verification, privilege escalation, `ansible.builtin.file`, `ansible.builtin.copy`, idempotency and checksum-based validation.

See [Commands.md](Commands.md) for the full runbook and [Notes.md](Notes.md) for concepts and interview questions.

> Documentation describes the verified, passed challenge and reproducible procedure, not an invented terminal transcript. Never commit plaintext credentials to public GitHub.
