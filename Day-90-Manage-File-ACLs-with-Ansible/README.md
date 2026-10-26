# Day 90 — Manage File Access Control Lists (ACLs) with Ansible

![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-ACL-blue)

## Challenge objective

Create **three empty, root-owned files** on different Stratos DC application servers and grant additional, fine-grained permissions through **POSIX Access Control Lists**, using Ansible only.

| Managed host | Required file | Owner | ACL entity | ACL rights |
|---|---|---|---|---|
| `stapp01` | `/opt/security/blog.txt` | `root` | group `tony` | `r` |
| `stapp02` | `/opt/security/story.txt` | `root` | user `steve` | `rw` |
| `stapp03` | `/opt/security/media.txt` | `root` | group `banner` | `rw` |

- **Control node:** jump host, user `thor`
- **Existing inventory:** `/home/thor/ansible/inventory`
- **New playbook:** `/home/thor/ansible/playbook.yml`
- **Required validator command:** `ansible-playbook -i inventory playbook.yml`
- **Outcome:** KodeKloud challenge passed (reported by the learner). Illustrative output below is not a captured terminal transcript.

## Architecture

```text
thor@jump-host  /home/thor/ansible/
  inventory + playbook.yml
            |
            | SSH + sudo + ansible.posix.acl
            +-- stapp01 -> /opt/security/blog.txt  -> group:tony:r--
            +-- stapp02 -> /opt/security/story.txt -> user:steve:rw-
            +-- stapp03 -> /opt/security/media.txt -> group:banner:rw-

All files: owner=root, group=root, size=0 bytes
```

## Solution: `playbook.yml`

```yaml
---
- name: Create root-owned security file on App Server 1
  hosts: stapp01
  become: true
  gather_facts: false
  tasks:
    - name: Ensure ACL utilities are present
      ansible.builtin.yum:
        name: acl
        state: present
    - name: Create security directory
      ansible.builtin.file:
        path: /opt/security
        state: directory
        owner: root
        group: root
        mode: '0755'
    - name: Create empty blog file owned by root
      ansible.builtin.copy:
        content: ''
        dest: /opt/security/blog.txt
        owner: root
        group: root
        mode: '0640'
    - name: Give tony group read access
      ansible.posix.acl:
        path: /opt/security/blog.txt
        etype: group
        entity: tony
        permissions: r
        state: present

- name: Create root-owned security file on App Server 2
  hosts: stapp02
  become: true
  gather_facts: false
  tasks:
    - name: Ensure ACL utilities are present
      ansible.builtin.yum:
        name: acl
        state: present
    - name: Create security directory
      ansible.builtin.file:
        path: /opt/security
        state: directory
        owner: root
        group: root
        mode: '0755'
    - name: Create empty story file owned by root
      ansible.builtin.copy:
        content: ''
        dest: /opt/security/story.txt
        owner: root
        group: root
        mode: '0660'
    - name: Give steve user read-write access
      ansible.posix.acl:
        path: /opt/security/story.txt
        etype: user
        entity: steve
        permissions: rw
        state: present

- name: Create root-owned security file on App Server 3
  hosts: stapp03
  become: true
  gather_facts: false
  tasks:
    - name: Ensure ACL utilities are present
      ansible.builtin.yum:
        name: acl
        state: present
    - name: Create security directory
      ansible.builtin.file:
        path: /opt/security
        state: directory
        owner: root
        group: root
        mode: '0755'
    - name: Create empty media file owned by root
      ansible.builtin.copy:
        content: ''
        dest: /opt/security/media.txt
        owner: root
        group: root
        mode: '0660'
    - name: Give banner group read-write access
      ansible.posix.acl:
        path: /opt/security/media.txt
        etype: group
        entity: banner
        permissions: rw
        state: present
```

### Why this design?

1. **Separate plays** target the precise server/file/ACL pair, preventing permissions from being applied to the wrong host.
2. **`become: true`** allows file creation in root-controlled paths and ACL modification.
3. **`ansible.builtin.copy` with `content: ''`** ensures a zero-byte file rather than merely updating an existing file's timestamp.
4. **`ansible.posix.acl`** grants the named user/group access while preserving root file ownership.
5. **Quoted mode strings** avoid YAML octal interpretation mistakes. The selected base modes support the requested ACL mask; named ACL entries remain the authoritative test of additional access.

> The `ansible.posix` collection must be available on the controller, and the `acl` utilities (`setfacl`, `getfacl`) must be available on managed nodes. This playbook installs the latter using `yum` on the lab's RPM-based hosts.

## Execution

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible-doc ansible.posix.acl
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

If the module is missing and installation is permitted: `ansible-galaxy collection install ansible.posix`.

### Verify the ACLs

```bash
ansible -i inventory stapp01 -b -m command -a 'getfacl -p /opt/security/blog.txt'
ansible -i inventory stapp02 -b -m command -a 'getfacl -p /opt/security/story.txt'
ansible -i inventory stapp03 -b -m command -a 'getfacl -p /opt/security/media.txt'
```

Look for `group:tony:r--`, `user:steve:rw-`, and `group:banner:rw-`, respectively, and verify `# owner: root` and effective rights (ACL mask). Confirm all file sizes are zero.

### Success criteria

- All three hosts report `failed=0` and `unreachable=0`.
- Each file exists, is an empty regular file, and remains owned by `root`.
- Named ACL entries grant **exactly** the required additional access.
- The playbook succeeds with the validator's exact command and without interactive arguments.

## Troubleshooting snapshot

| Symptom | Check |
|---|---|
| `ansible.posix.acl` not found | Check/install controller collection |
| `setfacl` or `getfacl` missing | Install `acl` on managed hosts |
| SSH failure | Review inventory authentication, SSH host-key trust |
| Sudo failure | Verify noninteractive `become` credentials/rights |
| Wrong effective ACL permissions | Examine `mask::` and `#effective:` in `getfacl` |
| Wrong ownership or non-empty file | Inspect `copy` task, destination and metadata |

See [Commands.md](Commands.md) for the runbook and [Notes.md](Notes.md) for the underlying Linux ACL concepts, security considerations, and interview questions.

## Security note

Do not commit inventory passwords, private SSH keys, or real infrastructure secrets to this repository. Keep the original lab inventory out of version control if it contains credentials.
