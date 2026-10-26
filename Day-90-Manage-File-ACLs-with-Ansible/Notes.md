# Day 90 — Linux ACL and Ansible Deep-Dive Notes

## 1. Why ACLs exist

Traditional UNIX permissions have three classes: owner, owning group, and others. Access Control Lists permit additional **named users** or **named groups** to receive permissions without changing the owning user or group.

Day 90 keeps all files owned by `root` but grants additional access on specific application servers.

## 2. Requirements at a glance

| Host | Path | Named ACL entry | Meaning |
|---|---|---|---|
| `stapp01` | `/opt/security/blog.txt` | `group:tony:r--` | Members of group `tony` may read |
| `stapp02` | `/opt/security/story.txt` | `user:steve:rw-` | Named user `steve` may read and write |
| `stapp03` | `/opt/security/media.txt` | `group:banner:rw-` | Members of group `banner` may read and write |

## 3. Anatomy of a POSIX ACL

Example:

```text
# file: /opt/security/story.txt
# owner: root
# group: root
user::rw-
user:steve:rw-
group::rw-
mask::rw-
other::---
```

- `user::` means *file owner* permissions.
- `user:steve:` means an **additional named user**.
- `group::` means *owning group* permissions.
- `group:banner:` means an **additional named group**.
- `mask::` limits the effective permissions of named users, named groups, and the owning group (but not the file owner or others).
- `other::` applies when no earlier ACL user/group match grants access.

`getfacl` can show `#effective:` annotations when an ACL mask restricts the permission bits written on an entry. For example, `user:steve:rw- #effective:r--` would **not** meet a read/write requirement.

## 4. User ACL versus group ACL

```yaml
ansible.posix.acl:
  path: /opt/security/story.txt
  etype: user
  entity: steve
  permissions: rw
  state: present
```

Compare with:

```yaml
ansible.posix.acl:
  path: /opt/security/media.txt
  etype: group
  entity: banner
  permissions: rw
  state: present
```

`etype` must match the requirement; a user ACL and a group ACL are not interchangeable even if the names happen to be similar.

## 5. Why root ownership and ACL permissions can coexist

```yaml
ansible.builtin.copy:
  content: ''
  dest: /opt/security/blog.txt
  owner: root
  group: root
  mode: '0640'
```

The file owner remains `root`. The subsequent ACL task adds a separate named permission entry. The on-disk mode displayed by `ls -l` may reflect the ACL **mask** rather than only the original owning-group permissions, so rely on `getfacl` for detailed verification.

## 6. File creation: `copy` vs. `file`

- `file: state=touch` creates a missing file but **does not truncate** a pre-existing file and updates timestamps on repeated execution.
- `copy: content=''` enforces **empty contents**, while simultaneously specifying owner, group and base mode.
- Repeated `copy` runs with matching content and metadata are generally idempotent.

## 7. ACL prerequisites

The controller needs `ansible.posix` for `ansible.posix.acl`. Remote hosts generally require `setfacl` and `getfacl` from the `acl` package, plus filesystem support for ACLs. For this RPM-based lab, the playbook uses `ansible.builtin.yum: name=acl state=present`.

## 8. Importance of task ordering

Recommended sequence:

1. Install ACL tooling on the managed server.
2. Ensure `/opt/security` exists.
3. Create a root-owned empty file and set its base permissions.
4. Apply the named ACL last.

Applying the named ACL last avoids unintentionally changing the ACL mask through a later broad `chmod`/mode-setting operation.

## 9. Why `become: true` is used

Creating files directly under `/opt` and setting root ownership require privileged operations. Ansible connects using the identity from inventory and escalates as needed via sudo. Authentication (`ansible_user`) and privilege escalation (`become`) are separate concepts.

## 10. Security considerations

- Minimize permissions: `r` is different from `rw`; never grant writes unless needed.
- Use a **named group** if access should follow team membership; use a **named user** for individual access.
- Verify access and the ACL mask, not just the requested ACL entry.
- Prefer dedicated, restricted directory policies over broad `0777` access.
- Do not version-control SSH credentials, private keys, or real environment passwords.

## 11. Troubleshooting

| Symptom | Likely explanation | Next check |
|---|---|---|
| `couldn't resolve module/action ansible.posix.acl` | Collection missing | `ansible-doc ansible.posix.acl` |
| `setfacl: not found` | Remote `acl` package missing | Install `acl` |
| `Operation not supported` | Underlying filesystem limitations | Inspect filesystem/mount |
| `Permission denied` | SSH/sudo insufficient | Test `ansible -b ... id -u` |
| ACL exists but effective write missing | Restrictive mask | `getfacl -p PATH` |
| Wrong user/group ACL | `etype`/`entity` mismatch | Compare task and requirement |
| File size nonzero | Contents not reset | Use `copy: content: ''` |
| Wrong ownership | Owner/group not enforced | `stat` and `getfacl` |

## 12. Interview questions

**Q1. Why use ACLs rather than chmod alone?**  
ACLs allow named users/groups to have additional rights without changing file ownership or relying only on the three traditional classes.

**Q2. What is an ACL mask?**  
The maximum effective rights available to named ACL users, named groups, and the owning group.

**Q3. Difference between `etype: user` and `etype: group`?**  
They grant permissions to a named user account versus a named group membership.

**Q4. Does `owner: root` prevent a named user from writing?**  
No. A named-user ACL with `rw` and a suitable mask can authorize writing while ownership remains root.

**Q5. What is the difference between `setfacl` and `getfacl`?**  
`setfacl` configures ACLs, while `getfacl` displays them.

**Q6. Why keep the ACL task after the file creation task?**  
The file must exist before an ACL can be set, and subsequent mode changes can modify the mask.

**Q7. How is an ACL playbook idempotent?**  
With `state: present`, Ansible ensures the specified ACL exists with requested permissions and avoids unnecessary modification when already correct.

---

**Status:** KodeKloud Day 90 passed. Configuration examples are the documented solution, not claimed raw logs from the user's session.
