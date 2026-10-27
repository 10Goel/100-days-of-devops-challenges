# Day 91 — Commands and Verification Runbook

All commands run on **`thor@jump-host`**, unless otherwise noted. The inventory is pre-provided; never commit credentials, tokens or private keys.

## 1. Locate the lab inventory

```bash
cd /home/thor/ansible
pwd
ls -la
ansible-inventory -i inventory --graph
```

Ensure `stapp01`, `stapp02`, and `stapp03` appear. Do not overwrite the supplied inventory.

## 2. Test connectivity and escalation

```bash
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
```

Expected: `pong` for each host and UID `0` under sudo.

## 3. Create the playbook

```bash
vi /home/thor/ansible/playbook.yml
```

Paste this **complete, lab-tested solution specification**:

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

Save in vi: `Esc`, `:wq`, `Enter`.

## 4. Validate syntax and execute

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The recap must show `failed=0` and `unreachable=0` on all three servers.

## 5. Inspect webpage and exact line order

```bash
ansible -i inventory all -m command -a 'cat /var/www/html/index.html'
ansible -i inventory all -m command -a 'head -n 1 /var/www/html/index.html'
ansible -i inventory all -m command -a 'wc -l /var/www/html/index.html'
ansible -i inventory all -m command -a 'sed -n l /var/www/html/index.html'
```

Exactly two lines must be present:

```text
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

`sed -n l` appends `$` to show each line ending, helping reveal stray whitespace or unexpected lines.

## 6. Validate ownership, permissions and Apache service

```bash
ansible -i inventory all -m command -a 'stat -c "%n %U:%G %a" /var/www/html/index.html'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
ansible -i inventory all -m command -a 'rpm -q httpd'
```

Expected metadata: `/var/www/html/index.html apache:apache 744`; `httpd` is `active`, and (with this playbook) `enabled`.

## 7. Optional HTTP smoke test

```bash
ansible -i inventory all -m command -a 'curl -fsS http://localhost/'
```

This assumes the default Apache listener at port 80 and unmodified document root.

## 8. Troubleshooting

```bash
# If parsing or authentication is failing
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping -vv

# If service status is unexpected
ansible -i inventory all -b -m command -a 'systemctl status httpd --no-pager'

# If content is rejected despite successful Ansible recap
ansible -i inventory all -m command -a 'sed -n l /var/www/html/index.html'
```

## 9. Important rerun behavior

Re-running the playbook produces the correct final content, but `copy` may remove the welcome line before `lineinfile` adds it again, causing these tasks to report changes. Do not mislabel this pattern fully idempotent.
