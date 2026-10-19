# Day 83 — Commands and Verification Runbook

> Commands execute on the **jump host** unless noted. Use only the current KodeKloud lab's verified credentials. Do not place real passwords in Git.

## 1. Inspect the provided workspace

```bash
cd /home/thor/ansible
pwd
ls -la
cat inventory                     # CAUTION: may display passwords; don't share output
```

Inspect first: the lab supplies an existing inventory that must be adjusted, not destroyed.

## 2. Update inventory for App Server 3

```bash
vi /home/thor/ansible/inventory
chmod 600 /home/thor/ansible/inventory
ls -l /home/thor/ansible/inventory
```

Illustrative INI configuration; choose a group name consistent with any existing requirements:

```ini
[app_servers]
stapp03 ansible_host=stapp03 ansible_connection=ssh ansible_user=banner ansible_password='REDACTED_LAB_PASSWORD'
```

If the environment uses SSH keys, omit `ansible_password` and configure the appropriate key instead. A password may require `sshpass` on the control node.

## 3. Write the playbook

```bash
vi /home/thor/ansible/playbook.yml
```

Contents:

```yaml
---
- name: Create an empty file on App Server 3
  hosts: stapp03
  gather_facts: false
  become: false
  tasks:
    - name: Create empty file
      ansible.builtin.file:
        path: /tmp/file.txt
        state: touch
        mode: '0644'
```

Inspect it:

```bash
cat /home/thor/ansible/playbook.yml
```

## 4. Verify inventory parsing

```bash
cd /home/thor/ansible
ansible-inventory -i inventory --graph
```

Expected to include `stapp03`:

```text
@all:
  |--@ungrouped:
  |--@app_servers:
  |  |--stapp03
```

Avoid publishing `ansible-inventory --list` output because it may contain sensitive connection variables.

## 5. Establish host-key trust (if needed)

**Initial observed error:**

```text
Using a SSH password instead of a key is not possible because
Host Key checking is enabled and sshpass does not support this.
```

Verify the server's host-key fingerprint from a trusted lab source before accepting it:

```bash
ssh banner@stapp03
# Confirm the host key, sign in, then:
exit
ssh-keygen -F stapp03
```

`known_hosts` belongs to the account running Ansible, which here is `thor` on the jump host. There is no need to disable SSH host-key checking.

## 6. Test Ansible connectivity

```bash
command -v sshpass
ansible -i inventory stapp03 -m ping
```

Successful response shape:

```text
stapp03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Ansible `ping` tests remote execution and Python support, not ICMP connectivity.

## 7. Syntax-check and execute

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

The second command is exactly what the KodeKloud validator specifies, with **no additional arguments**.

Inspect the final recap: `unreachable=0` and `failed=0` indicate successful execution.

## 8. Verify the remote file

```bash
ansible -i inventory stapp03 -m stat -a 'path=/tmp/file.txt'
```

Required fields:

```text
exists: true
isreg: true
size: 0
```

Or, via SSH:

```bash
ssh banner@stapp03 'stat -c "%n %s bytes" /tmp/file.txt'
```

Expected:

```text
/tmp/file.txt 0 bytes
```

The ad-hoc `command` module, if used for `stat`, often reports `CHANGED` even when the shell command is read-only; prefer Ansible's dedicated `stat` module for accurate change reporting.

## 9. Final checklist

- [ ] Inventory path and hostname match task requirements.
- [ ] Connection variables authenticate noninteractively.
- [ ] Host key is trusted after fingerprint verification.
- [ ] Playbook path is `/home/thor/ansible/playbook.yml`.
- [ ] Play targets `stapp03` and creates `/tmp/file.txt`.
- [ ] Exact validation command completes without failure.
- [ ] Remote file exists, is regular, and is zero bytes.
- [ ] No real credentials have been committed to GitHub.
