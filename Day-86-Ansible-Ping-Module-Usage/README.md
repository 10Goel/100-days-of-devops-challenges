# Day 86 — Passwordless SSH for Ansible Managed Nodes

![Status](https://img.shields.io/badge/KodeKloud-Day_86_Passed-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-SSH_Authentication-blue)

## Challenge overview
The Nautilus DevOps team needed the **jump host** (`thor`) to act as an Ansible controller and connect to **App Server 3** (`stapp03`, user `banner`) without an interactive SSH password. The supplied inventory at `/home/thor/ansible/inventory` had to support a successful Ansible `ping` to App Server 3.

**Outcome:** KodeKloud Day 86 **PASSED**. We verified public-key SSH authentication and corrected an inventory configuration that initially caused Ansible to attempt password authentication.

## Requirements
| Field | Value |
|---|---|
| Controller | Jump host, account `thor` |
| Managed node | App Server 3 (`stapp03`) |
| Remote account | `banner` |
| Inventory | `/home/thor/ansible/inventory` |
| Authentication | SSH public key (noninteractive) |
| Verification | `ansible -i inventory stapp03 -m ping` |

## Architecture
```text
Jump host (thor)                            App Server 3 (banner@stapp03)
~/.ssh/id_ed25519       SSH handshake        ~/.ssh/authorized_keys
PRIVATE KEY        --------------------->   PUBLIC KEY entry
~/.ssh/known_hosts   verifies host key      SSH server host key
       |
       +-- Ansible reads inventory --> stapp03 | SUCCESS => ping: pong
```

> The private key must stay on the controller. `authorized_keys` controls who may authenticate to a remote account; `known_hosts` protects the client against impersonated servers.

## Implementation summary
1. Inspected existing inventory and SSH key material without overwriting valid keys.
2. Prepared Ed25519 keys for `thor` and installed the public key in `banner`'s `authorized_keys` using `ssh-copy-id`.
3. Verified passwordless login using `ssh -o BatchMode=yes` and strict host-key checking.
4. Fixed the existing inventory entry so `stapp03` uses `ansible_user=banner` and the controller's private key rather than an inherited or explicit SSH password.
5. Confirmed inventory parsing and Ansible module execution with `ansible -i inventory stapp03 -m ping`.
6. KodeKloud validator confirmed the task passed.

## Final inventory configuration
Retain unrelated inventory hosts and group settings; replace or update only the App Server 3 connection settings as appropriate:

```ini
stapp03 ansible_host=stapp03 ansible_user=banner ansible_connection=ssh ansible_ssh_private_key_file=/home/thor/.ssh/id_ed25519
```

Remove `ansible_password`, `ansible_ssh_pass`, and equivalent inherited SSH password variables **for `stapp03`**. The existing inventory may place the host in a group; preserve the grouping that the lab provides.

## Verification
```bash
cd /home/thor/ansible
ssh -i ~/.ssh/id_ed25519 -o BatchMode=yes -o StrictHostKeyChecking=yes banner@stapp03 'whoami && hostname'
ansible-inventory -i inventory --graph
ansible -i inventory stapp03 -m ping
```

Expected successful Ansible response:
```text
stapp03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Troubleshooting incident
During the challenge, **SSH key authentication worked**, returning `banner` and `stapp03`, but Ansible initially reported:

```text
stapp03 | UNREACHABLE! => {
  "msg": "Invalid/incorrect password: Permission denied, please try again."
}
```

The mismatch indicated that Ansible's effective connection variables still directed it toward password authentication. We corrected the inventory to select the SSH key and removed stale password variables. See [Notes.md](Notes.md) for troubleshooting details.

## Repository files
- [README.md](README.md) — task, architecture and successful solution
- [Commands.md](Commands.md) — reproducible command runbook
- [Notes.md](Notes.md) — SSH, Ansible internals, security and interview notes

**Security:** Do not commit private keys, actual passwords, or sensitive host inventories to a public repository. The inventory shown here contains no secrets.
# Day 86 — Passwordless SSH for Ansible Managed Nodes

![Status](https://img.shields.io/badge/KodeKloud-Day_86_Passed-brightgreen) ![Topic](https://img.shields.io/badge/Ansible-SSH_Authentication-blue)

## Challenge overview
The Nautilus DevOps team needed the **jump host** (`thor`) to act as an Ansible controller and connect to **App Server 3** (`stapp03`, user `banner`) without an interactive SSH password. The supplied inventory at `/home/thor/ansible/inventory` had to support a successful Ansible `ping` to App Server 3.

**Outcome:** KodeKloud Day 86 **PASSED**. We verified public-key SSH authentication and corrected an inventory configuration that initially caused Ansible to attempt password authentication.

## Requirements
| Field | Value |
|---|---|
| Controller | Jump host, account `thor` |
| Managed node | App Server 3 (`stapp03`) |
| Remote account | `banner` |
| Inventory | `/home/thor/ansible/inventory` |
| Authentication | SSH public key (noninteractive) |
| Verification | `ansible -i inventory stapp03 -m ping` |

## Architecture
```text
Jump host (thor)                            App Server 3 (banner@stapp03)
~/.ssh/id_ed25519       SSH handshake        ~/.ssh/authorized_keys
PRIVATE KEY        --------------------->   PUBLIC KEY entry
~/.ssh/known_hosts   verifies host key      SSH server host key
       |
       +-- Ansible reads inventory --> stapp03 | SUCCESS => ping: pong
```

> The private key must stay on the controller. `authorized_keys` controls who may authenticate to a remote account; `known_hosts` protects the client against impersonated servers.

## Implementation summary
1. Inspected existing inventory and SSH key material without overwriting valid keys.
2. Prepared Ed25519 keys for `thor` and installed the public key in `banner`'s `authorized_keys` using `ssh-copy-id`.
3. Verified passwordless login using `ssh -o BatchMode=yes` and strict host-key checking.
4. Fixed the existing inventory entry so `stapp03` uses `ansible_user=banner` and the controller's private key rather than an inherited or explicit SSH password.
5. Confirmed inventory parsing and Ansible module execution with `ansible -i inventory stapp03 -m ping`.
6. KodeKloud validator confirmed the task passed.

## Final inventory configuration
Retain unrelated inventory hosts and group settings; replace or update only the App Server 3 connection settings as appropriate:

```ini
stapp03 ansible_host=stapp03 ansible_user=banner ansible_connection=ssh ansible_ssh_private_key_file=/home/thor/.ssh/id_ed25519
```

Remove `ansible_password`, `ansible_ssh_pass`, and equivalent inherited SSH password variables **for `stapp03`**. The existing inventory may place the host in a group; preserve the grouping that the lab provides.

## Verification
```bash
cd /home/thor/ansible
ssh -i ~/.ssh/id_ed25519 -o BatchMode=yes -o StrictHostKeyChecking=yes banner@stapp03 'whoami && hostname'
ansible-inventory -i inventory --graph
ansible -i inventory stapp03 -m ping
```

Expected successful Ansible response:
```text
stapp03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Troubleshooting incident
During the challenge, **SSH key authentication worked**, returning `banner` and `stapp03`, but Ansible initially reported:

```text
stapp03 | UNREACHABLE! => {
  "msg": "Invalid/incorrect password: Permission denied, please try again."
}
```

The mismatch indicated that Ansible's effective connection variables still directed it toward password authentication. We corrected the inventory to select the SSH key and removed stale password variables. See [Notes.md](Notes.md) for troubleshooting details.

**Security:** Do not commit private keys, actual passwords, or sensitive host inventories to a public repository. The inventory shown here contains no secrets.
