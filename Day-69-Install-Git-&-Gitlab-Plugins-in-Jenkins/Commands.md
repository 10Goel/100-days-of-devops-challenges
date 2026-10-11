# Day 69 — Jenkins Plugin Installation: Steps & Commands

> **Important:** This challenge is performed primarily in the **Jenkins web interface**. The following Linux commands are *optional diagnostic checks*, not installation requirements. Use them only when authorized and on the correct Jenkins host. Do not run them on the jump host by mistake.

## A. Required UI workflow

| Step | Location / action | Check |
|---|---|---|
| 1 | Lab header → **Jenkins** → Sign in using provided lab credentials | Dashboard opens |
| 2 | **Manage Jenkins → Plugins → Available plugins** | Available plugin catalog opens |
| 3 | Search **Git**; select exact plugin (ID `git`) | Correct plugin selected |
| 4 | Search **GitLab**; select exact plugin (ID `gitlab-plugin`) | Correct plugin selected |
| 5 | Click **Install** (or install separately if prior selection is cleared) | Download progress page opens |
| 6 | Review each failed entry's **Details** | Read the *earliest* cause |
| 7 | If prompted, tick **Restart Jenkins when installation is complete and no jobs are running** | Restart starts after work finishes |
| 8 | Wait for Jenkins login page; sign in again | UI is responsive |
| 9 | **Manage Jenkins → Plugins → Installed plugins**; search `git` | Both Git and GitLab enabled |
| 10 | **Plugins → Download progress** | No active plugin installation/update tasks |
| 11 | Lab challenge → **Check** | Submit for grading |

### Observed plugin versions (example from this lab)

```text
Git client    6.6.1       Enabled
Git           5.11.0      Enabled
GitLab        1.9.13      Enabled
```

Versions can change; the challenge checks plugin presence rather than requiring these exact observed versions.

## B. Observed failure and fix

**Failure details (Credentials plugin):**

```text
Failed to load: Credentials Plugin
Update required: bouncycastle API Plugin
Old loaded: 2.30.1.82-277.v70ca_0b_877184
Minimum:    2.30.1.83-289.v8426fcd19371
```

**Actual fix:** Download progress → tick **Restart Jenkins when installation is complete and no jobs are running** → wait for login → re-check **Installed plugins**.

Avoid force-reinstalling or removing plugin files before verifying the dependency version after restart.

## C. Optional read-only server checks

If you have approved SSH access to the **Jenkins server**, these commands can supplement UI verification:

```bash
# Confirm identity and host
whoami
hostname

# Show service status (when available on this Jenkins host)
sudo service jenkins status

# Show recent service logs on systemd systems
sudo journalctl -u jenkins --no-pager -n 100

# Basic HTTP response check against a local Jenkins instance
curl -I http://localhost:8080

# Inspect whether default HTTP port is listening
sudo ss -lntp | grep ':8080'
```

A successful HTTP response does not prove that every Jenkins plugin is enabled; **Installed plugins** remains the primary verification source.

### Check downloaded plugin artifacts (optional)

```bash
# Only if JENKINS_HOME is /var/lib/jenkins on this host
sudo ls -lah /var/lib/jenkins/plugins/ | grep -Ei 'git|credentials|bouncycastle'
```

Files in the plugins directory are **not** definitive proof that a plugin has loaded successfully.

## D. Safely restarting Jenkins

**Preferred in this lab:** Use the Jenkins checkbox on the plugin installation screen. It waits for installation to finish and for jobs to become idle.

For other authorized Jenkins environments, a controlled restart may also be initiated by administrators. **Do not restart a shared or production server without checking running builds and obtaining approval.**

## E. Do not do the following

```text
✗ apt install gitlab-plugin           # Not how Jenkins plugins are installed
✗ Remove ~/.jenkins or JENKINS_HOME    # Destroys server state
✗ Delete random *.jpi / *.hpi files    # Risks dependency breakage
✗ Restart blindly during active builds
✗ Commit administrator credentials to GitHub
```

## Final checklist

- [ ] Jenkins UI is accessible after restart.
- [ ] **Git** (`git`) is installed and enabled.
- [ ] **GitLab** (`gitlab-plugin`) is installed and enabled.
- [ ] No plugin installation/update jobs are still running.
- [ ] Security advisories are noted for production follow-up.
- [ ] KodeKloud **Check** has been run and outcome recorded separately.
