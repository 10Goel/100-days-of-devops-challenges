# Day 89 — Apache Installation and Service Management with Ansible

![Status](https://img.shields.io/badge/KodeKloud-PASSED-2ea44f) ![Automation](https://img.shields.io/badge/Tool-Ansible-1f6feb) ![Platform](https://img.shields.io/badge/OS-RHEL%20family-6f42c1)

## Challenge overview

The Nautilus DevOps team needed an **Ansible playbook** to install Apache HTTP Server (`httpd`) on every application server in Stratos DC, **start the service**, and **enable it at boot**. The controller is the jump host and the execution account is `thor`. **Day 89 was passed in KodeKloud.**

## Requirements

| Item | Value |
|---|---|
| Controller | Jump host (`thor`) |
| Existing inventory | `/home/thor/ansible/inventory` |
| Required playbook | `/home/thor/ansible/playbook.yml` |
| Target systems | All application servers (`stapp01`, `stapp02`, `stapp03`) |
| Package | `httpd` |
| Desired service state | Running **and** enabled at startup |
| Exact validator command | `ansible-playbook -i inventory playbook.yml` |

## Architecture

```text
                      Jump host (thor)
                  /home/thor/ansible/
                    inventory + playbook.yml
                              |
                       Ansible over SSH
                         (sudo/become)
                 _____________|_____________
                |             |             |
            stapp01        stapp02        stapp03
             tony           steve          banner
                |             |             |
                +---- httpd installed ------+
                +---- httpd started --------+
                +---- httpd enabled --------+
```

## Working playbook

Create `/home/thor/ansible/playbook.yml`:

```yaml
---
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

### Why this works

1. `hosts: all` matches every host in the existing inventory; make sure the inventory contains **only the intended managed servers**.
2. `become: true` enables elevated privileges for package and service operations.
3. `yum` with `state: present` installs `httpd` if absent without demanding an upgrade on each run.
4. `service` with `state: started` ensures Apache is currently running.
5. `enabled: true` configures automatic startup after boot.

## Execution

From the jump host:

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The validation command runs **without additional flags**. Inventory/SSH/sudo authentication therefore must already be configured for the `thor` user.

## Verification

```bash
ansible -i inventory all -m command -a 'rpm -q httpd'
ansible -i inventory all -b -m command -a 'systemctl is-active httpd'
ansible -i inventory all -b -m command -a 'systemctl is-enabled httpd'
```

Expected outcomes for **each** of `stapp01`, `stapp02`, `stapp03`:

- `rpm -q httpd` prints an installed package version.
- `systemctl is-active httpd` prints `active`.
- `systemctl is-enabled httpd` prints `enabled`.
- The play recap reports `unreachable=0` and `failed=0`.

The first run may report `changed` for installation and service startup. When the desired state is already met, a subsequent run should normally report `changed=0`.

## Troubleshooting highlights

| Symptom | Likely cause | Diagnostic action |
|---|---|---|
| `UNREACHABLE` | SSH credentials, routing, host-key trust | Verify direct SSH and inventory settings |
| `Missing sudo password` | Become credentials missing | Configure authorized escalation in inventory or secret store |
| `No package httpd available` | Package repositories unavailable | Inspect enabled repositories and connectivity |
| Service does not start | Bad config, port conflict, or package problem | `systemctl status httpd` / `journalctl -u httpd` |
| Play succeeds on only some hosts | Incomplete inventory | `ansible-inventory -i inventory --graph` |

For commands and deeper discussion, see [Commands.md](Commands.md) and [Notes.md](Notes.md).

## Key takeaways

- Package presence and service state are separate desired-state controls.
- **Started** means running now; **enabled** means started automatically at boot.
- Ansible can apply both settings consistently across multiple servers.
- No lab passwords, private SSH keys, or sensitive inventory values belong in a public repository.

**Outcome:** KodeKloud Day 89 successfully completed.
