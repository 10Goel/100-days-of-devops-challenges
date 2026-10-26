# Day 90 — Commands and Verification Runbook

> Run on the **jump host as `thor`** unless explicitly stated otherwise. Do not publish credential-bearing inventory output.

## 1. Confirm environment and inventory

```bash
whoami
cd /home/thor/ansible
pwd
ls -la
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
```

Expected targets: `stapp01`, `stapp02`, `stapp03`. Root UID should be `0` on escalated commands. Correct SSH/sudo failures in the pre-existing inventory without wiping unrelated entries.

## 2. Confirm required Ansible module

```bash
ansible-doc ansible.posix.acl
ansible-galaxy collection list
# Only if the collection is missing and installation is permitted:
ansible-galaxy collection install ansible.posix
```

Managed hosts need the `acl` package for `setfacl` and `getfacl` (the playbook installs it).

## 3. Create the playbook

```bash
vi /home/thor/ansible/playbook.yml
```

Use the complete YAML in [README.md](README.md), with three plays targeting each host and applying named ACL entries.

## 4. Syntax check and execute

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The second command is the **exact KodeKloud validator invocation**. No `-k`, `-K`, or `--become` flags should be needed because the playbook and existing inventory provide the necessary configuration.

## 5. Verify root ownership, file type and zero bytes

```bash
ansible -i inventory stapp01 -b -m stat -a 'path=/opt/security/blog.txt'
ansible -i inventory stapp02 -b -m stat -a 'path=/opt/security/story.txt'
ansible -i inventory stapp03 -b -m stat -a 'path=/opt/security/media.txt'
```

For each host: `exists: true`, `isreg: true`, `size: 0`, owner `root`.

## 6. Verify named ACLs (most important)

```bash
ansible -i inventory stapp01 -b -m command -a 'getfacl -p /opt/security/blog.txt'
ansible -i inventory stapp02 -b -m command -a 'getfacl -p /opt/security/story.txt'
ansible -i inventory stapp03 -b -m command -a 'getfacl -p /opt/security/media.txt'
```

Relevant entries expected:

```text
# stapp01
# owner: root
# group: root
group:tony:r--

# stapp02
# owner: root
# group: root
user:steve:rw-

# stapp03
# owner: root
# group: root
group:banner:rw-
```

Confirm masks do not limit named entries: `r--` for `tony` read, `rw-` for `steve` and `banner` read/write. The rest of the output depends on the base permissions.

## 7. Confirm repeatability

```bash
ansible-playbook -i inventory playbook.yml
```

Resources already in the desired state should generally return `ok` without changes.

## 8. Diagnostic commands

```bash
ansible -i inventory all -m ping -vvv
ansible -i inventory all -b -m command -a 'command -v getfacl'
# If the above command module reports shell built-in issues, use:
ansible -i inventory all -b -m shell -a 'command -v getfacl && command -v setfacl'
```

`ansible.builtin.command` does not use a shell, so shell built-ins such as `command -v` are better run through `ansible.builtin.shell` (or check `/usr/bin/getfacl` directly). Avoid unnecessarily disabling SSH host-key validation.
