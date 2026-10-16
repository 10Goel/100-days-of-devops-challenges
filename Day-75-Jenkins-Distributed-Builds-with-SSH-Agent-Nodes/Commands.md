# Day 75 — Commands and Jenkins Configuration Runbook

> **Execution convention:** Commands run from the KodeKloud jump host unless a different shell is explicitly named. Passwords are entered interactively from authorized lab credentials, never placed in public files. Only run installations if necessary. The instructions below reflect the working approach, not a claim that every diagnostic command ran in the successful attempt.

## 1. Target configuration

| Node | Host/user | Label | Remote root | Agent Java Path (after verification) |
|---|---|---|---|---|
| `App_server_1` | `tony@stapp01` | `stapp01` | `/home/tony/jenkins` | `/home/tony/java21` |
| `App_server_2` | `steve@stapp02` | `stapp02` | `/home/steve/jenkins` | `/home/steve/java21` |
| `App_server_3` | `banner@stapp03` | `stapp03` | `/home/banner/jenkins` | `/home/banner/java21` |

## 2. App Server 1 — Java and workspace

```bash
ssh tony@stapp01
sudo dnf install -y java-21-openjdk-headless
mkdir -p /home/tony/jenkins
chmod 755 /home/tony/jenkins

# Discover an executable from the installed RPM; do not reuse a path from another lab.
JAVA21=$(rpm -ql java-21-openjdk-headless | grep '/bin/java$' | head -n 1)
test -n "$JAVA21" && test -x "$JAVA21"
"$JAVA21" -version
ln -sfn "$JAVA21" /home/tony/java21
/home/tony/java21 -version
test -w /home/tony/jenkins && echo 'Agent directory writable'
exit
```

## 3. App Server 2 — Java and workspace

```bash
ssh steve@stapp02
sudo dnf install -y java-21-openjdk-headless
mkdir -p /home/steve/jenkins
chmod 755 /home/steve/jenkins
JAVA21=$(rpm -ql java-21-openjdk-headless | grep '/bin/java$' | head -n 1)
test -n "$JAVA21" && test -x "$JAVA21"
"$JAVA21" -version
ln -sfn "$JAVA21" /home/steve/java21
/home/steve/java21 -version
test -w /home/steve/jenkins && echo 'Agent directory writable'
exit
```

## 4. App Server 3 — Java and workspace

```bash
ssh banner@stapp03
sudo dnf install -y java-21-openjdk-headless
mkdir -p /home/banner/jenkins
chmod 755 /home/banner/jenkins
JAVA21=$(rpm -ql java-21-openjdk-headless | grep '/bin/java$' | head -n 1)
test -n "$JAVA21" && test -x "$JAVA21"
"$JAVA21" -version
ln -sfn "$JAVA21" /home/banner/java21
/home/banner/java21 -version
test -w /home/banner/jenkins && echo 'Agent directory writable'
exit
```

> The symlink paths are **explicitly tested** before they are entered into Jenkins. `rpm -ql` can return no match if the package is absent or packaged differently; if so, stop and inspect the package instead of creating a broken symlink. Avoid changing `alternatives --config java` globally.

## 5. Jenkins controller — trusted SSH host keys

```bash
ssh jenkins@jenkins
whoami                # jenkins
printf '%s\n' "$HOME" # /var/lib/jenkins
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts
```

After confirming each fingerprint with the trusted lab identity, connect and accept the host key if prompted:

```bash
ssh tony@stapp01 'hostname'
ssh steve@stapp02 'hostname'
ssh banner@stapp03 'hostname'
```

Each login may prompt for that server's password. Confirm all host entries are stored:

```bash
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
ls -l /var/lib/jenkins/.ssh/known_hosts
```

Optional strict verification (password prompts may still occur):

```bash
ssh -o StrictHostKeyChecking=yes tony@stapp01 'hostname'
ssh -o StrictHostKeyChecking=yes steve@stapp02 'hostname'
ssh -o StrictHostKeyChecking=yes banner@stapp03 'hostname'
```

Do not bypass host-key verification to make SSH succeed.

## 6. Jenkins UI — plugin

1. Sign in to Jenkins as the lab administrator.
2. Open **Manage Jenkins → Plugins → Installed plugins**.
3. Verify **SSH Build Agents** (ID: `ssh-slaves`) is enabled.
4. If absent, install it from **Available plugins**. Restart only when prompted, after plugin installation finishes and no jobs are running.

## 7. Jenkins UI — credentials

Open **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**. For each app server, add **Username with password** with the password from the lab server inventory:

| ID | Username | Description |
|---|---|---|
| `app-server-1-ssh` | `tony` | SSH for stapp01 |
| `app-server-2-ssh` | `steve` | SSH for stapp02 |
| `app-server-3-ssh` | `banner` | SSH for stapp03 |

Do not disclose the password in screenshots or output.

## 8. Jenkins UI — node configuration

Navigate to **Manage Jenkins → Nodes → New Node → Permanent Agent**. For each new node:

| Field | App_server_1 | App_server_2 | App_server_3 |
|---|---|---|---|
| Name | `App_server_1` | `App_server_2` | `App_server_3` |
| Executors | `1` | `1` | `1` |
| Remote root directory | `/home/tony/jenkins` | `/home/steve/jenkins` | `/home/banner/jenkins` |
| Labels | `stapp01` | `stapp02` | `stapp03` |
| Usage | Use this node as much as possible | Same | Same |
| Launch method | Launch agents via SSH | Same | Same |
| Host | `stapp01` | `stapp02` | `stapp03` |
| Port | `22` | `22` | `22` |
| Credentials | `app-server-1-ssh` | `app-server-2-ssh` | `app-server-3-ssh` |
| Host key strategy | Known hosts file | Same | Same |
| Advanced → Java Path | `/home/tony/java21` | `/home/steve/java21` | `/home/banner/java21` |

Save and **Launch agent**. Examine **Node → Log** before creating or troubleshooting the next node.

## 9. Verification

Navigate to **Manage Jenkins → Nodes** and check all three statuses are **Online**. For a successful launch, logs show:

```text
[SSH] Authentication successful.
[SSH] Starting sftp client.
[SSH] Copying latest remoting.jar...
[SSH] Starting agent process: cd "/home/.../jenkins" && /home/.../java21 -jar remoting.jar ...
channel started
Agent successfully connected and online
```

Optional execution test, using a Freestyle job restricted to a specific node label:

```bash
printf 'NODE_NAME=%s\n' "$NODE_NAME"
hostname
whoami
java -version
```

Repeat for `stapp01`, `stapp02`, and `stapp03`. This is optional validation and was not required as a separate task deliverable.

## 10. Troubleshooting commands

**Find the current Java 21 RPM executable** (on an app server):

```bash
rpm -ql java-21-openjdk-headless | grep '/bin/java$'
readlink -f "$HOME/java21"
"$HOME/java21" -version
```

**Verify workspace ownership and permissions:**

```bash
ls -ld "$HOME/jenkins"
test -w "$HOME/jenkins" && echo Writable
```

**Verify host trust** (on Jenkins Linux account):

```bash
ls -l ~/.ssh/known_hosts
ssh-keygen -F stapp01
ssh-keygen -F stapp02
ssh-keygen -F stapp03
```

**Failure interpretation:**

```text
No Known Hosts file was found              => Host-key trust missing
Authentication failed                      => Credential mismatch
UnsupportedClassVersionError 61 vs 55      => Java 11 too old for Java 17 bytecode
No such file or directory (Java executable) => Invalid configured Java Path
Exit code=127                              => Shell command not found / cannot execute
Agent successfully connected and online    => Remoting connection established
```

## 11. Submission

Submit with **Check** only when all three nodes are online, their labels and remote roots match the task, and launch logs show a successful agent connection.

**Observed outcome:** Day 75 passed.
