# Day 93 — Technical Notes: Ansible `when` Conditionals

## 1. Why conditional execution matters

A play targets a collection of hosts; conditional statements decide which individual tasks apply to each host. This supports heterogeneous environments without duplicating entire plays.

```yaml
- name: Copy file only to App Server 1
  ansible.builtin.copy:
    src: /usr/src/data/blog.txt
    dest: /opt/data/blog.txt
  when: inventory_hostname == 'stapp01'
```

The expression is evaluated independently on every host. False conditions produce a **skipped** result, not a failed result.

## 2. `hosts: all` versus `when`

- `hosts: all` selects all hosts in the inventory for the play.
- `when` selects the subset of those hosts where a task should execute.
- Three copy tasks can therefore target three different machines within one play.

Using `hosts: stapp01` for the entire play would violate this challenge's explicit `hosts: all` constraint.

## 3. `inventory_hostname` versus `ansible_hostname` versus `ansible_nodename`

| Variable | Meaning | Fact gathering required? |
|---|---|---|
| `inventory_hostname` | Host alias defined in the inventory | No |
| `ansible_hostname` | Short OS hostname (gathered fact) | Usually yes |
| `ansible_nodename` | OS node name (gathered fact) | Usually yes |

With `gather_facts: false`, `inventory_hostname` is ideal **when the inventory aliases exactly match** `stapp01`, `stapp02`, `stapp03`. If they are aliases such as `app1`, use the actual inventory values or gather facts before comparing remote system names.

## 4. `when` syntax

Correct:

```yaml
when: inventory_hostname == 'stapp01'
```

Do not add Jinja2 braces around a conditional:

```yaml
# Avoid
when: "{{ inventory_hostname == 'stapp01' }}"
```

`when` takes an expression, not a Jinja2 output string. More complex examples:

```yaml
when:
  - inventory_hostname == 'stapp01'
  - deploy_files | default(true)
```

A list of conditions is an implicit logical AND. For logical OR, use `when: condition_a or condition_b`.

## 5. Controller-to-managed-host copy semantics

`ansible.builtin.copy` reads `src` on the **controller**, then transfers it to `dest` on the selected managed host. `remote_src: true` changes this behavior and is not appropriate for the given `/usr/src/data` sources on the jump host.

## 6. File ownership, privilege escalation, and permissions

Remote users `tony`, `steve`, and `banner` may not have permission to create `/opt/data`. The play uses `become: true` to execute remote tasks with elevated privilege while explicitly assigning the correct final owner and group.

The required mode `0755` means:

```text
owner:  read, write, execute (7)
group:  read, execute        (5)
others: read, execute        (5)
```

These are the lab's required modes for text files; in production, `0644` would often be safer because regular text does not need execute permissions. YAML permissions should be quoted as `'0755'`.

## 7. Idempotency and validation

Ansible's `copy` module checks whether file contents/metadata already match. On a subsequent run, unmodified targets normally return `ok` rather than `changed`. To verify byte-level equality, compare SHA-256 checksums from the controller source and the remote destination.

Success means:

1. All three hosts are reachable and tasks complete without failure.
2. Each server receives its matching file at `/opt/data`.
3. Each file has the required owner, group, and mode.
4. Source and destination hashes match.

The `skipped` count is a feature of the design: every host should skip the two copy tasks intended for other hosts.

## 8. Troubleshooting matrix

| Symptom | Root cause | Fix |
|---|---|---|
| Every copy task skipped | `inventory_hostname` mismatches inventory alias | Inspect `ansible-inventory --graph` |
| Source file missing | Not present on controller | Check `/usr/src/data` on jump host |
| Permission denied in `/opt` | Insufficient privilege | Verify `become: true` and noninteractive sudo |
| User/group lookup failure | Wrong remote account or group | Check `id tony`, `id steve`, or `id banner` on target |
| Connection failure | SSH credentials or host key | Test Ansible ping and SSH trust |
| Unexpected mode | Incorrect octal specification | Use quoted `'0755'` |
| Hash mismatch | Wrong source or unexpected file modification | Inspect the corresponding `src`/`dest` mapping |

## 9. How to improve the design in a real project

For a larger deployment, replace repeated copy tasks with a host-specific data structure in `host_vars`, a mapping keyed by `inventory_hostname`, or inventory groups and a single task. In this KodeKloud challenge, explicit `when` statements are retained because conditional execution is the feature being assessed.

For production: avoid plaintext inventory credentials, prefer Ansible Vault/SSH keys, restrict sudo access, use least-privilege file modes, and validate deployments through CI.

## 10. Interview preparation

**Q: What does `when` do in Ansible?**  
A: It evaluates an expression for each managed host and executes or skips the associated task based on the result.

**Q: Is `when` evaluated on the controller or on the remote node?**  
A: Ansible evaluates the conditional using the current host's variables while orchestrating task execution from the controller.

**Q: What is the difference between `hosts` and `when`?**  
A: `hosts` establishes a play's host set; `when` controls task execution within that set.

**Q: Why use `inventory_hostname` here?**  
A: It's reliably available without fact gathering and matches the known inventory aliases.

**Q: What happens if a condition is false?**  
A: The task is skipped for that host; that is not an error.

**Q: How can you ensure the transferred file is identical?**  
A: Compare checksums of the controller source and remote destination.

**Q: Why is `become` necessary?**  
A: To create protected destination directories and set target ownership.

**Q: How do you prevent repeated unnecessary copies?**  
A: Use `ansible.builtin.copy` with stable content/metadata; it is idempotent under unchanged conditions.

---

**Milestone:** Day 93 passed — conditional host-specific Ansible deployment completed.
