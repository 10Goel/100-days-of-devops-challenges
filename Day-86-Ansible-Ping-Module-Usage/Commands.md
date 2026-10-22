# Day 86 — Commands Runbook

> Run on the **jump host as `thor`**, except for the remote commands executed by SSH. Use the current lab's authorized credentials when prompted.

## 1. Inspect supplied inventory and SSH material
```bash
whoami
cd /home/thor/ansible
pwd
ls -la
# Caution: avoid sharing an inventory with passwords
cat inventory
ansible-inventory -i inventory --graph
ls -la ~/.ssh
```

## 2. Create a key pair only if one does not exist
```bash
# Inspect before creating; do not overwrite existing keys
ls -l ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
mkdir -p ~/.ssh
chmod 700 ~/.ssh
# Only if the key pair is absent:
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N '' -C 'thor@jump-host'
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

**Lab-specific note:** The observed successful session used `~/.ssh/id_ed25519`. A blank key passphrase supports noninteractive lab automation but is not the preferred production protection for unattended keys.

## 3. Install public key on App Server 3
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub banner@stapp03
```

Verify the SSH host fingerprint against trusted infrastructure information before accepting it. Authenticate with `banner`'s current **Linux SSH password** when prompted. Only the **public** key is installed remotely.

## 4. Prove passwordless login
```bash
ssh -i ~/.ssh/id_ed25519 \
  -o BatchMode=yes \
  -o StrictHostKeyChecking=yes \
  banner@stapp03 'whoami && hostname'
```
Expected remote output:
```text
banner
stapp03
```

## 5. Adjust the existing inventory
```bash
vi /home/thor/ansible/inventory
```
Example `stapp03` entry:
```ini
stapp03 ansible_host=stapp03 ansible_user=banner ansible_connection=ssh ansible_ssh_private_key_file=/home/thor/.ssh/id_ed25519
```

Remove SSH password variables from this host and any inherited group variables where they affect `stapp03`, while preserving other hosts. Avoid putting private key contents in the inventory.

## 6. Validate effective settings
```bash
ansible-inventory -i inventory --graph
ansible-inventory -i inventory --host stapp03
```

The effective settings should show `ansible_user: banner` and `ansible_ssh_private_key_file: /home/thor/.ssh/id_ed25519`, with no `ansible_password`/`ansible_ssh_pass`. **Do not paste unredacted inventory output publicly**, as it can contain secrets.

## 7. Run the required connectivity check
```bash
ansible -i inventory stapp03 -m ping
```
Expected:
```text
stapp03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 8. Troubleshooting (only when needed)
```bash
# Confirm trusted server key
ssh-keygen -F stapp03

# Debug SSH identity selection without password prompts
ssh -v -i ~/.ssh/id_ed25519 -o BatchMode=yes banner@stapp03 'hostname'

# Display SSH invocation and module errors; redact credentials from shared logs
ansible -i inventory stapp03 -m ping -vvv
```

Typical fixes:
- `Permission denied (publickey)`: verify the correct public key is installed for `banner`, remote `~/.ssh` permissions and the client key path.
- `Invalid/incorrect password`: check effective Ansible variables for obsolete `ansible_password`.
- `Host key verification failed`: independently verify the fingerprint and add the correct key to `known_hosts`; do not disable checking.
- `UNREACHABLE`: inspect DNS/network, SSH, remote username and authentication.

## 9. Completion
When `ansible -i inventory stapp03 -m ping` succeeds noninteractively, submit the KodeKloud **Check** action. Day 86 was confirmed **PASSED**.
