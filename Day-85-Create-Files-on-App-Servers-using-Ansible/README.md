# Day 85 — Ansible File Creation, Ownership & Permissions

**KodeKloud 100 Days of DevOps · Successfully completed**

## Objective
Create an empty `/home/webdata.txt` file across all three Stratos DC application servers. File permissions must be **0777**, with server-specific user and group ownership. Use a single reusable Ansible playbook.

| Target | Login | Required owner:group | File | Mode |
|---|---|---|---|---|
| stapp01 | tony | tony:tony | `/home/webdata.txt` | `0777` |
| stapp02 | steve | steve:steve | `/home/webdata.txt` | `0777` |
| stapp03 | banner | banner:banner | `/home/webdata.txt` | `0777` |

**Required files:** `/home/thor/playbook/inventory` and `/home/thor/playbook/playbook.yml` on the jump host.

**Validator command:** `ansible-playbook -i inventory playbook.yml` (from `/home/thor/playbook`).

## Architecture
```text
Jump host (thor)
 ├── inventory: target nodes + file_owner variables
 └── playbook.yml: one play, one copy task
       ├── stapp01 → /home/webdata.txt (tony:tony, 0777, 0 bytes)
       ├── stapp02 → /home/webdata.txt (steve:steve, 0777, 0 bytes)
       └── stapp03 → /home/webdata.txt (banner:banner, 0777, 0 bytes)
```

## Implementation

### Inventory
Place the following template in `/home/thor/playbook/inventory`. Replace placeholders with current lab credentials; never commit populated secret-bearing inventory files.

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password='APP1_SSH_PASSWORD' ansible_become_password='APP1_SUDO_PASSWORD' file_owner=tony
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='APP2_SSH_PASSWORD' ansible_become_password='APP2_SUDO_PASSWORD' file_owner=steve
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password='APP3_SSH_PASSWORD' ansible_become_password='APP3_SUDO_PASSWORD' file_owner=banner
```

Restrict permissions using `chmod 600 /home/thor/playbook/inventory`.

### Playbook
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

### Run
```bash
cd /home/thor/playbook
ansible-inventory -i inventory --graph
ansible -i inventory app_servers -m ping
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

### Verify
```bash
ansible -i inventory app_servers -b -m command -a "stat -c '%n %U:%G %a %s bytes' /home/webdata.txt"
```

Expected host-specific results: `tony:tony 777 0 bytes`, `steve:steve 777 0 bytes`, `banner:banner 777 0 bytes`. Verify `failed=0` and `unreachable=0` for all three hosts in the recap. Task counts vary with the initial state.

## Why this solution
- Inventory variable `file_owner` supplies different values for a common playbook.
- `ansible.builtin.copy` with `content: ''` guarantees **zero-byte contents**, unlike `file: state=touch` which doesn't clear existing data.
- `become: true` permits privileged file creation and ownership changes.
- `mode: '0777'` preserves octal interpretation in YAML; this world-writable permission is required **only for the lab** and not normally appropriate for production.
- Desired-state execution should be idempotent when file content, owner, group and mode are unchanged.

## Troubleshooting
| Symptom | Action |
|---|---|
| SSH authentication denied | Verify login users and password variables |
| Host-key error with sshpass | Verify server fingerprint, accept known host key on jump host |
| Sudo password error | Check authorization and `ansible_become_password` |
| Wrong ownership | Check `file_owner` for the specific host |
| File not empty | Ensure `copy: content: ''` |
| Wrong permission bits | Quote `'0777'` in YAML |

**Result:** User confirmed Day 85 passed. See [Commands.md](Commands.md) and [Notes.md](Notes.md) for a detailed operational guide and conceptual reference.
