# Day 80 — Jenkins Chained Builds: Concepts, Diagnostics and Interview Notes

## 1. Why chained builds?

Chained builds split one workflow into independently auditable jobs:

```text
Deploy code (upstream)
       |
       +-- SUCCESS/stable --> Restart Apache (downstream)
       |
       +-- FAILURE/UNSTABLE --> Do not restart
```

Benefits: separation of concerns, independent ownership, clearer build history, explicit dependencies, and reduced risk of restarting a service when deployment fails. **This isn't the same as a multistage Pipeline:** here the components are distinct Jenkins jobs.

## 2. Upstream vs downstream

- **Upstream:** `xfusion-app-deployment` pulls updates from Gitea's `master` branch to `/var/www/html` on `stapp01`.
- **Downstream:** `manage-services` restarts `httpd` on `stapp01` and checks HTTP availability.
- **Trigger:** **Build after other projects are built**, watching the upstream job, with **Stable** selected.

### Jenkins build states

| Result | Meaning | Stable-only downstream? |
|---|---|---|
| SUCCESS | Job executed and completed successfully | Yes |
| UNSTABLE | Completed with a nonfatal issue, such as failing tests | No |
| FAILURE | Build step or command failed | No |
| ABORTED | Build was interrupted | No |
| NOT_BUILT | Stage/job not run | No |

The stable trigger evaluates the upstream build result, not merely whether Jenkins managed to start it.

## 3. How Jenkins knows a shell step failed

The Freestyle **Execute shell** step interprets the command's exit status. `set -e` makes the shell terminate on many failing simple commands; `set -u` rejects unset variables. In our job, failures in `ssh`, `git pull`, `systemctl` or `curl` propagate back to Jenkins and mark the build failed.

```text
Git authentication fails -> ssh remote step exits nonzero
  -> upstream FAILURE -> stable-only downstream not triggered
```

Failure propagation is a crucial correctness property. Avoid commands like `git pull ... || true` that falsely mark broken deployments successful.

## 4. Jenkins executor versus remote SSH host

The **Built-In Node** runs Jenkins build logic and `ssh` as the Jenkins Linux service account. App Server 1 runs the commands inside the SSH session as `sarah`.

```text
Jenkins controller executor             App Server 1
jenkins@jenkins                         sarah@stapp01
        |                                    |
        +------ SSH authentication --------->|
        |      git or sudo remote commands   |
```

Jenkins can execute a job even without configuring an SSH Build Agent, provided its own node allows builds and SSH credentials/host verification are correct. Production best practice is to prefer isolated agents and managed Jenkins credentials over controller-host builds.

## 5. Three separate security gates

1. **SSH host verification:** `known_hosts` ensures the server's public host key matches the trusted identity.
2. **SSH user authentication:** public/private key pair lets `jenkins` log into `sarah@stapp01` without interactive password input.
3. **Sudo authorization:** determines whether `sarah` can run a privileged service command as root without a password prompt.

Passing the first two does **not** grant the third.

### Observed failure

```text
Started by upstream project "xfusion-app-deployment" build number 3
Host: stapp01
User: sarah
Restarting httpd...
sudo: a password is required
Finished: FAILURE
```

**Diagnosis:** The downstream trigger was working; SSH had successfully authenticated, and command execution reached the remote host. The restart was blocked by the `sudo -n` requirement for noninteractive execution.

### Remediation and safety

A controlled sudoers entry can permit only the command(s) needed by the automation:

```sudoers
sarah ALL=(root) NOPASSWD: /usr/bin/systemctl restart httpd, /usr/bin/systemctl is-active --quiet httpd
```

- Validate executable path with `command -v systemctl`.
- Create the file with `visudo -f`, not an unsafe editor on the main sudoers file.
- Validate with `visudo -cf` before triggering jobs.
- Do not use `NOPASSWD: ALL`, embed sudo passwords, disable host-key checks, or change global policy without approval.
- Authorization to edit sudoers must already exist; an unprivileged account cannot self-grant permissions.

## 6. Git deployment: what `git pull --ff-only` does

Conceptual sequence:

```text
Before: local master A--B
        origin/master A--B--C
After:  local master A--B--C
```

`git pull --ff-only origin master` fetches the remote branch and moves the checked-out local branch forward **only if** the change is fast-forwardable. It won't create a merge commit or discard local commits. If Git histories diverge, investigate rather than using `reset --hard`.

The checkout is **already at `/var/www/html`**. Running `git clone` into `/var/www/html/web` would create the wrong deployment path. A clean working tree check helps protect server-side modifications. Git permission/remote authentication must be checked using the actual remote execution user, `sarah`.

## 7. Jenkins trigger types compared

| Technique | Initiator | Suitable scenario |
|---|---|---|
| Build Now | Human | Manual test or controlled deploy |
| Generic Webhook Trigger | External HTTP event | Day 79 push-driven deployment |
| Poll SCM | Jenkins scheduled polling | When webhooks cannot be used |
| Build after other projects are built | Upstream Jenkins result | **Day 80 downstream dependency** |
| Pipeline `build` step | Pipeline code | Multi-job orchestration in Pipeline as Code |

Day 80 does not require recreating the Day 79 webhook. The upstream job can be manually triggered to test the chain; success-gating is the key requirement.

## 8. Restart vs reload

- `systemctl restart httpd`: stops and starts Apache; matches Day 80 exactly and can cause a short interruption.
- `systemctl reload httpd`: reloads configuration when supported; not equivalent to the explicit task requirement.
- `systemctl is-active --quiet httpd`: returns success/failure without human-readable status output.
- `curl -fsS http://localhost:8080/ -o /dev/null`: checks HTTP availability, but not that a specific page version is deployed.

In production, accompany restart with application health and content checks, load-balancer probes, downtime management, and rollback planning.

## 9. Troubleshooting decision tree

```text
Downstream didn't start?
  -> Was upstream SUCCESS (stable)?
  -> Is "Build after other projects are built" enabled?
  -> Does Projects to watch contain exact upstream name?

Downstream started but failed before remote command?
  -> SSH key? known_hosts? DNS? user?

Connected to stapp01 but restart failed?
  -> sudo -n -l and sudoers exact command arguments

Restart succeeded but website failed?
  -> systemctl status httpd, ss -lntp, curl, Apache logs,
     load-balancer routing and expected document root
```

## 10. Interview questions and answers

**Q1. What is a downstream job?**  A separate job automatically scheduled based on the result of another, upstream job.

**Q2. Why choose Stable rather than Always?**  To keep service restarts from running after a failed or unstable deployment.

**Q3. How do you distinguish SSH authentication from sudo authorization?**  SSH establishes the remote user session; sudo separately checks which privileged commands that user may run.

**Q4. Why use `sudo -n`?**  It fails immediately rather than hanging or prompting during noninteractive CI/CD execution.

**Q5. Why use `git pull --ff-only`?**  To update a linear branch safely without silently merging divergent history.

**Q6. Why does `Finished: SUCCESS` on the upstream not guarantee the downstream succeeded?**  Each job has its own credentials, runtime environment, authorization, steps and build result.

**Q7. What was the Day 80 bug?**  Jenkins correctly launched `manage-services`, but `sarah` lacked noninteractive sudo authorization to restart Apache.

**Q8. How do you prove job chaining?**  Check downstream Console Output for `Started by upstream project` and confirm the upstream build result was stable.

## 11. Completed challenge outcome

KodeKloud reported **Day 80 PASSED**. The observed failure case and the permission-fix workflow are documented because they demonstrate root-cause analysis rather than just job configuration.
