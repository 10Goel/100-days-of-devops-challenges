# Day 84 — Ansible Notes: Multi-Host File Distribution

## 1. Ansible architecture

The **control node** runs Ansible and owns `/usr/src/sysops/index.html`. The **managed nodes** (`stapp01`, `stapp02`, `stapp03`) receive the file at `/opt/sysops/index.html` over SSH. Ansible does not ordinarily require persistent agents on these machines.

## 2. Inventory and host groups

```ini
[app_servers]
stapp01 ansible_user=tony
stapp02 ansible_user=steve
stapp03 ansible_user=banner
```

- `app_servers` is the group targeted by the play's `hosts` key.
- `ansible_host` identifies the SSH destination, if different from the inventory alias.
- `ansible_user` is the remote SSH login identity.
- `ansible_password` is the SSH password when password authentication is used.
- `ansible_become_password` is the escalation password, which may be different.

Protect inventories containing sensitive variables and prefer Vault, key authentication or a secrets manager in production.

## 3. `file` versus `copy`

```yaml
- name: Ensure directory exists
  ansible.builtin.file:
    path: /opt/sysops
    state: directory
    mode: '0755'
```

The `file` module ensures the remote directory exists and has the desired metadata.

```yaml
- name: Copy website file
  ansible.builtin.copy:
    src: /usr/src/sysops/index.html
    dest: /opt/sysops/index.html
    mode: '0644'
```

By default, `copy.src` refers to the control node (`remote_src: false`), while `dest` refers to each managed node. Setting `remote_src: true` would change the meaning of `src` and is incorrect for this challenge.

## 4. Why privilege escalation matters

`/opt` is often root-owned. Logging in as `tony`, `steve`, or `banner` does not automatically provide write access. `become: true` enables authorized elevated execution of remote tasks, commonly through sudo. It does not affect where the source file is read.

## 5. File modes

- `0755` for `/opt/sysops`: owner may read/write/traverse; others may read/traverse.
- `0644` for `index.html`: owner may read/write; others may read.
- Quote modes in YAML to avoid ambiguous interpretation.

## 6. Idempotency

Declarative modules aim to converge systems toward desired state. On the first run, creating a missing directory and copying a missing file normally reports `changed`. On a repeated run with matching content and metadata, tasks normally report `ok`. No exact `changed` count is guaranteed because the initial state may differ.

## 7. Authentication versus host trust

SSH first verifies the **identity of the remote server** against trusted host keys and separately authenticates the **login account**. With password-based SSH and `sshpass`, first-time host-key verification may prevent Ansible from connecting. Establish trust interactively from the control node, verify fingerprints against trusted lab information, then run Ansible. Avoid disabling host-key verification.

## 8. Verification and integrity

`ansible.builtin.stat` checks remote file existence and metadata:

```bash
ansible -i inventory app_servers -b -m stat -a 'path=/opt/sysops/index.html'
```

Compare content hashes:

```bash
sha256sum /usr/src/sysops/index.html
ansible -i inventory app_servers -b -m command -a 'sha256sum /opt/sysops/index.html'
```

All four SHA-256 digests must match. A read-only `command` invocation may still be marked `CHANGED` by Ansible; that alone does not mean it modified the file.

## 9. Troubleshooting guide

| Error | Root cause to investigate | Resolution |
|---|---|---|
| `UNREACHABLE` | SSH network or authentication | Check usernames, hostnames, passwords and SSH |
| `sshpass` host-key failure | Remote server not trusted | Verify its fingerprint and add via interactive SSH |
| Sudo password required | Missing authorized become credentials | Configure `ansible_become_password` or approved sudo access |
| `/opt/sysops` permission denied | Remote user lacks filesystem access | Check `become` permissions |
| Source file missing | Controller-side source absent | Inspect `/usr/src/sysops/index.html` on jump host |
| Only one server targeted | Inventory grouping mismatch | Inspect `ansible-inventory --graph` |
| Wrong destination | Unexpected `dest` path | Use `/opt/sysops/index.html` |

## 10. Related modules

- `copy`: transfers file content to managed nodes.
- `fetch`: retrieves files from managed nodes to the controller.
- `template`: renders Jinja2 templates with variables.
- `file`: manages directory/file state and metadata.
- `synchronize`: rsync-based synchronization when installed and appropriately configured.

## 11. Interview questions

**Q: Where is `copy.src` located?** On the control node by default.

**Q: What does `hosts: app_servers` mean?** Run the play against every host in that inventory group.

**Q: How are SSH authentication and `become` different?** The first logs into a machine; the second provides authorized elevated privileges for tasks.

**Q: What is the significance of `failed=0` and `unreachable=0`?** No reported task failures or unreachable hosts in the recap; verify final filesystem state separately.

**Q: How can you prove all destination files match the original?** Compare SHA-256 digests of the source and all remote copies.

**Q: Why might a second playbook execution show `changed=0`?** The system already matches the declared content and file modes.

## 12. Production improvements

Use Ansible Vault or a dedicated secrets manager, SSH keys, least-privilege sudo policies, environment-specific inventories, and checksum verification in CI/CD. Never commit populated inventory files containing passwords to a public Git repository.

---
**Outcome:** Day 84 passed. Example configurations here are reproducible, sanitized documentation rather than a claimed verbatim terminal recording.
