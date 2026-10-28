# Day 92 — Technical Notes: Ansible Roles, Jinja2 and Template Deployment

## 1. Why roles matter

An Ansible role groups related tasks, templates, handlers, defaults and other resources into a reusable unit. A playbook declares the target host and calls the role; the role owns implementation details.

```yaml
roles:
  - role: /home/thor/ansible/role/httpd
```

The lab uses a **nonstandard `role/` directory**, rather than the conventional `roles/`. An absolute role path ensures discovery without extra command-line options, meeting the validator's requirement.

## 2. Role entry points

- `tasks/main.yml`: list of tasks executed when the role runs.
- `templates/`: Jinja2 inputs used by `ansible.builtin.template`.
- `handlers/main.yml`: optional tasks triggered by `notify`, typically service restarts after configuration changes.
- `defaults/main.yml`: optional low-precedence default variables.
- `vars/main.yml`: optional role variables, generally with higher precedence than defaults.

A role task file should **not** include a top-level `hosts:` declaration. Host selection belongs in the playbook.

## 3. Jinja2 and `inventory_hostname`

The source template:

```jinja2
This file was created using Ansible on {{ inventory_hostname }}
```

Rendered for `stapp01`:

```text
This file was created using Ansible on stapp01
```

`inventory_hostname` is Ansible's host identifier from inventory. It is not necessarily identical to the system's OS hostname (`ansible_hostname`) or network address (`ansible_host`). The task explicitly requests `inventory_hostname`.

## 4. `template` versus `copy`

| Module | Typical use |
|---|---|
| `ansible.builtin.template` | Render dynamic Jinja2 text using variables |
| `ansible.builtin.copy` | Copy static controller-side files or exact literal content |
| `ansible.builtin.lineinfile` | Manage one line in an existing file |
| `ansible.builtin.blockinfile` | Manage an identified multiline block |
| `ansible.builtin.file` | Manage files, directories, ownership and modes |

For this challenge, `template` is required because the content depends on the current inventory host.

## 5. Template lookup inside a role

With the template located at:

```text
/home/thor/ansible/role/httpd/templates/index.html.j2
```

this task can simply use `src: index.html.j2`:

```yaml
- name: Deploy dynamic webpage
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: tony
    group: tony
    mode: '0655'
```

Ansible resolves the template relative to the role's `templates` directory.

## 6. Understanding mode `0655`

Linux permission bits: owner `6` = read + write; group `5` = read + execute; others `5` = read + execute.

```text
-rw-r-xr-x
```

This is unusual for an `.html` file but must match the lab's exact requirement. Always quote octal modes in YAML (`'0655'`) to avoid ambiguity.

## 7. Privilege escalation and ownership

`become: true` allows installation of RPM packages, service management, and writing into `/var/www/html`. It **does not** force the deployed file to be root-owned: the template task explicitly sets `owner: tony` and `group: tony`.

The existing inventory must contain a valid noninteractive SSH/sudo configuration because KodeKloud executes:

```bash
ansible-playbook -i inventory playbook.yml
```

without `--ask-pass`, `--ask-become-pass`, or other extra arguments.

## 8. Idempotency and clean-state recovery

- `yum: state=present` installs Apache only if needed.
- `service: state=started` ensures the service runs; `enabled: true` configures boot startup.
- `template` writes the file when content or managed attributes differ.
- When the validator deletes `httpd` and `index.html`, the role must recreate both from scratch.
- When the role is run again unchanged, it should normally converge without modifying resources.

**Key distinction:** A playbook succeeding against a preconfigured host is not proof that it works from a clean state. The lab explicitly tests that difference.

## 9. Troubleshooting examples

| Symptom | Explanation | Resolution |
|---|---|---|
| `role was not found` | Role path not discoverable | Use correct absolute path or configure `roles_path` |
| `Could not find template` | File absent / misplaced | Put `index.html.j2` inside role's `templates/` |
| Output shows literal `{{ inventory_hostname }}` | Template wasn't rendered | Use the `template` module, not a static copy |
| Wrong hostname | Hardcoded value or wrong variable | Use `inventory_hostname` |
| Wrong file owner/group | Incorrect task metadata | Set `tony:tony` |
| Mode reported `644` or `755` | Incorrect permissions | Set quoted `'0655'` |
| `UNREACHABLE` | SSH connection problem | Inspect inventory authentication and host-key trust |
| `Missing sudo password` | Noninteractive escalation absent | Configure authorized become credentials |
| Apache not active | Installation/service failure | Inspect `systemctl status` and journal |

## 10. Security notes

- Do not publish the lab inventory if it contains passwords.
- Do not publish SSH private keys, tokens, or sudo passwords.
- Prefer SSH key authentication and secure secrets handling in production.
- `0655` is required for this lab, not a general recommendation for web content.

## 11. Interview questions

**Q1. Why use an Ansible role rather than putting everything in one playbook?**  
A: Roles provide modularity, reusable components and standard resource lookup paths.

**Q2. What is the difference between `inventory_hostname` and `ansible_hostname`?**  
A: `inventory_hostname` is the inventory alias; `ansible_hostname` is a fact gathered from the remote OS.

**Q3. Where does the `template` module look for role templates?**  
A: In the role's `templates/` directory, among Ansible's role-aware search paths.

**Q4. What does `become: true` change?**  
A: It elevates remote task privileges, commonly using sudo; explicit file ownership still applies.

**Q5. Why is clean-state validation valuable?**  
A: It detects hidden manual prerequisites and confirms the automation actually provisions missing resources.

**Q6. What is the difference between service `started` and `enabled`?**  
A: `started` controls current state, while `enabled` configures startup at boot.

**Q7. Is an HTML permission mode of `0655` typical?**  
A: No; its execute bits are unusual, but the exact challenge requires it.

---

**Outcome:** KodeKloud Day 92 passed. The snippets are a documented reference configuration, not a claim of an independently captured terminal transcript.
