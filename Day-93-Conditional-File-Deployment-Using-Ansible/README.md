# Day 93 — Conditional File Deployment Using Ansible `when`

![Status](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-Conditionals-blue)

## Challenge overview

The Nautilus DevOps team needed to distribute three **different existing files** from the jump host to specific application servers in Stratos DC. The implementation must use **one play targeting `all` hosts** and select the correct copy task with Ansible's `when` conditionals.

**Challenge status:** Passed (Day 93 of 100 Days of DevOps).

## Requirements

| Destination host | Source on controller | Remote destination | Remote owner/group | Mode |
|---|---|---|---|---|
| `stapp01` | `/usr/src/data/blog.txt` | `/opt/data/blog.txt` | `tony:tony` | `0755` |
| `stapp02` | `/usr/src/data/story.txt` | `/opt/data/story.txt` | `steve:steve` | `0755` |
| `stapp03` | `/usr/src/data/media.txt` | `/opt/data/media.txt` | `banner:banner` | `0755` |

- Existing inventory: `/home/thor/ansible/inventory`
- Playbook: `/home/thor/ansible/playbook.yml`
- Required execution: `ansible-playbook -i inventory playbook.yml` from `/home/thor/ansible`
- The play must use `hosts: all` and `when` statements.

## Architecture

```text
Jump host (thor) — Ansible control node
/usr/src/data/
  ├── blog.txt  ──[when: stapp01]──> stapp01:/opt/data/blog.txt  (tony:tony 0755)
  ├── story.txt ──[when: stapp02]──> stapp02:/opt/data/story.txt (steve:steve 0755)
  └── media.txt ──[when: stapp03]──> stapp03:/opt/data/media.txt (banner:banner 0755)
```

## Final Ansible playbook

Create `/home/thor/ansible/playbook.yml`:

```yaml
---
- name: Deploy host-specific files using when conditions
  hosts: all
  become: true
  gather_facts: false

  tasks:
    - name: Ensure the destination directory exists
      ansible.builtin.file:
        path: /opt/data
        state: directory
        mode: '0755'

    - name: Copy blog.txt to App Server 1
      ansible.builtin.copy:
        src: /usr/src/data/blog.txt
        dest: /opt/data/blog.txt
        owner: tony
        group: tony
        mode: '0755'
      when: inventory_hostname == 'stapp01'

    - name: Copy story.txt to App Server 2
      ansible.builtin.copy:
        src: /usr/src/data/story.txt
        dest: /opt/data/story.txt
        owner: steve
        group: steve
        mode: '0755'
      when: inventory_hostname == 'stapp02'

    - name: Copy media.txt to App Server 3
      ansible.builtin.copy:
        src: /usr/src/data/media.txt
        dest: /opt/data/media.txt
        owner: banner
        group: banner
        mode: '0755'
      when: inventory_hostname == 'stapp03'
```

### How the conditions work

Each task is evaluated independently for every inventory host. For example, `when: inventory_hostname == 'stapp01'` runs the `blog.txt` task on `stapp01`, but produces `skipping` on `stapp02` and `stapp03`. The destination directory task runs everywhere.

| Task | stapp01 | stapp02 | stapp03 |
|---|---|---|---|
| Create `/opt/data` | Run | Run | Run |
| Copy `blog.txt` | Run | Skip | Skip |
| Copy `story.txt` | Skip | Run | Skip |
| Copy `media.txt` | Skip | Skip | Run |

## Run and validate

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ls -l /usr/src/data/{blog,story,media}.txt
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

A successful recap has **`failed=0` and `unreachable=0`** for all three servers. Each host normally skips two file-copy tasks. The exact `changed` count depends on existing state.

### Verify the deployed artifacts

```bash
ansible -i inventory stapp01 -m command -a "stat -c '%n %U:%G %a' /opt/data/blog.txt"
ansible -i inventory stapp02 -m command -a "stat -c '%n %U:%G %a' /opt/data/story.txt"
ansible -i inventory stapp03 -m command -a "stat -c '%n %U:%G %a' /opt/data/media.txt"
```

Expected metadata:

```text
/opt/data/blog.txt tony:tony 755
/opt/data/story.txt steve:steve 755
/opt/data/media.txt banner:banner 755
```

Use SHA-256 to compare each source file to its destination (full commands in [Commands.md](Commands.md)). The copied files should have identical contents.

## Key takeaways

- `hosts: all` determines where a play is considered; `when` determines whether each task runs on each host.
- `inventory_hostname` is available even with `gather_facts: false`.
- `copy.src` is resolved **on the Ansible control node**, unless `remote_src: true` is used.
- `become: true` permits writing to protected directories and setting ownership.
- `mode: '0755'` must be quoted to preserve correct octal semantics in YAML.
- Repeat executions should converge on the desired state without recopying unchanged files.

## Troubleshooting

| Symptom | Investigation |
|---|---|
| `UNREACHABLE` | Check SSH hostname, credentials, host-key trust |
| Sudo failure | Check Ansible become credentials and account privileges |
| Source file not found | Verify source exists on **jump host**, not managed host |
| All copy tasks skipped | Compare inventory aliases against `when` expressions |
| Wrong owner/group | Check matching account exists and `become: true` is active |
| Wrong mode | Quote `mode: '0755'` |

> **Security:** Do not commit real SSH passwords, sudo passwords, or private keys to GitHub.
