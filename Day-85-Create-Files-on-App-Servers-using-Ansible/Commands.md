# Day 85 — Commands Runbook

Commands run on `thor@jump-host`. Substitute authorized current-lab credentials only in your temporary private inventory.

## 1. Prepare
```bash
mkdir -p /home/thor/playbook
cd /home/thor/playbook
pwd
ls -la
```

## 2. Configure inventory
```bash
vi /home/thor/playbook/inventory
```
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password='APP1_SSH_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD' file_owner=tony
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='APP2_SSH_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD' file_owner=steve
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password='APP3_SSH_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD' file_owner=banner
```
```bash
chmod 600 /home/thor/playbook/inventory
ansible-inventory -i inventory --graph
```

## 3. Prepare SSH host-key trust
Manually authenticate with each server from jump host, compare its fingerprint with trusted lab information, then exit back to jump host.
```bash
ssh tony@stapp01
# exit after successful login
ssh steve@stapp02
# exit after successful login
ssh banner@stapp03
# exit after successful login
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
```
Do not disable SSH host-key verification globally.

## 4. Ping all managed hosts
```bash
ansible -i inventory app_servers -m ping
```
Expected: each node responds with `pong`.

## 5. Create playbook
```bash
vi /home/thor/playbook/playbook.yml
```
```yaml
---
- name: Create empty webdata file with server-specific ownership
  hosts: app_servers
  become: true
  gather_facts: false

  tasks:
    - name: Create zero-byte file with required mode and ownership
      ansible.builtin.copy:
        content: ''
        dest: /home/webdata.txt
        owner: "{{ file_owner }}"
        group: "{{ file_owner }}"
        mode: '0777'
```

## 6. Validate and run
```bash
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```
The second command is the exact KodeKloud validator invocation. Look for `unreachable=0 failed=0` on all 3 hosts.

## 7. Verify metadata and file size
```bash
ansible -i inventory app_servers -b -m command -a "stat -c '%n %U:%G %a %s bytes' /home/webdata.txt"
ansible -i inventory app_servers -b -m stat -a 'path=/home/webdata.txt'
```
Per host, verify: zero-byte regular file; owner:group of `tony:tony`, `steve:steve`, `banner:banner`; mode `0777` (displayed as `777` by `stat -c %a`).

## 8. Repeat to check idempotency
```bash
ansible-playbook -i inventory playbook.yml
```
Unchanged files normally result in `ok`, with no further changes.

## Diagnostic commands
```bash
ansible-inventory -i inventory --graph
ansible -i inventory stapp01 -m ping
ansible -i inventory stapp02 -m ping
ansible -i inventory stapp03 -m ping
ansible-playbook -i inventory playbook.yml -v
```
Use verbose output only for manual debugging; inspect it for sensitive data before publishing.
