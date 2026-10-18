# Day 82 — Ansible Inventory: Technical Notes

## 1. Key vocabulary

| Term | Meaning in this lab |
|---|---|
| Control node | The jump host, where Ansible is run as `thor` |
| Managed node | App Server 1, identified as `stapp01` |
| Inventory | File listing target hosts, groups, and connection variables |
| Playbook | `playbook.yml`, a YAML specification of operations on remote hosts |
| Play | A mapping between target hosts (`hosts:`) and tasks |
| Task | A unit of work, such as installing a package or starting a service |
| Module | Ansible operation, e.g. `ping`, `package`, `service` |
| Privilege escalation | Running tasks as another user, commonly root, using `become` |

## 2. Inventory versus playbook

Inventory answers **where and how to connect**. A playbook answers **what work should be done**.

```text
inventory                             playbook.yml
  |                                      |
  +-- stapp01                           +-- hosts: all
  +-- SSH user: tony                     +-- gather facts
  +-- SSH authentication               +-- install httpd
  +-- become authentication            +-- start httpd
            \                            /
             \                          /
               ansible-playbook -i inventory playbook.yml
                                  |
                                  v
                             App Server 1
```

The group name in the inventory matters when a playbook's `hosts:` targets a named group. For `hosts: all`, any properly inventoried host is eligible, subject to other limits or conditions.

## 3. INI inventory essentials

```ini
[app_servers]
stapp01 ansible_connection=ssh ansible_user=tony ansible_password="<SSH_PASSWORD>" ansible_become_password="<SUDO_PASSWORD>"
```

- `stapp01`: inventory hostname and SSH destination in this lab.
- `[app_servers]`: optional grouping for targeting and organization.
- `ansible_connection=ssh`: use OpenSSH transport.
- `ansible_user=tony`: account to authenticate as on the remote Linux host.
- `ansible_password`: SSH password, if password authentication is used.
- `ansible_become_password`: sudo/escalation password, if required by the playbook and remote sudo policy.

Some playbooks set `become: true` directly. A password variable by itself does **not** turn on escalation. If `become` is enabled, Ansible uses the configured method (commonly sudo). For this lab, the inventory had to support the already supplied playbook without additional arguments.

### Password storage

Putting plaintext secrets into INI inventory is expedient for short-lived training environments but unsafe for source control. In long-lived systems prefer:

- SSH keys and appropriate host-key verification.
- Ansible Vault for protected credentials.
- Dedicated credential stores integrated with automation platforms.
- Least-privilege sudo rules rather than giving wide administrative permissions.

## 4. What Ansible does during execution

1. Parses the `-i inventory` target definitions.
2. Reads `playbook.yml` and resolves each play's `hosts:` pattern.
3. Establishes SSH access to the matched managed host.
4. Gathers system facts if fact gathering is enabled.
5. Executes the declared tasks and optionally elevates privileges.
6. Prints per-task status and the final recap.

Unlike an SSH login test, `ansible -m ping` validates that Ansible can authenticate, invoke its remote module, and receive a response. It is not an ICMP echo check.

## 5. Understanding the actual recap

```text
stapp01 : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

- `ok=3`: three tasks completed successfully; changed tasks also count as OK.
- `changed=2`: two tasks reported that they altered remote state (Apache installation and service startup).
- `unreachable=0`: Ansible could contact the host.
- `failed=0`: no task reported an unhandled failure.
- `skipped=0`: no task was skipped.
- `rescued=0`, `ignored=0`: no rescued or ignored task failures were reported.

**Do not equate `changed=2` with two failures.** Changes are normal during initial configuration.

## 6. Idempotency

Ansible modules generally describe the desired state, instead of blindly reapplying shell commands.

For example:

```yaml
- name: Install httpd
  ansible.builtin.package:
    name: httpd
    state: present

- name: Start httpd
  ansible.builtin.service:
    name: httpd
    state: started
```

This snippet is a **conceptual example**, not a claim about the exact unshown contents of the supplied playbook. If the package is already installed and service already started, a subsequent run often reports no changes. Idempotency depends on each module and task implementation.

## 7. Troubleshooting map

| Symptom | Likely cause | First check |
|---|---|---|
| No hosts matched | Inventory group differs from `hosts:` | Inspect `playbook.yml` and `ansible-inventory --graph` |
| `UNREACHABLE` | SSH authentication, DNS or transport failure | `ssh tony@stapp01`, host spelling, connection variables |
| Host key verification failed | Unknown/untrusted SSH server identity | Verify fingerprint, then establish trusted `known_hosts` |
| `Permission denied` | Incorrect remote account password or access | Confirm current lab credentials; avoid repeated guesses |
| `Missing sudo password` | Playbook uses escalation without usable auth | Inspect `become` and `ansible_become_password` |
| `sshpass` missing | Password-based SSH support unavailable | `command -v sshpass` |
| `Syntax Error` | Invalid playbook YAML or module syntax | `--syntax-check` and inspect indicated line |
| `failed=1` for package/service | Missing privileges, repository issue or service startup failure | Read first failing task's output, not just the recap |

## 8. Practical operational guidance

- Use the exact inventory path required by automation or validation.
- Name hosts consistently; do not replace `stapp01` with an unrelated alias when the task requires that literal hostname.
- Verify the playbook target before choosing an inventory group.
- Test inventory parsing, connection, syntax, and execution in that order.
- Restrict credential-bearing files (`chmod 600`), and never publish secrets or unredacted `ansible-inventory --list` output.
- Keep environment configuration separate from playbook logic so that the same automation can target different hosts safely.

## 9. Interview-ready questions

**Why use an inventory?** To select managed hosts and centralize connection metadata independently of playbook tasks.

**How is `ansible -m ping` different from `ping`?** Ansible's module verifies remote module execution through its configured transport; normal `ping` tests ICMP reachability.

**What is the difference between `ansible_user` and `ansible_become_user`?** The former is the SSH login account; the latter is the identity used after escalation (normally root if `become` is enabled).

**Why can a play succeed with `changed=2`?** `changed` reports desired-state modifications, not errors.

**Why should inventories with passwords stay out of Git?** Inventory variables can expose reusable credentials; use secret-management tools and restrict access.

---

**Day 82 result:** KodeKloud challenge passed, Ansible recap `ok=3 changed=2 unreachable=0 failed=0`.
