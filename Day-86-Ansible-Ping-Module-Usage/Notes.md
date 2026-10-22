# Day 86 — Notes: Passwordless SSH and Ansible Authentication

## 1. Control nodes and managed nodes
The **controller** (`thor` on the jump host) runs Ansible. The **managed node** (`banner@stapp03`) receives SSH connections and executes Ansible modules. Ansible normally does not require a persistent agent installed on the target.

## 2. SSH authentication flow
1. The SSH client authenticates the server's host key using trusted `known_hosts` entries.
2. The client proves possession of the controller's **private** SSH key.
3. The server compares the matching **public** key against the remote account's `~/.ssh/authorized_keys`.
4. If authorized, the login succeeds without an account-password prompt.

```text
thor@jump-host                             banner@stapp03
id_ed25519 (secret)                        .ssh/authorized_keys (public)
known_hosts (trusted server identity)      sshd presents server host key
```

`authorized_keys` is **server-side user authorization**; `known_hosts` is **client-side server identity verification**. These files cannot substitute for one another.

## 3. Essential tools
| Command / option | Purpose |
|---|---|
| `ssh-keygen -t ed25519` | Generate Ed25519 identity pair |
| `ssh-copy-id -i ...pub` | Add the public key to the remote user's authorized list |
| `ssh -i ...` | Select a specific private identity |
| `BatchMode=yes` | Suppress interactive password/passphrase prompts |
| `StrictHostKeyChecking=yes` | Require trusted host keys |
| `ssh-keygen -F host` | Find host entries in known_hosts |
| `ssh -v` | View authentication decisions |

A no-passphrase private key can be used in a short-lived isolated lab. In production, consider hardware-backed or passphrase-protected keys with an agent, limited permissions, rotation and monitoring.

## 4. Inventory connection variables
```ini
stapp03 ansible_host=stapp03 ansible_user=banner ansible_connection=ssh ansible_ssh_private_key_file=/home/thor/.ssh/id_ed25519
```

- `ansible_host`: actual DNS name or IP address.
- `ansible_user`: login identity (`banner`).
- `ansible_connection=ssh`: use the SSH connection plugin.
- `ansible_ssh_private_key_file`: controller-side key location.
- `ansible_password` / `ansible_ssh_pass`: request password authentication and should not remain set for this host when the goal is key-based login.

**Variable precedence matters:** A value in host/group inventory, vars files, playbooks, or extra vars may override another. Inspect the *effective* host variables with `ansible-inventory -i inventory --host stapp03`, taking care not to leak secrets.

## 5. Why manual SSH worked while Ansible ping failed
Observed sequence:
1. `ssh-copy-id` added one public key for `banner@stapp03`.
2. `ssh -i ~/.ssh/id_ed25519 -o BatchMode=yes -o StrictHostKeyChecking=yes ...` returned `banner` and `stapp03` without prompting.
3. Ansible `ping` reported **Invalid/incorrect password** rather than an SSH-key error.
4. The inventory was edited to remove obsolete password variables and select the private key.
5. Task subsequently passed.

The evidence points to an **Ansible effective authentication configuration mismatch**, not a failed public-key installation.

## 6. Ansible ping is not ICMP ping
`ansible -i inventory stapp03 -m ping` connects via the configured transport (typically SSH), runs a small remote module and expects `pong`. This confirms remote execution and generally a usable Python interpreter; it does not send ICMP echo packets like the shell `ping` command.

## 7. Security precautions
- Keep `~/.ssh/id_ed25519` private (usually mode `0600`).
- Keep `~/.ssh` accessible only to its owner (usually `0700`).
- Do not commit keys, lab passwords, or populated inventories to GitHub.
- Verify host keys against trusted fingerprints; do not disable verification for convenience.
- Use least-privilege remote accounts and prefer key rotation for long-lived automation.
- `become` and `ansible_become_password` handle privilege escalation; they are separate from SSH login authentication.

## 8. Troubleshooting matrix
| Symptom | Interpretation | Response |
|---|---|---|
| `Permission denied (publickey)` | Wrong/missing authorized public key or identity | Check `ssh-copy-id`, remote permissions and `ssh -i -v` |
| Ansible `Invalid/incorrect password` despite working SSH key | Ansible still attempts password auth | Inspect effective inventory variables; remove stale passwords |
| `Host key verification failed` | Server host identity not trusted/matches wrong key | Verify fingerprint before updating `known_hosts` |
| `UNREACHABLE` | Connection could not be established | Validate DNS, SSH, username and key path |
| `Failed to connect ... Python` | Authentication works but module cannot execute | Verify Python on managed node |

## 9. Quick interview questions
**Q: Is passwordless SSH the same as disabling SSH authentication?**  
No. SSH still authenticates the client using proof of a private key.

**Q: Can the public key be shared?**  
Yes, it is intended for the target's `authorized_keys`; the private key must be protected.

**Q: Why does `BatchMode=yes` matter for automation?**  
It prevents jobs from hanging while waiting for interactive credentials.

**Q: What is the difference between `known_hosts` and `authorized_keys`?**  
The former verifies remote hosts to the client; the latter grants client identities access to a remote account.

**Q: Why might Ansible fail when plain SSH works?**  
Ansible can select different credentials or connection variables, such as a password in inventory that overrides the key-based path.

**Q: Does the Ansible ping module check ICMP reachability?**  
No, it validates Ansible module execution on the remote host.

## 10. Learning milestone
**Day 86 PASSED:** established key-based SSH from the Ansible controller to App Server 3 and corrected inventory authentication variables so Ansible can execute modules without extra connection arguments or interactive passwords.
