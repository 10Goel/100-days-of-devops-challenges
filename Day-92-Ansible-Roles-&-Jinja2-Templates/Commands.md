# Day 92 — Commands and Verification Runbook

> All commands run on the **jump host as `thor`**, except remote actions executed through Ansible. The pre-existing inventory is not reproduced because it may contain credentials.

## 1. Inspect the provided Ansible project

```bash
cd /home/thor/ansible
pwd
ls -la
find role/httpd -maxdepth 3 -type f -print
cat role/httpd/tasks/main.yml
cat playbook.yml
ansible-inventory -i inventory --graph
```

Check that `stapp01` exists in inventory. Inspect before editing; preserve existing working role tasks.

## 2. Verify SSH and sudo

```bash
ansible -i inventory stapp01 -m ping
ansible -i inventory stapp01 -b -m command -a 'id -u'
```

Expected: `pong`, then remote UID `0`. If either fails, review authorized credentials, SSH host-key trust, and sudo configuration in the provided inventory.

## 3. Create the required template

```bash
mkdir -p /home/thor/ansible/role/httpd/templates
cat > /home/thor/ansible/role/httpd/templates/index.html.j2 <<'EOF_TEMPLATE'
This file was created using Ansible on {{ inventory_hostname }}
EOF_TEMPLATE
cat /home/thor/ansible/role/httpd/templates/index.html.j2
```

The template must contain `{{ inventory_hostname }}`; don't hardcode `stapp01`.

## 4. Back up existing role and playbook

```bash
cp /home/thor/ansible/role/httpd/tasks/main.yml /home/thor/ansible/role/httpd/tasks/main.yml.bak
cp /home/thor/ansible/playbook.yml /home/thor/ansible/playbook.yml.bak
```

## 5. Ensure the role tasks implement installation, service and template deployment

**Prefer editing the existing tasks file rather than overwriting it.** The complete effective task sequence is:

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

To edit:

```bash
vi /home/thor/ansible/role/httpd/tasks/main.yml
```

The `tasks/main.yml` file must be a list of tasks, not a full play containing `hosts:` or `roles:`.

## 6. Update the entry playbook

```bash
cat > /home/thor/ansible/playbook.yml <<'EOF_PLAYBOOK'
---
- name: Configure Apache on App Server 1 using the httpd role
  hosts: stapp01
  become: true
  gather_facts: true

  roles:
    - role: /home/thor/ansible/role/httpd
EOF_PLAYBOOK
```

An absolute path works with the lab's nonstandard `role/httpd/` directory and avoids reliance on additional `roles_path` settings.

## 7. Syntax and execution

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The **second command** is the exact KodeKloud validation invocation. Success means `failed=0` and `unreachable=0` for `stapp01`.

## 8. Verify content and permissions

```bash
ansible -i inventory stapp01 -m command -a 'cat /var/www/html/index.html'
ansible -i inventory stapp01 -m command -a "stat -c '%n %U:%G %a' /var/www/html/index.html"
ansible -i inventory stapp01 -m command -a 'rpm -q httpd'
ansible -i inventory stapp01 -b -m command -a 'systemctl is-active httpd'
ansible -i inventory stapp01 -b -m command -a 'systemctl is-enabled httpd'
```

Expected content:

```text
This file was created using Ansible on stapp01
```

Expected metadata:

```text
/var/www/html/index.html tony:tony 655
```

Service state should be `active`, with `enabled` if the service role includes `enabled: true`.

Optional HTTP response check, where the lab Apache listens on its default port:

```bash
ansible -i inventory stapp01 -m command -a 'curl -fsS http://localhost/'
```

## 9. Check repeatability

```bash
ansible-playbook -i inventory playbook.yml
```

If all resources already match the desired state, tasks should normally report `ok` and no changes.

## 10. Troubleshooting

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible -i inventory stapp01 -b -m command -a 'systemctl status httpd --no-pager'
ansible -i inventory stapp01 -b -m command -a 'journalctl -u httpd -n 30 --no-pager'
```

- **Role not found:** ensure the full role directory exists and is referenced correctly.
- **Template not found:** verify `role/httpd/templates/index.html.j2` exists.
- **Wrong hostname:** verify `inventory_hostname` is referenced in Jinja2.
- **Wrong file metadata:** apply `owner: tony`, `group: tony`, `mode: '0655'` to the template task.
- **Missing sudo password:** configure authorized noninteractive escalation in inventory; the validator supplies no additional flags.
- **HTTPD not installed after clean run:** verify installation and service tasks execute before template deployment.

Never include real passwords or private keys in a GitHub commit.
