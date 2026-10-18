# Day 82 — Commands and Execution Runbook

> Commands below were used for the Day 82 inventory workflow or are explicitly identified as optional checks. Credential-bearing values are redacted. The supplied `playbook.yml` was not rewritten.

## 1. Inspect working directory and playbook

Run on the jump host as `thor`:

```bash
cd /home/thor/playbook
pwd
ls -la
cat playbook.yml
```

Check the playbook's `hosts:` field, `become` setting, and declared tasks before creating an inventory group.

## 2. Create the inventory

```bash
vi /home/thor/playbook/inventory
```

INI template (replace placeholders **locally**, never in GitHub):

```ini
[app_servers]
stapp01 ansible_connection=ssh ansible_user=tony ansible_password="<SSH_PASSWORD>" ansible_become_password="<SUDO_PASSWORD>"
```

- The host must appear as `stapp01`, matching the Nautilus server name.
- When the playbook uses `hosts: all`, the above group works.
- When it uses another group, replace `[app_servers]` with that matching group.
- `ansible_become_password` is relevant when tasks require sudo and the target account needs a password. It is not necessarily the same as the SSH password.

Secure and inspect the file:

```bash
chmod 600 /home/thor/playbook/inventory
ls -l /home/thor/playbook/inventory
```

## 3. Confirm Ansible inventory recognition

```bash
cd /home/thor/playbook
ansible-inventory -i inventory --graph
```

Expected structural example:

```text
@all:
  |--@ungrouped:
  |--@app_servers:
  |  |--stapp01
```

For detailed local inspection only (may reveal sensitive connection variables):

```bash
ansible-inventory -i inventory --list
```

Do **not** publish unredacted output.

## 4. Check managed-node connectivity

```bash
ansible -i inventory all -m ping
```

Observed result included:

```json
{
  "changed": false,
  "ping": "pong"
}
```

If host-key trust is not yet established, connect and **verify the server fingerprint** before accepting it:

```bash
ssh tony@stapp01
# After verifying and accepting the correct host key, authenticate and then:
exit
```

The initial lab attempt to authenticate manually included one rejected password attempt, followed by a successful login. The eventual Ansible connection and playbook run were successful.

Check whether `sshpass` is available if password-based Ansible SSH reports it missing:

```bash
command -v sshpass
```

Observed in the lab: `/usr/bin/sshpass`.

## 5. Syntax validation

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
```

Observed output:

```text
playbook: playbook.yml
```

A syntax check does not establish that the playbook tasks will succeed remotely.

## 6. Execute the exact task validation command

```bash
ansible-playbook -i inventory playbook.yml
```

**Observed play recap:**

```text
PLAY [all]
TASK [Gathering Facts]             ok: [stapp01]
TASK [Install httpd package]       changed: [stapp01]
TASK [Start service httpd]         changed: [stapp01]

PLAY RECAP
stapp01 : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

The recap established that the managed node was reachable and the playbook completed successfully without extra flags. KodeKloud subsequently reported **PASSED**.

## 7. Optional follow-up checks

These are useful for learning, but they were **not required evidence** for the original task:

```bash
# Gather facts from the inventory target
ansible -i inventory stapp01 -m setup -a 'filter=ansible_distribution*'

# Check service status on the remote target (sudo/become may be needed)
ansible -i inventory stapp01 -b -m command -a 'systemctl is-active httpd'
```

A repeat playbook run can help examine idempotency; `changed=0` is common for already-converged `package` and `service` tasks, but do not claim that result without running it.

## 8. GitHub-safe publishing

Create a documentation-only repository folder:

```text
Day-82-Ansible-Inventory-Setup/
├── README.md
├── Commands.md
└── Notes.md
```

Never include the real `inventory` file if it embeds credentials. A redacted inventory template is shown above for reproducibility.
