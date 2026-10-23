# Day 87 — Ansible Package Management: Technical Notes

## 1. Concept and execution model

Ansible is an agentless automation system: `thor` executes a playbook from the jump host, connects to the managed nodes via SSH, and uses appropriate Python-based modules to enforce a desired state.

```text
Inventory group `app_servers`
           |
           v
Play: hosts=app_servers, become=true
           |
           v
Task: ansible.builtin.yum(name=samba, state=present)
           |
           v
stapp01 + stapp02 + stapp03
```

## 2. Inventory groups and variables

`[app_servers]` names an inventory group, which the playbook references as `hosts: app_servers`. `stapp01`, `stapp02`, and `stapp03` are host identifiers; `ansible_host` can supply an alternate address. Different nodes may need distinct `ansible_user` and `ansible_password` settings.

Sensitive values should normally be vaulted or obtained from an approved credential store. `chmod 600 inventory` restricts casual local access but does **not** encrypt secrets.

## 3. SSH authentication versus privilege escalation

- `ansible_user` specifies who logs in by SSH.
- `ansible_password` is a password for SSH authentication when keys are not used.
- `become: true` enables privilege escalation, normally via sudo.
- `ansible_become_password` supplies a sudo password where needed.

Successful Ansible `ping` does **not** imply sudo will succeed. Verify root execution with:

```bash
ansible -i inventory app_servers -b -m command -a 'id -u'
```

## 4. Why use `ansible.builtin.yum`?

The challenge explicitly requests the yum module. It operates against RPM-based package managers and modern implementations may delegate to DNF depending on the target's package-manager facts and Ansible version. Use the exact requested module in this lab rather than substituting `shell: yum install ...`.

```yaml
ansible.builtin.yum:
  name: samba
  state: present
```

`state: present` ensures the package is installed, rather than forcing the newest available version or starting Samba services. The exercise does not request configuring `/etc/samba/smb.conf`, enabling `smb`, or opening firewall ports.

## 5. Idempotency and change reporting

When Samba is absent, the task installs it and usually reports `changed`. When installed and compliant, a subsequent run normally reports `ok` and `changed=0`. This is desired-state management, unlike manually repeating `yum install` commands without recorded automation.

## 6. How to read a play recap

```text
stapp01 : ok=2 changed=1 unreachable=0 failed=0
```

- `ok`: successful tasks, including tasks that changed something.
- `changed`: tasks that changed system state.
- `unreachable`: failed contact with the host.
- `failed`: task errors.

Counts shown are illustrative. A completed challenge needs each host reachable and the package installed; exact counts depend on initial state.

## 7. Common troubleshooting cases

| Symptom | Likely explanation | Investigation |
|---|---|---|
| Host unreachable | DNS/network/user/password problem | SSH manually and check inventory |
| `sshpass` host-key error | No trusted host key | Verify fingerprint and add to `known_hosts` securely |
| `Missing sudo password` | No become password configured | Provide authorized escalation credentials |
| `Permission denied` while installing | User lacks sudo rights | Validate `sudo` rights and become settings |
| `No package samba available` | Repository metadata/configuration issue | Inspect enabled RPM repositories |
| Inventory parsing failure | Incorrect INI syntax | `ansible-inventory --graph` |
| Python/module error | Incompatible interpreter or package bindings | Review module error and remote interpreter |

## 8. Related modules

| Module | Typical platform | Purpose |
|---|---|---|
| `ansible.builtin.yum` | RPM-based systems | Install/update/remove RPM packages |
| `ansible.builtin.dnf` | Modern DNF systems | Package management via DNF |
| `ansible.builtin.apt` | Debian/Ubuntu | APT package management |
| `ansible.builtin.package` | Multiple platforms | Generic package interface |
| `ansible.builtin.service` | Multiple platforms | Manage services; **not required in Day 87** |

## 9. Interview questions

**Why not use `shell: sudo yum install -y samba`?**  
The package module expresses desired state, improves checkability and repeatability, and avoids ad-hoc command handling.

**Why `become: true`?**  
Installing system packages requires privileges generally unavailable to an unprivileged SSH user.

**Does `changed=0` mean failure?**  
No. It often means the desired package was already installed.

**What does Ansible's `ping` test?**  
Remote module execution and Python availability, not an ICMP echo.

**What proves the task was executed correctly?**  
A play recap with no failed/unreachable hosts and successful `rpm -q samba` on each target.

## 10. Security and production improvements

Prefer SSH keys with verified host identity, Ansible Vault or centrally managed credentials, dedicated automation identities, restricted sudo rules, and repository review of inventory changes. Avoid keeping real SSH or sudo passwords in GitHub.

---
**Day 87 outcome:** PASSED (user confirmed). The documentation describes the successful procedure without inventing command logs.
