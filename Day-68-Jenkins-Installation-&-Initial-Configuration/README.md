# 🚀 Day 68 — Jenkins Installation and Initial Configuration

![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white) ![Linux](https://img.shields.io/badge/OS-Debian%2FUbuntu-1793D1?logo=linux&logoColor=white) ![Package](https://img.shields.io/badge/Install-APT-2C3E50) ![Level](https://img.shields.io/badge/Level-Beginner%20%E2%86%92%20Intermediate-blue)

> **KodeKloud / xFusionCorp Industries — DevOps Challenge, Day 68**  
> **Objective:** Install Jenkins on the assigned Jenkins server using `apt`, start it using the `service` command, and complete the first-run wizard with the required administrator profile.

## 📌 Challenge Requirements

| Item | Requirement |
|---|---|
| Target host | Dedicated Jenkins server (accessed from jump host) |
| Installation | `apt` only |
| Service start | `service jenkins start` |
| Web UI | Jenkins application accessed via lab's **Jenkins** button |
| Admin username | `theadmin` |
| Admin full name | `Jim` |
| Admin email | `jim@jenkins.stratos.xfusioncorp.com` |
| Admin password | **Configured as specified by the lab** (not stored in this public repository) |

> [!IMPORTANT]
> Do not install Jenkins on the jump host. Do not commit SSH credentials, admin passwords, or the Jenkins initial unlock secret to Git.

## 🏗️ Environment and Architecture

```text
┌────────────────────┐       SSH (port 22)        ┌────────────────────────────┐
│ Jump Host          │ ─────────────────────────▶ │ Jenkins Server             │
│ thor@jumphost      │                            │ Linux + Java runtime       │
└────────────────────┘                            │ APT-installed Jenkins      │
                                                  │ Jenkins HTTP port 8080     │
┌────────────────────┐       Browser / lab UI      │ /var/lib/jenkins (JENKINS_HOME)│
│ Jenkins Web UI     │ ─────────────────────────▶ │                            │
└────────────────────┘                            └────────────────────────────┘
```

**Why Java?** Jenkins runs on the JVM. The Jenkins version you install determines which Java versions are supported; confirm requirements before installation.

## ✅ Implementation Workflow

### 1. Access the correct machine

From the jump host, SSH into the designated Jenkins machine. Verify identity with `hostname`, `whoami`, and `cat /etc/os-release` before changing packages.

### 2. Install the Java runtime

Check `java -version`. If a compatible Java runtime is absent, use `apt` to install an appropriate supported OpenJDK release (for modern Jenkins LTS, Java 21 is a standard choice when supported by the distribution).

### 3. Configure the Jenkins APT repository

Obtain the Jenkins project's current repository signing key from the **official documentation**, add the stable APT repository, run `apt update`, and install `jenkins` with `apt install`.

### 4. Start Jenkins

```bash
service jenkins start
service jenkins status
```

The challenge explicitly requires `service` to start Jenkins. Check the default port with `ss -lntp | grep ':8080'`.

### 5. Complete the setup wizard

1. Open the lab-provided **Jenkins** UI.
2. Read `/var/lib/jenkins/secrets/initialAdminPassword` *on the Jenkins server* and use it to unlock Jenkins; never publish its contents.
3. Select **Install suggested plugins** and wait for successful completion.
4. Create `theadmin` with the lab-specified password, full name `Jim`, and the required email address.
5. Confirm the Jenkins URL as appropriate for the lab, finish setup, and open the dashboard.

## 🔎 Validation and Expected Results

| Verification | Command / UI | Success indicator |
|---|---|---|
| Jenkins package present | `dpkg -l jenkins` | Package status starts with `ii` |
| Java available | `java -version` | Supported Java runtime reported |
| Jenkins running | `service jenkins status` | `active (running)` on systemd-based host |
| HTTP listener | `ss -lntp \| grep ':8080'` | Port `8080` is listening |
| Web service responds | `curl -I http://localhost:8080` | An HTTP response is returned |
| Admin user created | Jenkins UI → **People** / user settings | `theadmin` and correct profile details |
| Setup complete | Jenkins dashboard | First-run wizard no longer shown |

> [!NOTE]
> A successful HTTP response does not by itself prove that the full Jenkins setup is complete. Verify the dashboard and the administrator profile separately. Completion of this task should be recorded only after the lab's **Check** succeeds.

## 🛠️ Troubleshooting Guide

| Symptom | Diagnose | Next action |
|---|---|---|
| `service jenkins start` hangs/times out | `service jenkins status` | Wait for initialization; inspect logs before restarting |
| Jenkins service fails | `journalctl -u jenkins -n 100 --no-pager` | Identify Java, permissions, or runtime errors |
| Log file not found | `journalctl -u jenkins` | Modern systemd packages may log to the journal |
| Port not open | `ss -lntp \| grep ':8080'` | Confirm service started; check port conflict |
| Jenkins APT package unavailable | `apt policy jenkins` | Verify repository/key configuration and `apt update` output |
| Initial password file missing | `service jenkins status` | Confirm Jenkins has initialized and correct host is used |
| UI not accessible | `curl -I http://localhost:8080` | Separate service failure from browser/lab proxy issue |

## 📚 Jenkins Fundamentals Covered

This challenge introduces a *Jenkins controller*, its Java runtime, `JENKINS_HOME`, plugin installation, user administration, service management, and first-run security setup. For additional explanations of jobs, Pipelines, Jenkinsfiles, agents, stages, credentials, triggers, and CI/CD, see [Notes.md](./Notes.md).
## 📖 References

- [Official Jenkins installation on Debian/Ubuntu](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu)
- [Jenkins — Using Jenkins](https://www.jenkins.io/doc/book/using/)
- [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Securing Jenkins](https://www.jenkins.io/doc/book/security/)

---

**Outcome:** A reproducible checklist for installing and initializing Jenkins for the Day 68 lab, with verification steps and security-conscious GitHub documentation. This README does **not** assert a successful challenge check without independent confirmation.
