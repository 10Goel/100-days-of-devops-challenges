# Day 82 — Ansible Inventory Setup for App Server 1

![Ansible](https://img.shields.io/badge/Ansible-Inventory%20%26%20Playbooks-EE0000?logo=ansible&logoColor=white) ![Linux](https://img.shields.io/badge/Platform-Linux-333333?logo=linux&logoColor=white) ![Result](https://img.shields.io/badge/KodeKloud-PASSED-brightgreen)

## Overview

The Nautilus DevOps team needed an Ansible inventory on the **jump host** to run an existing playbook against **App Server 1**. The playbook installs Apache HTTP Server (`httpd`) and starts its service. The inventory needed to supply the host and connection details so the provided validation command could run **without additional CLI arguments**.

**Result:** Day 82 passed. The playbook completed on `stapp01` with `ok=3`, `changed=2`, `unreachable=0`, and `failed=0`.

## Requirements

| Item | Requirement |
|---|---|
| Control node | Jump host (`thor`) |
| Inventory path | `/home/thor/playbook/inventory` |
| Inventory type | INI |
| Managed host | App Server 1, `stapp01` |
| Remote login | `tony` (lab account) |
| Existing playbook | `/home/thor/playbook/playbook.yml` |
| Required execution | `ansible-playbook -i inventory playbook.yml` |
| Playbook effect | Install `httpd`; ensure its service is started |

## Architecture

```text
+-------------------------------------+
| Jump host: Ansible control node     |
| User: thor                          |
| /home/thor/playbook/                |
|   |-- inventory                     |
|   `-- playbook.yml                  |
+-------------------+-----------------+
                    | SSH (inventory credentials)
                    v
+-------------------------------------+
| App Server 1: stapp01               |
| Login: tony                         |
| Ansible runs httpd tasks            |
+-------------------------------------+
```

## Implementation

1. Inspected the existing `playbook.yml`, especially its `hosts:` selector and privilege requirements.
2. Created an INI inventory at the exact required path, including `stapp01` and the connection variables needed for non-interactive access.
3. Restricted the inventory file to the owning user (`chmod 600`) because lab credentials were stored there.
4. Confirmed inventory parsing with `ansible-inventory --graph` and `--list`.
5. Verified connectivity using `ansible -i inventory all -m ping`.
6. Verified playbook syntax, then executed **the exact prescribed command**.
7. Confirmed the Ansible play recap and submitted the challenge successfully.

### Inventory template (redacted)

> **Security:** This is a *template*, not a copy of the secret-bearing lab inventory. Replace placeholders only in your private lab; never commit real credentials.

```ini
[app_servers]
stapp01 ansible_connection=ssh ansible_user=tony ansible_password="<SSH_PASSWORD>" ansible_become_password="<SUDO_PASSWORD>"
```

If the playbook targets a specific inventory group, the section name must match that `hosts:` value; a playbook targeting `all` can use the example group above. The actual credential needs depend on SSH and sudo policy.

## Verification evidence

The recorded playbook execution included:

```text
PLAY [all]
TASK [Gathering Facts]             ok: [stapp01]
TASK [Install httpd package]       changed: [stapp01]
TASK [Start service httpd]         changed: [stapp01]

PLAY RECAP
stapp01 : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

The terminal also showed the successful Ansible ping response (`"ping": "pong"`) and a passing syntax check. **KodeKloud marked Day 82 as passed.**

## Why this matters

Ansible inventories separate **which machines to manage** from **what actions to perform**. Connection and privilege variables make automation reproducible and allow the playbook to run through the same command locally and in validation or CI workflows.

## Security considerations

- Never commit passwords, private SSH keys, or credential-bearing inventory files.
- Prefer SSH keys or Ansible Vault for persisted automation.
- Protect sensitive lab inventories with restrictive file permissions.
- Do not publish the raw output of `ansible-inventory --list` if it contains credentials.

---

**Challenge:** KodeKloud 100 Days of DevOps — Day 82  
**Outcome:** PASSED
