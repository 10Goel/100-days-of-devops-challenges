# Day 84 — Commands Runbook

Run from the **jump host** (`thor@jump-host`). Replace password placeholders with current authorized lab credentials. Do not publish actual credentials.

## 1. Prerequisite inspection

```bash
whoami
hostname
ls -l /usr/src/sysops/index.html
file /usr/src/sysops/index.html
mkdir -p /home/thor/ansible
cd /home/thor/ansible
pwd
ls -la
```

Do not recreate a missing source file: the challenge requires copying the provided file.

## 2. Inventory

```bash
vi /home/thor/ansible/inventory
```

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password='APP1_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD'
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='APP2_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD'
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password='APP3_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD'
```

```bash
chmod 600 /home/thor/ansible/inventory
ls -l /home/thor/ansible/inventory
ansible-inventory -i inventory --graph
```

Avoid publishing unredacted `ansible-inventory --list` output; it may expose inventory variables.

## 3. First-time SSH trust

From the jump host, verify fingerprints against trusted infrastructure information before accepting them. Authenticate and exit each session:

```bash
ssh tony@stapp01
# exit
ssh steve@stapp02
# exit
ssh banner@stapp03
# exit
```

```bash
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
```

## 4. Test connectivity

```bash
cd /home/thor/ansible
ansible -i inventory app_servers -m ping
```

Expected: `pong` from all three servers.

## 5. Playbook creation

```bash
vi /home/thor/ansible/playbook.yml
```

```yaml
---
- name: Copy sysops HTML to all application servers
  hosts: app_servers
  become: true
  gather_facts: false

  tasks:
    - name: Ensure sysops directory exists
      ansible.builtin.file:
        path: /opt/sysops
        state: directory
        mode: '0755'

    - name: Copy HTML from the jump host
      ansible.builtin.copy:
        src: /usr/src/sysops/index.html
        dest: /opt/sysops/index.html
        mode: '0644'
```

## 6. Validate and execute

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Illustrative successful recap (counts may vary):

```text
stapp01 : ok=2 changed=2 unreachable=0 failed=0
stapp02 : ok=2 changed=2 unreachable=0 failed=0
stapp03 : ok=2 changed=2 unreachable=0 failed=0
```

## 7. Verify copied files

```bash
ansible -i inventory app_servers -b -m stat -a 'path=/opt/sysops/index.html'
sha256sum /usr/src/sysops/index.html
ansible -i inventory app_servers -b -m command -a 'sha256sum /opt/sysops/index.html'
```

All four SHA-256 hashes should match.

## 8. Re-run to verify idempotency

```bash
ansible-playbook -i inventory playbook.yml
```

When remote content and permissions are already correct, the file and copy tasks should generally report `ok` without changes.
