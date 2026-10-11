# 🧰 Day 68 — Jenkins Commands Reference

> **Scope:** Commands used for the installation challenge, followed by optional diagnostics and learning exercises. Run installation commands on the **Jenkins server**, **not the jump host**.
>
> **Privileges:** Lab instructions use the server's `root` account. For non-root users with appropriate permissions, prepend `sudo` to privileged commands.

## 1. Connect and verify the host

```bash
# Run on jump host. Use the Jenkins hostname shown in the lab.
ssh root@jenkins

# Run after SSH succeeds.
whoami
hostname
cat /etc/os-release
```

> The SSH password comes from the challenge prompt. Do **not** place it in a terminal command or GitHub repository. If `jenkins` does not resolve, consult the lab's host details rather than guessing another address.

## 2. Check Java

```bash
java -version
```

If a compatible version is missing, and the OS supports OpenJDK 21:

```bash
apt update
apt install -y fontconfig openjdk-21-jre
java -version
```

If unavailable:

```bash
cat /etc/os-release
apt-cache policy openjdk-21-jre
```

**Why:** Jenkins is a Java application. The required Java version varies by Jenkins release, so do not install a random unsupported version.

## 3. Configure the official Jenkins stable APT repository

```bash
apt install -y wget ca-certificates
mkdir -p /etc/apt/keyrings

# Official Jenkins 2026 Debian repository key.
wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

# Add repository with explicit key binding.
printf '%s\n' \
  'deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/' \
  > /etc/apt/sources.list.d/jenkins.list

apt update
apt-cache policy jenkins
```

> Repository keys rotate. If the key URL or repository instructions change, follow the latest [official Jenkins Debian/Ubuntu guide](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu), rather than disabling APT signature verification.

## 4. Install Jenkins using `apt`

```bash
apt install -y jenkins

dpkg -l jenkins
apt-cache policy jenkins
```

Check that `dpkg -l` begins with `ii` for the Jenkins package.

## 5. Start and inspect the service

```bash
# Required by the lab: start via service, not systemctl.
service jenkins start
service jenkins status

# Default Jenkins HTTP port.
ss -lntp | grep ':8080'

# Verify HTTP response from the local controller.
curl -I http://localhost:8080
```

**Status:** On a systemd host, look for `active (running)`. A temporary startup delay is possible while Jenkins initializes.

## 6. Retrieve the initial administrator unlock password

```bash
# Read privately; do not paste its output in chat, screenshots, logs or Git.
cat /var/lib/jenkins/secrets/initialAdminPassword
```

1. Open the lab's **Jenkins** button in the browser.
2. Paste the unlock secret into the first-run screen.
3. Choose **Install suggested plugins**.
4. Create the named administrator account with the **password supplied by the task**, then finish the wizard.

**Required profile (safe to document):**

```text
Username: theadmin
Full name: Jim
Email: jim@jenkins.stratos.xfusioncorp.com
Password: [provided in lab task; do not commit]
```

## 7. Troubleshooting commands (run only if needed)

### Service and logs

```bash
service jenkins status
journalctl -u jenkins -n 100 --no-pager
journalctl -u jenkins -f

# The file might not exist on some packaging versions.
if [ -f /var/log/jenkins/jenkins.log ]; then
  tail -n 100 /var/log/jenkins/jenkins.log
fi
```

### Port, connectivity, memory

```bash
ss -lntp | grep ':8080'
curl -I http://127.0.0.1:8080
free -h
df -h
```

### Java, package, repository

```bash
java -version
which java
dpkg -l | grep -E '^(ii|hi)\s+(jenkins|openjdk)'
apt-cache policy jenkins
ls -l /etc/apt/sources.list.d/jenkins.list /etc/apt/keyrings/jenkins-keyring.asc
```

### Restart only when justified by diagnosis

```bash
service jenkins restart
service jenkins status
```

Do not repeatedly restart or reinstall the service without checking errors.

## 8. Optional commands: learning / post-lab checks

> **Not required for Day 68.** These commands are read-only unless noted otherwise.

```bash
# Check controller home directory and ownership (avoid exposing secrets).
ls -ld /var/lib/jenkins

# See the service's actual execution details.
service jenkins status

# Observe the process and listening sockets.
ps aux | grep '[j]enkins'
ss -lntp | grep ':8080'
```

### Important commands at a glance

| Command | Purpose | Required for Day 68? |
|---|---|---|
| `ssh root@jenkins` | Access target server | Yes (host may vary) |
| `java -version` | Verify prerequisite | Yes |
| `apt update` | Refresh package indexes | Yes |
| `apt install -y jenkins` | Install Jenkins | Yes |
| `service jenkins start` | Start Jenkins | **Yes — explicitly required** |
| `service jenkins status` | Verify running state | Yes |
| `ss -lntp` | Confirm listening port | Recommended |
| `cat /var/lib/jenkins/secrets/initialAdminPassword` | Unlock initial setup | Yes; private only |
| `journalctl -u jenkins` | Diagnose startup | If needed |

## 9. Final UI checklist

- [ ] Jenkins was installed with `apt` on the proper server.
- [ ] Jenkins was started via `service jenkins start`.
- [ ] Jenkins status is running.
- [ ] First-run wizard was unlocked.
- [ ] Suggested plugins were installed.
- [ ] User `theadmin` exists with the lab-required password.
- [ ] User full name is `Jim` and email is correct.
- [ ] Dashboard opens and the challenge's **Check** succeeds.

---

> **Safety note:** Avoid publishing the initial unlock secret, SSH password, admin password, or any Jenkins credential in public repositories.
