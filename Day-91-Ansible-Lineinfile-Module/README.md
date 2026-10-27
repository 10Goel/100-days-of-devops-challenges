# Day 91 — Apache Web Deployment with Ansible `lineinfile`

![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-lineinfile-blue)

> **100 Days of DevOps — Day 91** | Nautilus / Stratos Datacenter

## Overview

Provision Apache HTTP Server on three application servers and publish a precise, two-line webpage. The first line **must be inserted using `ansible.builtin.lineinfile`** at the beginning of the file. Ensure correct ownership and file mode, with all operations controlled from the jump host.

## Requirements

| Setting | Required value |
|---|---|
| Controller | `thor@jump-host` |
| Inventory | `/home/thor/ansible/inventory` (pre-existing) |
| Playbook | `/home/thor/ansible/playbook.yml` |
| Hosts | `stapp01`, `stapp02`, `stapp03` |
| Package / service | `httpd` / running |
| Webpage | `/var/www/html/index.html` |
| First line | `Welcome to xFusionCorp Industries!` |
| Second line | `This is a Nautilus sample file, created using Ansible!` |
| File owner / group | `apache:apache` |
| Permissions | `0744` |
| Validator | `ansible-playbook -i inventory playbook.yml` |

## Architecture

```text
thor@jump-host
  └── /home/thor/ansible/
        ├── inventory (existing; SSH and sudo setup)
        └── playbook.yml
              ├── stapp01: httpd + /var/www/html/index.html
              ├── stapp02: httpd + /var/www/html/index.html
              └── stapp03: httpd + /var/www/html/index.html
```

## Solution playbook

Save at `/home/thor/ansible/playbook.yml`:

```yaml
---
- name: Configure Apache and Nautilus website
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install Apache HTTP server
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Ensure Apache is started and enabled
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Create webpage with exact initial content
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: |
          This is a Nautilus sample file, created using Ansible!
        owner: apache
        group: apache
        mode: '0744'

    - name: Add welcome line at the beginning using lineinfile
      ansible.builtin.lineinfile:
        path: /var/www/html/index.html
        line: 'Welcome to xFusionCorp Industries!'
        insertbefore: BOF
        state: present

    - name: Ensure required ownership and permissions
      ansible.builtin.file:
        path: /var/www/html/index.html
        state: file
        owner: apache
        group: apache
        mode: '0744'
```

### Design choices

1. **`yum`** guarantees the `httpd` package exists.
2. **`service`** ensures Apache is running now and enabled on boot.
3. **`copy`** establishes *exactly* the supplied initial text, clearing unrelated stale content.
4. **`lineinfile` with `insertbefore: BOF`** inserts the requested welcome line at the beginning.
5. **`file`** explicitly applies `apache:apache` and mode `0744`.

**Trade-off:** Because `copy` resets the file to one line before `lineinfile` adds the other, those two tasks may report `changed` on every run. The resulting file is correct, but this particular sequence is **not fully idempotent in its task-level execution**. See [Notes.md](Notes.md) for an improved production pattern.

## Run and verify

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
ansible -i inventory all -m command -a 'cat /var/www/html/index.html'
ansible -i inventory all -m command -a 'stat -c "%n %U:%G %a" /var/www/html/index.html'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
```

Expected file content on every server:

```text
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

Expected file metadata includes `apache:apache 744`; service output must be `active`. Successful Ansible recap reports **`failed=0` and `unreachable=0`** on all servers.

## Incident and resolution

An earlier attempt was marked **failed** by the KodeKloud grader with `incorrect content found in '/var/www/html/index.html' file on stapp01`, even though all Ansible tasks returned success. The revised task also specified different exact text and permissions from the previous attempt. The passing approach ensured the initial content was reset before inserting the requested line using `lineinfile`.

**Key lesson:** Ansible playbook success does not prove an application-level assertion; verify the *entire file*, not just the presence of a substring.

**Result:** Day 91 passed, as confirmed by the challenge completion report.
