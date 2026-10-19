# Day 83 — Ansible Notes: Inventories, SSH, and File Automation

## 1. Control node vs managed node

Ansible runs on a **control node** (here the Nautilus jump host as `thor`) and connects to a **managed node** (here `stapp03`) to execute tasks. Ansible typically does not require a persistent agent on a managed Linux server; it connects using SSH and executes the necessary module payloads.

## 2. Inventory fundamentals

An INI inventory defines host aliases, host groups, and connection variables.

```ini
[app_servers]
stapp03 ansible_host=stapp03 ansible_user=banner ansible_connection=ssh
```

- `stapp03`: the **inventory hostname** used by `hosts: stapp03`.
- `ansible_host`: the DNS name or IP address to contact; redundant when identical to the inventory hostname.
- `ansible_user`: Linux SSH login name.
- `ansible_connection`: transport, typically `ssh` for Linux.
- `ansible_password`: optional SSH password; store securely.
- `ansible_become`: whether to elevate privileges (not required here).
- `ansible_become_password`: password for privilege escalation, distinct from the SSH password even when they happen to match.

`hosts: stapp03` targets the inventory host directly, whether it is in `[app_servers]` or another group.

## 3. The difference between SSH trust and SSH authentication

SSH performs two conceptually separate checks:

1. **Host-key verification**: Am I contacting the intended server? `~/.ssh/known_hosts` records accepted public host keys.
2. **User authentication**: May I log in as `banner`? This may use a password or SSH key.

On Day 83, a password-backed Ansible attempt failed because the host's key was not trusted and `sshpass` could not answer the interactive host-key verification prompt. Connecting once interactively as `thor` after verifying the host fingerprint recorded the host key. Subsequent Ansible authentication could then proceed normally.

Do **not** solve this by globally disabling host-key checking: doing so removes an important defense against server impersonation.

### Useful diagnostics

```bash
ssh-keygen -F stapp03
ssh banner@stapp03
ansible -i inventory stapp03 -m ping
```

Use the jump host account for these commands; SSH trust files are per-account.

## 4. `ansible.builtin.file` module

The `file` module manages files, directories, links, ownership, and permissions.

```yaml
- name: Create empty file
  ansible.builtin.file:
    path: /tmp/file.txt
    state: touch
    mode: '0644'
```

### Key states

| State | Meaning |
|---|---|
| `touch` | Create missing file or update timestamps of existing file |
| `file` | Manage metadata of an existing file; it does not create a missing file |
| `directory` | Ensure directory exists |
| `absent` | Remove file/directory (destructive — use with care) |
| `link` | Create/manage symbolic link |
| `hard` | Create/manage hard link |

**Important distinction:** `touch` does not force an existing file to become empty. It creates a zero-byte file only when the path did not exist. If strict zero-byte enforcement is needed regardless of preexisting content, a different approach must be chosen deliberately; the lab expected creation of a new empty file.

### Permissions

`mode: '0644'` means owner read/write, group read, other read. Quote the mode so YAML and Ansible interpret it consistently.

## 5. Privilege escalation (`become`)

Creating `/tmp/file.txt` as `banner` generally does not require root permissions, so the example play uses `become: false`. Use `become: true` only when the target action requires privileged access. Never assume the SSH password is equivalent to a sudo password, and do not add unnecessary sudo rights.

## 6. Inventory validation versus execution

| Check | Command | What it proves |
|---|---|---|
| Inventory graph | `ansible-inventory -i inventory --graph` | Host selection resolves correctly |
| Reachability | `ansible -i inventory stapp03 -m ping` | SSH and Python-based remote module execution work |
| Syntax | `ansible-playbook -i inventory playbook.yml --syntax-check` | YAML/playbook structure parses |
| Execution | `ansible-playbook -i inventory playbook.yml` | Tasks execute with actual connection variables |
| File facts | `ansible -i inventory stapp03 -m stat -a 'path=/tmp/file.txt'` | File is present and can be inspected |

A syntax check does **not** prove that the remote host is reachable or that the intended file will be created.

## 7. Idempotency and `changed`

Ansible modules often express desired state. Running an idempotent playbook twice usually makes changes only the first time. `file` with `state: touch` is a nuance: it can update timestamps on an existing file and may report changes on repeated runs. Therefore, one should not interpret a `changed` result as proof that a new file was created during each run.

## 8. Common failure modes

| Symptom | Likely cause | Diagnostic or fix |
|---|---|---|
| `UNREACHABLE` / host-key checking error | Host not in trusted `known_hosts` | Verify fingerprint and establish interactive SSH trust |
| Authentication failure | Wrong `ansible_user` or password/key | Test SSH as exact remote user |
| `sshpass` missing | Password authentication requires helper | `command -v sshpass` |
| Host pattern matches no hosts | Inventory alias differs from play `hosts:` | Check inventory graph and play target |
| Permission denied on `/tmp/file.txt` | Existing file owned by another user or filesystem restrictions | Inspect remote file ownership and permissions before changing |
| YAML parse error | Bad indentation/colon/quoting | Run `--syntax-check` |
| File still has data | `touch` doesn't truncate existing files | Inspect `stat`; use an explicit content-management approach only if authorized |

## 9. Security guidance for GitHub

- Never publish real `ansible_password`, `ansible_become_password`, SSH private keys, or permanent lab tokens.
- Commit sanitized inventory examples only, with placeholders.
- Prefer Ansible Vault, SSH keys, and organization-managed secrets for real infrastructure.
- Limit permissions on inventory files containing credentials (for example, `chmod 600`).
- Do not disable SSH host-key checking merely to make automated commands pass.

## 10. Interview practice

**Q: Why did Ansible ping initially fail if direct SSH worked later?**  
The SSH host key was not trusted initially, and the password-driven automation could not respond to the trust prompt. A verified interactive SSH session populated `known_hosts` for the control user.

**Q: Does `state: touch` erase existing content?**  
No. It creates a missing file or updates timestamps on an existing one.

**Q: What's the difference between an inventory hostname and `ansible_host`?**  
The inventory hostname is the logical name selected by plays; `ansible_host` tells Ansible where to connect.

**Q: Does `--syntax-check` execute tasks?**  
No. It parses and validates the playbook structure without applying the remote changes.

**Q: Why is `become: false` appropriate here?**  
An ordinary user can normally create a new file inside `/tmp`, so privilege escalation is unnecessary.

## Outcome

Day 83 was **confirmed passed** in KodeKloud after the inventory and playbook workflow was completed. The meaningful debugging lesson was treating SSH host trust separately from SSH user authentication.
