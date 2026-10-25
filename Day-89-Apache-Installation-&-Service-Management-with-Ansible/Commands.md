# Day 89 — Commands.md

A reproducible runbook for the **passed** Day 89 task. Run commands on the **jump host as `thor`**, unless noted.

## 1. Inspect the lab environment

```bash
whoami
hostname
cd /home/thor/ansible
pwd
ls -la
ansible-inventory -i inventory --graph
```

The provided inventory should include `stapp01`, `stapp02`, and `stapp03`. It may use either direct entries or a group. **Do not publish credential-bearing inventory contents.**

## 2. Check authentication and reachability

```bash
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
```

- Ping should return `pong` on each node.
- `id -u` with `-b` should print `0` for each node.

### If a host key is not trusted

Verify the SSH fingerprint from trusted lab information before accepting. Connect one at a time:

```bash
ssh tony@stapp01
# exit after confirming host identity/login
ssh steve@stapp02
# exit after confirming host identity/login
ssh banner@stapp03
# exit after confirming host identity/login
```

```bash
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
```

If credentials or sudo settings are absent, inspect the existing inventory (`vi inventory`) and configure them using the authorized current-lab values. Avoid `-k` and `-K` in the final validation because no extra flags are allowed.

## 3. Create the playbook

```bash
vi /home/thor/ansible/playbook.yml
```

Paste:

```yaml
---
- name: Install and configure Apache on all application servers
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install Apache HTTP Server
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start and enable Apache HTTP Server
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

In `vi`: press `i` to insert; then `Esc`, `:wq`, Enter to save.

## 4. Validate and run

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Illustrative successful recap (actual `ok`/`changed` numbers vary):

```text
stapp01 : ok=3 changed=2 unreachable=0 failed=0
stapp02 : ok=3 changed=2 unreachable=0 failed=0
stapp03 : ok=3 changed=2 unreachable=0 failed=0
```

## 5. Verify package and service states

```bash
ansible -i inventory all -m command -a 'rpm -q httpd'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
```

Expected respectively: installed RPM version, `active`, `enabled` on **all three** hosts.

## 6. Verify idempotency

```bash
ansible-playbook -i inventory playbook.yml
```

Expected: `changed=0` on an unchanged fleet, barring environmental changes.

## 7. Debug failures

```bash
# Review service status on all hosts
ansible -i inventory all -b -m command -a 'systemctl status httpd --no-pager'

# Inspect recent unit logs
ansible -i inventory all -b -m command -a 'journalctl -u httpd -n 30 --no-pager'

# Review enabled package repositories on RPM-based systems
ansible -i inventory all -b -m command -a 'dnf repolist'

# Inventory graph for host selection
ansible-inventory -i inventory --graph
```

Do not alter unrelated services, Apache ports, or web documents for this exercise.
