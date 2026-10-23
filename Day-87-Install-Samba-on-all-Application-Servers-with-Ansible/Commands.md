# Day 87 — Commands and Execution Runbook

> Run these commands as `thor` on the jump host unless specified. Replace password placeholders from the current lab's authorized server details; do not commit live credentials.

## A. Prepare working directory

```bash
whoami
hostname
mkdir -p /home/thor/playbook
cd /home/thor/playbook
pwd
ls -la
```

## B. Configure inventory

```bash
vi /home/thor/playbook/inventory
```

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password='APP1_SSH_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD'
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='APP2_SSH_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD'
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password='APP3_SSH_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD'
```

```bash
chmod 600 /home/thor/playbook/inventory
ls -l /home/thor/playbook/inventory
ansible-inventory -i inventory --graph
```

Never publish the populated inventory or unredacted output of `ansible-inventory --list`.

## C. Establish trusted SSH host keys

From the jump host, verify the host-key fingerprint from trusted lab records when prompted, then authenticate and exit each server:

```bash
ssh tony@stapp01
# exit
ssh steve@stapp02
# exit
ssh banner@stapp03
# exit
```

Check local known-host entries:

```bash
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
```

## D. Test connectivity and sudo

```bash
ansible -i inventory app_servers -m ping
ansible -i inventory app_servers -b -m command -a 'id -u'
```

All three ping responses should contain `pong`, and all three sudo tests should return `0`.

## E. Create playbook

```bash
vi /home/thor/playbook/playbook.yml
```

```yaml
---
- name: Install Samba across Nautilus application servers
  hosts: app_servers
  become: true
  gather_facts: true

  tasks:
    - name: Ensure Samba package is installed
      ansible.builtin.yum:
        name: samba
        state: present
```

## F. Validate and execute

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The second command is the **exact invocation** used by the lab validator. Do not rely on `-k`, `-K`, or `-u` at run time.

## G. Validate result on each host

```bash
ansible -i inventory app_servers -m command -a 'rpm -q samba'
ansible-playbook -i inventory playbook.yml
```

Example recap after a successful first installation (actual counts vary):

```text
stapp01 : ok=2 changed=1 unreachable=0 failed=0
stapp02 : ok=2 changed=1 unreachable=0 failed=0
stapp03 : ok=2 changed=1 unreachable=0 failed=0
```

On a later execution, `changed=0` is expected if the desired package state is already satisfied.

## H. Diagnostic commands (only when needed)

```bash
ansible-inventory -i inventory --graph
ansible -i inventory app_servers -m ping -vv
ansible -i inventory app_servers -b -m command -a 'id -u'
ansible -i inventory app_servers -b -m command -a 'rpm -q samba'
```

**Troubleshooting:** `UNREACHABLE` usually concerns transport or authentication; `Missing sudo password` concerns escalation; `No package samba available` concerns package repositories. Avoid turning off SSH host-key checking as a shortcut.
