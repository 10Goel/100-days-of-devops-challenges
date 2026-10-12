# Day 71 — Jenkins Parameterized Automation: Learning Notes

## 1. Jenkins in a CI/CD workflow

**Jenkins** is an automation server that orchestrates jobs and pipelines. In this exercise it acted as an **orchestrator**, not the package-install destination: Jenkins started a shell step and the Storage Server performed the Linux package transaction.

```text
Jenkins controller / executor       Remote host
Receives build request              ststor01 (CentOS Stream 9)
Exposes PACKAGE                     Runs sudo dnf install
Starts shell script  ── SSH ──────► Verifies installed RPM
Receives exit code   ◄──────────── Returns status
Reports build result
```

Jenkins may execute shell steps on an agent rather than on the controller in larger systems. Never assume a build step runs on the controller without checking executor assignment.

## 2. Freestyle job vs Pipeline

| Attribute | Freestyle | Pipeline |
|---|---|---|
| Definition | Configured mostly through Jenkins UI | Commonly declared in a `Jenkinsfile` |
| Beginner-friendliness | Quick to configure | Requires pipeline syntax |
| Version control | Limited unless config-as-code tools are used | Strong when Jenkinsfile is in Git |
| Complex conditions, agents, approvals | Less convenient | Better suited |
| This task | **Freestyle + Execute shell** | Not required |

A Jenkins **job** is a stored automation configuration; a **build** is an individual execution of that job. Build numbers (#1, #2) help trace runs and diagnose failures.

## 3. Parameterized builds

A **parameter** lets the same job behave differently at runtime. Day 71 uses a **String Parameter** named `PACKAGE`. For Freestyle Execute shell steps Jenkins exposes it as an environment variable:

```bash
echo "$PACKAGE"         # Example: vim-enhanced
: "${PACKAGE:?Required}" # Fail if unset or empty
```

Other Jenkins parameter types can include Boolean, Choice, Text, Password, and plugin-provided parameter types. A parameter is input, **not automatically trusted**.

**Variable contrast:**

```text
PACKAGE        Jenkins parameter/environment variable
$PACKAGE       Shell expansion of variable value
'PACKAGE'      Literal text; no variable expansion
"$PACKAGE"     Expanded text protected from word splitting locally
```

## 4. Remote execution via SSH

SSH provides encrypted remote command execution. Authentication answers **who may connect**; authorization answers **what the account may do once connected**.

```text
jenkins@jenkins
  ~/.ssh/id_ed25519       private key — keep secret
  ~/.ssh/id_ed25519.pub   public key — install into remote authorized_keys
      │
      └──── SSH ────► natasha@ststor01
                     ~/.ssh/authorized_keys
```

- `ssh-keygen -t ed25519`: creates a key pair.
- `ssh-copy-id -i ...pub`: appends a public key to the target user's authorized keys.
- `known_hosts`: records trusted *server host keys*, distinct from the user's login keys.
- `ssh -o BatchMode=yes`: refuses interactive password/passphrase prompts; useful for CI jobs.
- The SSH command runs on whichever Linux user starts the build. Test keys as that user, not merely as root or the jump-host user.

**Security:** Verify host key fingerprints before trusting them; never use `StrictHostKeyChecking=no` as a blanket workaround. In production, favor Jenkins-managed SSH credentials/agents or a managed secret system rather than long-lived unprotected keys stored directly in a service home directory.

## 5. Remote sudo and least privilege

`sudo -n true` tests whether a command can execute under sudo noninteractively. The `-n` option means **non-interactive**: fail instead of prompting. The lab account successfully ran `sudo -n true`. For a production target, grant the automation identity access to only approved administrative operations; never broadly enable unrestricted passwordless sudo simply to make a job pass.

Note that successfully executing `sudo -n true` confirms this command is allowed; it does not, by itself, prove that every possible privileged command is allowed. The real DNF build confirmed the required install operation worked.

## 6. Linux package management (DNF and RPM)

The target ran **CentOS Stream 9** with `dnf`, whereas the Jenkins host ran **Ubuntu**. This is why OS detection was critical.

| Command | Function |
|---|---|
| `dnf install -y NAME` | Resolve dependencies and install an RPM package |
| `dnf info NAME` | Inspect package/repository details |
| `rpm -q NAME` | Query whether an RPM package is installed; nonzero exit if absent |
| `apt-get install` | Equivalent package-manager workflow on Debian/Ubuntu, with different package names |

`vim-enhanced` brought dependent packages such as `vim-common`, `vim-filesystem`, and `gpm-libs`; DNF handled the dependency resolution automatically.

## 7. Understanding the shell step

```bash
#!/bin/bash
set -euo pipefail

: "${PACKAGE:?PACKAGE parameter is required}"

if [[ ! "$PACKAGE" =~ ^[A-Za-z0-9][A-Za-z0-9._+:-]*$ ]]; then
    echo "ERROR: Invalid package name"
    exit 1
fi

ssh -o BatchMode=yes natasha@ststor01     "sudo -n dnf install -y -- '$PACKAGE'"
ssh -o BatchMode=yes natasha@ststor01     "rpm -q -- '$PACKAGE'"
```

- `#!/bin/bash`: execute with Bash (where honored by Jenkins shell step).
- `set -e`: exit on an unhandled failing command; Jenkins marks the step as failed.
- `set -u`: reject unset variables.
- `set -o pipefail`: a pipeline fails if any component fails.
- `: "${PACKAGE:?...}"`: fail early for missing/empty inputs.
- `[[ ... =~ regex ]]`: restrict allowed package-name characters and block common shell-injection syntax.
- `--`: marks the end of supported command options where accepted, to avoid accidental option parsing.
- `rpm -q`: machine-checkable installation verification.

The regex is a basic syntactic control. **Production jobs also need an allowlist** of permitted packages and user authorization, because even a syntactically valid package may be unsafe to install.

## 8. Exit codes and build status

Linux commands return `0` on success and nonzero on failure. Jenkins uses shell exit status to determine whether the build step passes.

```text
SSH/auth fails → nonzero → build fails
DNF install fails → nonzero → build fails
RPM query fails → nonzero → build fails
All succeed → zero → Finished: SUCCESS
```

Using a verification command after installation prevents a job from reporting success solely because a command was attempted.

## 9. Idempotency and repeatability

**Idempotency** means repeating an operation produces the same desired final state without unwanted changes. Re-running `dnf install -y vim-enhanced` when the installed package already satisfies the request generally does not reinstall it; it can finish successfully with a 'nothing to do' message. A repeatability test is valuable, though the supplied screenshots establish the successful first build and the user confirmed the grader passed, not a separately captured second build.

## 10. Remote execution vs Jenkins plugins

For Day 71, `Execute shell` plus SSH was enough. Alternative integration patterns include:

- SSH Agent plugin + Jenkins Credentials for short-lived SSH key injection.
- Publish Over SSH plugin for certain remote operations.
- Jenkins agent on the target for richer orchestration (changes trust and operational boundaries).
- Ansible launched by Jenkins for repeatable multi-host package management.
- Declarative Pipeline for version-controlled, reviewable CI/CD logic.

No additional plugin is inherently required by the task when shell-based SSH authentication is already working.

## 11. Failure scenarios and triage order

| Symptom | Check first |
|---|---|
| `Permission denied (publickey)` | Key owner, `authorized_keys`, remote username, key permissions |
| Password prompt stalls build | `BatchMode=yes`, SSH key and sudo behavior |
| Host key mismatch | Validate remote host identity, do not bypass checks |
| `sudo: a password is required` | Allowed sudo policy and exact command |
| `dnf` absent | Target OS and command path |
| `No match for argument` | Correct package spelling and enabled repositories |
| Red build after install | `rpm -q` name/version mismatch and exit code |
| Green build but missing target package | Check remote hostname and where commands execute |

## 12. Example migration to Jenkinsfile (learning only)

*Not part of the completed Freestyle task.* For a controlled production workflow, the job could move into version-controlled Pipeline-as-Code:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PACKAGE', defaultValue: 'vim-enhanced', description: 'Approved RPM package')
    }
    stages {
        stage('Install on Storage') {
            steps {
                sh """#!/bin/bash
                  set -euo pipefail
                  : "${PACKAGE:?PACKAGE required}"
                  [[ "$PACKAGE" =~ ^[A-Za-z0-9][A-Za-z0-9._+:-]*$ ]] || exit 1
                  ssh -o BatchMode=yes natasha@ststor01 \
                    "sudo -n dnf install -y -- '$PACKAGE'"
                  ssh -o BatchMode=yes natasha@ststor01 \
                    "rpm -q -- '$PACKAGE'"
                """
            }
        }
    }
}
```

This illustration assumes external SSH and sudo setup. Real pipelines should bind credentials securely and restrict allowed packages.

## 13. Interview revision questions

1. **What is a parameterized Jenkins build?** A job execution whose behavior is driven by runtime values such as `PACKAGE`.
2. **Why not run DNF on Jenkins itself?** The requirement targets the remote Storage Server; controller and target are separate machines.
3. **Why use SSH keys?** They allow noninteractive authentication without embedding login passwords in the build script.
4. **What does `sudo -n` change?** It fails rather than waiting for a password prompt.
5. **Why `rpm -q` after `dnf install`?** It validates package presence with an exit code.
6. **What does `set -euo pipefail` do?** It tightens shell error handling for automated builds.
7. **What is the difference between Freestyle and Pipeline?** UI-managed job steps versus commonly code-defined orchestration in a Jenkinsfile.
8. **How do you reduce command-injection risk?** Validate input, use allowlists, carefully quote values, and restrict the target account's privileges.
9. **What is idempotency?** Re-running automation leaves the target in the intended state without unwanted repeated changes.
10. **Why should host key checking remain enabled?** It helps detect unexpected SSH servers and man-in-the-middle attacks.

## 14. Learning outcome

This challenge connected Jenkins configuration with Linux operations, SSH identity and trust, remote privilege escalation, RPM package management, and reliable success/failure verification. The **first build completed successfully**, and the **KodeKloud Day 71 challenge passed**.
