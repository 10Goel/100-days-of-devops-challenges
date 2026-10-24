# Day 88 — Apache Web Server Deployment with Ansible

![Status](https://img.shields.io/badge/KodeKloud-Day_88_PASSED-brightgreen) ![Automation](https://img.shields.io/badge/Automation-Ansible-blue)

## Overview
Automate Apache HTTP Server deployment across all three Nautilus application servers. Install and enable `httpd`, insert an exact multiline welcome message using **`ansible.builtin.blockinfile`**, and ensure the resulting HTML file has owner/group `apache:apache` and mode `0644`.

**Challenge result: PASSED** (confirmed by the learner). The commands below are a reproducible runbook; sample outputs are illustrative rather than captured logs.

## Requirements

| Component | Value |
|---|---|
| Controller | Jump host (`thor`) |
| Existing inventory | `/home/thor/ansible/inventory` |
| Playbook | `/home/thor/ansible/playbook.yml` |
| Targets | `stapp01`, `stapp02`, `stapp03` (all app servers) |
| Package and service | `httpd`, installed and running |
| Content mechanism | `ansible.builtin.blockinfile` |
| Target file | `/var/www/html/index.html` |
| Owner / group | `apache:apache` |
| Permission mode | `0644` |
| Validator invocation | `ansible-playbook -i inventory playbook.yml` |

### Required content (exact spelling and punctuation)

```text
Welcome to XfusionCorp!
This is Nautilus sample file, created using Ansible!
Please do not modify this file manually!
```

## Architecture

```text
Jump host: /home/thor/ansible/
  ├─ inventory
  └─ playbook.yml
         |
         +-- SSH / sudo --> stapp01 --> httpd + /var/www/html/index.html
         +-- SSH / sudo --> stapp02 --> httpd + /var/www/html/index.html
         +-- SSH / sudo --> stapp03 --> httpd + /var/www/html/index.html
```

## Solution — `playbook.yml`

```yaml
---
- name: Configure Apache and publish managed HTML content
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Ensure Apache is installed
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Ensure Apache is running and enabled
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Ensure document root exists
      ansible.builtin.file:
        path: /var/www/html
        state: directory
        mode: '0755'

    - name: Insert required managed HTML block
      ansible.builtin.blockinfile:
        path: /var/www/html/index.html
        create: true
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!

    - name: Apply required owner group and file permissions
      ansible.builtin.file:
        path: /var/www/html/index.html
        state: file
        owner: apache
        group: apache
        mode: '0644'
```

**Why this ordering?** Installing `httpd` creates the `apache` account on the target distributions; starting the service satisfies runtime requirements; the content task adds the requested block, and the final task ensures the specified file metadata after content changes.

`blockinfile` usually surrounds the content with `# BEGIN ANSIBLE MANAGED BLOCK` and `# END ANSIBLE MANAGED BLOCK` markers. These are expected and help the module find the block on subsequent runs. The challenge specifies the three lines must be present, not that the file contains only those lines.

## Execution and verification

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Verify on **all** targets:

```bash
ansible -i inventory all -m command -a 'rpm -q httpd'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
ansible -i inventory all -m command -a 'cat /var/www/html/index.html'
ansible -i inventory all -m command -a "stat -c '%n %U:%G %a' /var/www/html/index.html"
```

Expected metadata on every host: `/var/www/html/index.html apache:apache 644`. Play recap must show `failed=0` and `unreachable=0` on all three hosts. A second run should normally be idempotent if nothing external changes.

## Learning outcomes

- Combine package, service, content, and metadata automation in one play.
- Use `blockinfile` instead of modifying an entire file.
- Understand `become: true`, Ansible's desired-state model, and the importance of exact paths and content.
- Validate package state, service health, file contents, ownership, permissions, and repeatability.

## Security and scope

Do not commit real host passwords, SSH private keys, or unredacted inventory secrets. This project documents a training lab. No firewall, Apache port, or virtual-host changes were required.
