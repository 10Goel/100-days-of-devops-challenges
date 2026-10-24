# Day 88 — Technical Notes: Apache Automation with Ansible

## What Day 88 tests

A single playbook must install the `httpd` RPM package, ensure Apache is running across three managed hosts, insert specified lines into `/var/www/html/index.html` **using `blockinfile`**, and set `apache:apache` ownership with permissions `0644`.

## 1. Structure of a declarative playbook

- `hosts: all` targets every managed host present in the lab inventory; verify that it contains only the three expected application servers.
- `become: true` elevates remote tasks because package management, service control and writes under `/var/www` require additional privileges.
- `gather_facts: true` gathers host facts before task execution.
- `tasks` run sequentially per host, while Ansible can execute them concurrently across multiple hosts.

## 2. Modules and responsibilities

| Module | Desired state | Practical role |
|---|---|---|
| `ansible.builtin.yum` | `name: httpd`, `state: present` | Install Apache if absent |
| `ansible.builtin.service` | `state: started`, `enabled: true` | Start Apache now and at boot |
| `ansible.builtin.file` | `state: directory`, `0755` | Ensure document root exists |
| `ansible.builtin.blockinfile` | Managed multiline block | Insert required page content |
| `ansible.builtin.file` | `owner`, `group`, `mode: '0644'` | Enforce final file metadata |

`yum` is an explicit module rather than the distro-neutral `package` module. On modern EL systems, its backend may be DNF, depending on Ansible and the managed system.

## 3. Why `blockinfile`?

`blockinfile` manages **a section** of a text file without taking ownership of the entire file. Default results look like:

```text
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!
This is Nautilus sample file, created using Ansible!
Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```

The markers identify the block to modify on repeat runs. Avoid independently editing those markers, or the next playbook run may add a second block.

**Module comparison:**

- `copy`: write a complete file from a controller-side source or supplied content.
- `template`: render an entire file through Jinja2 variables.
- `lineinfile`: manage individual matching lines.
- `blockinfile`: manage a bounded multiline section.
- `file`: manage file metadata/directory existence rather than file contents.

### Note about HTML

The challenge demands `blockinfile` and specific plain text. Default comment-style markers are appropriate for demonstrating the module and satisfying the textual requirement; for a real production HTML page, consider valid HTML comment markers (`<!-- ... -->`) or a template to avoid raw markers appearing in rendered output.

## 4. Ordering and dependencies

Install `httpd` first so its `apache` user/group exist. Ensure the service is running, create the document root, insert the required content, then set final metadata. If a previous file had different ownership, the final `file` task corrects it.

## 5. Linux permissions

`0644` means:

| Entity | Read | Write | Execute |
|---|---|---|---|
| Owner | Yes | Yes | No |
| Group | Yes | No | No |
| Others | Yes | No | No |

The initial zero denotes octal notation. Quote `mode: '0644'` to prevent unexpected YAML number parsing. `apache:apache` specifies both user and group, not just the effective process identity.

## 6. Idempotency

When the installed package, running service, managed block and file metadata already match the declared state, a repeat run should normally make no changes. If `blockinfile` reports `changed` repeatedly, inspect whether another tool edits the managed section or alters line endings/markers.

## 7. Validation strategy

Check each layer independently:

1. **Inventory and SSH:** `ansible-inventory -i inventory --graph` then `ansible -i inventory all -m ping`.
2. **Package:** `rpm -q httpd` on each managed host.
3. **Runtime:** `systemctl is-active httpd` and optionally `systemctl is-enabled httpd`.
4. **Content:** `cat /var/www/html/index.html` on each managed host; compare spelling and punctuation exactly.
5. **Metadata:** `stat -c '%U:%G %a' /var/www/html/index.html` must show `apache:apache 644`.
6. **Play recap:** `failed=0` and `unreachable=0` for each host.

The lab validation specifically uses:

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

## 8. Troubleshooting

| Symptom | Possible cause | Next check |
|---|---|---|
| `UNREACHABLE` | SSH credentials, host trust, DNS or network | Manual SSH / Ansible ping |
| `Missing sudo password` | Privilege escalation credentials absent | Check effective inventory or approved sudo setup |
| `User apache not found` | Package installation failed or wrong system | Inspect yum task and user database |
| `Permission denied` | No `become` or insufficient privileges | Check sudo and file ownership |
| Service refuses to start | Configuration, port collision, dependencies | `systemctl status httpd` and journal |
| Content differs | Typo, wrong path, modified block markers | Compare the exact three lines |
| Wrong file mode | Unquoted mode or unrelated process changes | Check `stat` and the final file task |

## 9. Interview preparation

**Q: What is the difference between `state: started` and `enabled: true`?**  
`started` ensures the service is running now; `enabled` configures startup at boot.

**Q: Why does `blockinfile` add BEGIN/END lines?**  
To identify a managed section for future edits, rather than replace the whole file.

**Q: Why use `become: true`?**  
To run remote privileged operations via approved escalation (typically sudo), independently of the initial SSH user.

**Q: What confirms a playbook executed successfully?**  
A recap with `failed=0` and `unreachable=0`, plus direct checks of the required system state.

**Q: Is an HTTP service that is `active` necessarily reachable by clients?**  
No. Bind addresses, ports, firewall, network path and proxies still matter.

**Q: What does idempotency mean here?**  
Repeating the playbook against an already-correct system should normally avoid unnecessary changes.

## 10. Scope and safety

This lab did not require modifying HTTP ports, Apache virtual hosts, SELinux policy, or firewall rules. Keep credentials/private keys out of GitHub, and use secure secrets management in real deployments.

**Completion:** Day 88 passed, as reported by the learner. Output examples in this documentation are explanatory, not claims of actual captured console logs.
