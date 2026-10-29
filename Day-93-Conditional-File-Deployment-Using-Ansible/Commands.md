# Day 93 — Commands and Verification Runbook

Run all controller-side commands on the **jump host as `thor`**.

## 1. Inspect the project

```bash
cd /home/thor/ansible
pwd
ls -la
ansible-inventory -i inventory --graph
ls -l /usr/src/data/blog.txt /usr/src/data/story.txt /usr/src/data/media.txt
```

The lab inventory already exists; do not overwrite it. It must include `stapp01`, `stapp02`, and `stapp03`.

## 2. Check Ansible connectivity and privilege escalation

```bash
ansible -i inventory all -m ping
ansible -i inventory all -b -m command -a 'id -u'
```

Expected: `pong` from all hosts and user ID `0` for the privileged test. Correct SSH/sudo inventory variables only if necessary. Don't expose credentials in public documentation.

## 3. Create the playbook

```bash
cat > /home/thor/ansible/playbook.yml <<'EOF'
---
- name: Deploy host-specific files using when conditionals
  hosts: all
  become: true
  gather_facts: false

  tasks:
    - name: Create /opt/data directory
      ansible.builtin.file:
        path: /opt/data
        state: directory
        mode: '0755'

    - name: Deploy blog.txt to stapp01
      ansible.builtin.copy:
        src: /usr/src/data/blog.txt
        dest: /opt/data/blog.txt
        owner: tony
        group: tony
        mode: '0755'
      when: inventory_hostname == 'stapp01'

    - name: Deploy story.txt to stapp02
      ansible.builtin.copy:
        src: /usr/src/data/story.txt
        dest: /opt/data/story.txt
        owner: steve
        group: steve
        mode: '0755'
      when: inventory_hostname == 'stapp02'

    - name: Deploy media.txt to stapp03
      ansible.builtin.copy:
        src: /usr/src/data/media.txt
        dest: /opt/data/media.txt
        owner: banner
        group: banner
        mode: '0755'
      when: inventory_hostname == 'stapp03'
EOF
```

## 4. Check syntax and run exact validator command

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
ansible-playbook -i inventory playbook.yml
```

Expect every host to complete with `failed=0` and `unreachable=0`. Each host skips two nonmatching copy tasks.

## 5. Verify destination metadata

```bash
ansible -i inventory stapp01 -m command -a "stat -c '%n %U:%G %a' /opt/data/blog.txt"
ansible -i inventory stapp02 -m command -a "stat -c '%n %U:%G %a' /opt/data/story.txt"
ansible -i inventory stapp03 -m command -a "stat -c '%n %U:%G %a' /opt/data/media.txt"
```

Expected owner/group/mode: `tony:tony 755`, `steve:steve 755`, `banner:banner 755`, respectively.

## 6. Compare source and destination checksums

```bash
sha256sum /usr/src/data/blog.txt /usr/src/data/story.txt /usr/src/data/media.txt
ansible -i inventory stapp01 -m command -a 'sha256sum /opt/data/blog.txt'
ansible -i inventory stapp02 -m command -a 'sha256sum /opt/data/story.txt'
ansible -i inventory stapp03 -m command -a 'sha256sum /opt/data/media.txt'
```

Each source hash should equal the hash for its corresponding destination. Note: Ansible's `command` module may report `CHANGED` for read-only verification commands; this is not a file modification.

## 7. Repeat run to confirm convergence

```bash
ansible-playbook -i inventory playbook.yml
```

A second run typically shows `changed=0` with the same existing files and metadata.

## 8. If a task fails

```bash
ansible-inventory -i inventory --graph
ansible -i inventory all -m ping
ansible-playbook -i inventory playbook.yml -vv
```

Avoid posting verbose logs publicly if inventory credentials or other secrets are printed.
