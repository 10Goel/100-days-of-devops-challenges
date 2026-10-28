# Day 92 — Ansible Roles and Jinja2 Templates

![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen) ![Ansible](https://img.shields.io/badge/Automation-Ansible-blue) ![Day](https://img.shields.io/badge/100_Days_of_DevOps-Day_92-orange)

## Overview

Implemented an existing `httpd` Ansible role to install and start Apache on App Server 1, render a dynamic `index.html` using Jinja2, and enforce the requested owner, group, and permissions. The challenge was **passed**.

## Exact challenge requirements

| Item | Value |
|---|---|
| Control node | Jump host (`thor`) |
| Inventory | `/home/thor/ansible/inventory` (provided) |
| Entry playbook | `/home/thor/ansible/playbook.yml` |
| Role directory | `/home/thor/ansible/role/httpd/` (singular `role`) |
| Managed node | App Server 1 (`stapp01`) |
| Template file | `role/httpd/templates/index.html.j2` |
| Role task file | `role/httpd/tasks/main.yml` |
| Deployed file | `/var/www/html/index.html` |
| Exact rendered line | `This file was created using Ansible on stapp01` |
| Owner / group | `tony:tony` |
| Mode | `0655` |
| Validator | `ansible-playbook -i inventory playbook.yml` |

The hostname must be generated through **`{{ inventory_hostname }}`**, never hardcoded in the template. The validator removes `httpd` and the deployed webpage before running the playbook, so the role must recreate both.

## Architecture

```text
Jump host: /home/thor/ansible/
  inventory
  playbook.yml  -- invokes -->  role/httpd/
                                  tasks/main.yml
                                  templates/index.html.j2
                                          |
                                    SSH + sudo
                                          v
                                   stapp01 (tony)
                                   httpd running
                                   /var/www/html/index.html
                                   tony:tony / 0655
```

## Project structure

```text
Day-92-Ansible-Roles-Jinja2-Templates/
├── README.md
├── Commands.md
└── Notes.md
```

On the lab jump host, the **operational** structure is:

```text
/home/thor/ansible/
├── inventory                 # supplied by lab
├── playbook.yml
└── role/
    └── httpd/
        ├── tasks/
        │   └── main.yml
        └── templates/
            └── index.html.j2
```

## Implementation

### 1. Template: `role/httpd/templates/index.html.j2`

```jinja2
This file was created using Ansible on {{ inventory_hostname }}
```

The template engine renders the host's **inventory alias**: `stapp01`.

### 2. Role tasks: `role/httpd/tasks/main.yml`

```yaml
---
- name: Install Apache HTTP Server
  ansible.builtin.yum:
    name: httpd
    state: present

- name: Start and enable Apache service
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

- name: Render host-specific homepage
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: tony
    group: tony
    mode: '0655'
```

**Note:** The lab provides an almost-complete role; in practice inspect and preserve working existing tasks, adding only the missing template operation. The above is a self-contained reference of the required effective tasks.

### 3. Entry playbook: `playbook.yml`

```yaml
---
- name: Configure Apache on App Server 1 using the httpd role
  hosts: stapp01
  become: true
  gather_facts: true

  roles:
    - role: /home/thor/ansible/role/httpd
```

The absolute role path is deliberate: the lab uses `role/`, not the conventional `roles/` search directory. `become: true` supplies elevated permissions for package/service/file operations, while the final file is owned by `tony`.

## Execution and validation

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ansible -i inventory stapp01 -m ping
ansible -i inventory stapp01 -b -m command -a 'id -u'
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Confirm:

```bash
ansible -i inventory stapp01 -m command -a 'cat /var/www/html/index.html'
ansible -i inventory stapp01 -m command -a "stat -c '%U:%G %a' /var/www/html/index.html"
ansible -i inventory stapp01 -m command -a 'rpm -q httpd'
ansible -i inventory stapp01 -b -m command -a 'systemctl is-active httpd'
ansible -i inventory stapp01 -b -m command -a 'systemctl is-enabled httpd'
```

Expected rendered content:

```text
This file was created using Ansible on stapp01
```

Expected file metadata:

```text
tony:tony 655
```

A successful play reports `failed=0` and `unreachable=0` for `stapp01`. Exact `ok`/`changed` counts depend on existing state.

## Clean-state acceptance

The KodeKloud validator removes Apache and `/var/www/html/index.html`, then invokes the playbook. A correct role must therefore install `httpd`, start the service, and recreate the rendered template **without extra arguments**. Do not manually remove packages in order to test this; allow the validator to perform its clean-state test.

## Lessons learned

- Roles make Ansible tasks modular and reusable.
- Jinja2 generates content from built-in variables, avoiding hardcoded hostnames.
- `template` sources inside a role are automatically resolved against its `templates/` directory.
- Explicit owner, group, and quoted octal mode enforce file metadata.
- Repeatable infrastructure automation must work after resources are deleted.

**Security:** No passwords, private SSH keys, or unredacted lab inventory are stored in this repository.
