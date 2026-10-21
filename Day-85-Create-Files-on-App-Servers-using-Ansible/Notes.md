# Day 85 — Ansible Notes: Host Variables, Ownership and Modes

## Control node and managed nodes
The jump host (`thor`) runs Ansible; `stapp01`, `stapp02`, and `stapp03` are managed nodes. A single inventory group `app_servers` allows one playbook to target the three hosts.

## Host variables and Jinja2
Each inventory record has `file_owner` unique to that host. `owner: "{{ file_owner }}"` and `group: "{{ file_owner }}"` substitute the host's effective variable at execution time. This avoids three nearly identical plays and enables scaling through `host_vars` or group variables in larger deployments.

## File creation approaches
`ansible.builtin.copy` with `content: ''` creates an empty file and clears preexisting contents if necessary. `ansible.builtin.file` with `state: touch` creates a missing file but will not truncate an existing file. For a guaranteed **blank file**, the `copy` approach is appropriate.

## Unix permissions: 0777
Owner, group, and others each receive `rwx` because the octal digit `7 = 4 (read) + 2 (write) + 1 (execute)`. Thus `0777` yields `-rwxrwxrwx` for a regular file. Quote `mode: '0777'` to avoid YAML numeric parsing ambiguity. Executable and world-writable text files are generally unsafe; Day 85 explicitly requires this value as a practice exercise.

## Ownership versus SSH identity
`ansible_user=tony` chooses which user logs in over SSH. `owner` and `group` choose the final filesystem metadata, potentially set using root privileges. `become: true` triggers sudo or another configured privilege escalation method; `ansible_become_password` may be different from the SSH password.

## Idempotency
An idempotent state ensures repeated playbook runs don't produce unnecessary modifications. `copy` compares desired content/metadata with the existing target. A second execution with unchanged state normally reports `ok` and `changed=0`. A `file: touch` task may update timestamps repeatedly.

## SSH host keys and sshpass
SSH host-key trust verifies the identity of the destination host; SSH password authentication verifies the identity of the remote user. With `sshpass`, unknown host keys can cause a host-checking failure. Validate fingerprints using trusted lab infrastructure and save them in the jump host's `known_hosts` through an interactive SSH connection. Don't disable host checking to suppress the error.

## Verify all requirements
```bash
ansible-inventory -i inventory --graph
ansible -i inventory app_servers -m ping
ansible-playbook -i inventory playbook.yml
ansible -i inventory app_servers -b -m command -a "stat -c '%n %U:%G %a %s bytes' /home/webdata.txt"
```
Correct final metadata:

| Host | Owner:Group | Mode | Size |
|---|---|---|---|
| stapp01 | tony:tony | 777 | 0 bytes |
| stapp02 | steve:steve | 777 | 0 bytes |
| stapp03 | banner:banner | 777 | 0 bytes |

The ad-hoc `command` module may print `CHANGED` even for read-only checks; this is not evidence it altered the file. Playbook success requires `failed=0` and `unreachable=0` on every host.

## Troubleshooting decision table
| Error | Likely cause | Fix |
|---|---|---|
| `UNREACHABLE` | SSH/DNS/authentication | Test host, username, password, network |
| `sshpass` host checking | Untrusted host key | Verify fingerprint, add to `known_hosts` |
| `Missing sudo password` | Become authentication missing | Set correct `ansible_become_password` |
| Wrong file owner | Wrong `file_owner` variable | Review host-specific inventory entries |
| Permission not 777 | Unquoted or incorrect mode | `mode: '0777'` |
| Nonempty file | Using `touch` on existing file | Use `copy` with empty content |

## Interview questions
**Why use a host variable?** To apply per-server differences without duplicating the playbook.

**Does `ansible -m ping` test sudo?** No; standard ping tests Ansible remote execution and Python, not necessarily privilege escalation.

**Why prefer `copy` here?** It enforces empty content, ownership, group and permissions as desired state.

**Why quote permission modes?** YAML may otherwise interpret numeric-looking values unexpectedly.

**What makes 0777 risky?** Any local user can modify the file. Prefer the minimum access necessary in production.

## Security and portfolio notes
Never upload actual lab passwords or private keys to GitHub; this documentation uses placeholders. In real projects, use SSH keys, Ansible Vault or a secret store and least-privilege sudo policies.

**Completion:** Day 85 passed according to the user; sample outcomes here are requirements-based, not claimed verbatim console transcripts.
