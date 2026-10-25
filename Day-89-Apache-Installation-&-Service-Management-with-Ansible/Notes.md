# Day 89 — Notes.md

## Learning objectives

Understand the distinction between **package management**, **service management**, **SSH authentication**, **privilege escalation**, and **idempotency** in Ansible. This challenge requires installing Apache and keeping the service both running and boot-enabled on three servers.

## 1. Ansible control and managed nodes

- **Control node:** jump host, operating as `thor`.
- **Inventory:** `/home/thor/ansible/inventory`, identifying managed servers and connection details.
- **Managed nodes:** `stapp01`, `stapp02`, `stapp03`.
- **Playbook:** `/home/thor/ansible/playbook.yml`, declaratively specifying desired state.

`hosts: all` selects all inventory hosts. In environments with extra hosts, use a dedicated group to avoid accidental changes. The lab specifies that the inventory contains all intended application servers.

## 2. Anatomy of a playbook

```yaml
- name: Install and configure Apache on all application servers
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Install Apache HTTP Server
      ansible.builtin.yum:
        name: httpd
        state: present
    - name: Start and enable Apache HTTP Server
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

- **Play:** selects hosts and common execution settings.
- **Task:** one action expressed as a module invocation.
- **Module:** reusable logic for managing a remote resource.
- **`become: true`:** elevates remote task privileges, usually using sudo.
- **`gather_facts: true`:** collects remote system metadata before tasks.

## 3. The Yum module

`ansible.builtin.yum` handles RPM-family package installation on supported target systems. Modern systems may route underlying package operations through DNF, depending on the Ansible version and OS.

- `name: httpd`: the exact required package.
- `state: present`: install if missing; do not force an upgrade to the newest version.
- `state: latest`: would request upgrades and is **not necessary** for Day 89.
- `state: absent`: removes a package; not appropriate here.

Ansible also offers `ansible.builtin.dnf`, `ansible.builtin.apt` and generic `ansible.builtin.package` for differing platforms. The exercise uses the RPM-family Apache package name `httpd`.

## 4. The service module

`ansible.builtin.service` is a service-manager abstraction that can control units on supported init systems, including systemd-backed machines.

```yaml
ansible.builtin.service:
  name: httpd
  state: started
  enabled: true
```

**Started ≠ enabled:**

| Setting | Meaning |
|---|---|
| `state: started` | Ensure the service is running now |
| `enabled: true` | Ensure the service is configured to start at system boot |
| `state: restarted` | Force a restart each time the task runs |
| `state: stopped` | Ensure the service is not running |
| `enabled: false` | Disable automatic startup |

For verification, `systemctl is-active httpd` should return `active`, and `systemctl is-enabled httpd` should return `enabled`.

## 5. Why task order matters

Installing `httpd` comes before starting its service. Reversing the order can fail because the service unit may not exist before installation.

Typical sequence:

```text
Inventory → SSH login → sudo → install httpd → start httpd → enable httpd → verify
```

## 6. SSH authentication versus sudo

`ansible_user` determines the remote login account; `become: true` determines whether remote tasks run with elevated privileges. A working SSH connection does not by itself grant root privileges.

The validator runs without extra `-k` or `-K` flags, so existing inventory or authorized SSH and privilege-escalation setup must provide noninteractive access. Never commit passwords, API tokens, or private keys. Prefer key-based login and secret managers / Ansible Vault for reusable real-world workflows.

**Host keys:** `known_hosts` identifies trusted remote servers. Accept SSH fingerprints only after checking trusted information; don't disable host verification as a workaround.

## 7. Idempotency

Ansible modules generally converge hosts to a desired state. After `httpd` has been installed, is running, and is enabled, rerunning the playbook normally produces no changes.

A first-run recap might show `changed=2`; a repeated run may show `changed=0`. Neither number alone proves success: check `failed=0`, `unreachable=0` and the actual package/service states.

## 8. Debugging scenarios

| Symptom | Diagnosis | Action |
|---|---|---|
| Ansible `UNREACHABLE` | SSH or inventory mismatch | Inspect hostname, user, key/password, routing, host trust |
| `Missing sudo password` | Privilege-escalation config incomplete | Provide authorized sudo credentials securely |
| `No package httpd available` | Repositories or metadata unavailable | Check DNF/YUM repository setup and network access |
| `Could not find service httpd` | Installation may not have completed | Verify package installation first |
| Apache fails to start | Port conflict or invalid config | Inspect unit status and journal logs |
| Some hosts missing from recap | Inventory or host-pattern problem | Inspect inventory graph |
| `changed` every run | Environment drift or forced action | Inspect task details and avoid unnecessary restarts |

### Useful diagnostic commands

```bash
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
ansible -i inventory all -m command -a 'rpm -q httpd'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
ansible -i inventory all -b -m command -a 'journalctl -u httpd -n 30 --no-pager'
```

## 9. Interview review

**Q1. Why specify `become: true`?**  
Package installation and service control usually need root privileges; SSH credentials alone do not provide them.

**Q2. `state: present` vs `state: latest`?**  
`present` guarantees installation; `latest` also seeks upgrades.

**Q3. What if the package is installed but the service is stopped?**  
The package task reports no change; the service task starts it.

**Q4. What if the service is running but disabled?**  
The service task enables it for future boots without requiring a restart just for enablement.

**Q5. Can Ansible manage multiple servers with one play?**  
Yes: inventory patterns such as `all` or a group name determine the managed fleet.

**Q6. How do you confirm task success beyond the recap?**  
Check installed package version, active state, and enabled state on every target.

**Q7. Why is the playbook repeatable?**  
It declares desired state through idempotent modules instead of issuing unconditional installation and restart commands.

## 10. Production extensions

Consider group-specific inventories, Ansible Vault/secret management, smoke checks (`uri` module), rollback/change controls, and CI/CD pipeline execution. Avoid changing Apache's configuration or firewall when the specification only asks to install and enable the service.

---

**Day 89 result: PASSED.** Expected sample command outputs are instructional examples, not purported verbatim logs from the lab.
