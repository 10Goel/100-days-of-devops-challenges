# Day 88 — Commands and Execution Runbook

> Run Ansible commands on the jump host as `thor`. The inventory is provided by the lab. Do not overwrite it blindly or commit credentials.

## 1. Inspect controller and inventory

```bash
whoami
cd /home/thor/ansible
pwd
ls -la
ansible-inventory -i inventory --graph
```

Ensure that `stapp01`, `stapp02` and `stapp03` appear. Inspect any sensitive inventory with an editor rather than pasting its contents into a public issue.

## 2. Test connections

```bash
ansible -i inventory all -m ping
```

Look for `pong` for each target. If a host-key verification failure occurs, connect once to the corresponding managed server from the controller and verify its fingerprint against trusted lab information:

```bash
ssh tony@stapp01    # exit after authentication
ssh steve@stapp02   # exit after authentication
ssh banner@stapp03  # exit after authentication
```

Run these **one at a time**; don't launch them as one sequence while still logged into another server. Use the lab's authorized credentials and do not turn off host-key verification.

## 3. Create playbook

```bash
vi /home/thor/ansible/playbook.yml
```

```yaml
---
- name: Install Apache and deploy managed sample page
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install httpd
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start and enable httpd
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Ensure document root
      ansible.builtin.file:
        path: /var/www/html
        state: directory
        mode: '0755'

    - name: Add the required block to index.html
      ansible.builtin.blockinfile:
        path: /var/www/html/index.html
        create: true
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!

    - name: Set owner group and permissions
      ansible.builtin.file:
        path: /var/www/html/index.html
        state: file
        owner: apache
        group: apache
        mode: '0644'
```

Save the file in `vi` with `Esc`, `:wq`, Enter.

## 4. Validate YAML and run the exact validator command

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Ensure the play recap lists all three servers with `unreachable=0` and `failed=0`. Actual `changed` counters vary by previous state.

## 5. Verify package, service and webpage

```bash
ansible -i inventory all -m command -a 'rpm -q httpd'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
ansible -i inventory all -m command -a 'cat /var/www/html/index.html'
ansible -i inventory all -m command -a "stat -c '%n %U:%G %a' /var/www/html/index.html"
```

Expected on each host:

```text
/var/www/html/index.html apache:apache 644
```

## 6. Optional network check

Check the local web server from each managed host using the service's configured HTTP address/port, if needed. A service being active does not by itself prove that network ingress and firewall configuration are correct.

## 7. Check idempotency

```bash
ansible-playbook -i inventory playbook.yml
```

If no external changes intervene, tasks should normally return `ok` and `changed=0` on repeat execution.

## 8. Common troubleshooting diagnostics

```bash
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
ansible -i inventory all -b -m command -a 'systemctl status httpd --no-pager'
ansible -i inventory all -m command -a "stat -c '%U:%G %a' /var/www/html/index.html"
```

The ad-hoc `command` module may report `CHANGED` for read-only checks; this does not mean those commands modified files.
