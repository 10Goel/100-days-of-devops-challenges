# Day 69 — Jenkins Plugin Management: Beginner to Intermediate Notes

## 1. Why Jenkins uses plugins

Jenkins is an automation server whose core offers essential job scheduling, orchestration, and UI capabilities. **Plugins** extend Jenkins with new features: SCM integrations, credentials stores, pipeline steps, UI integrations, build tools, security realms, notifications, and more.

Common areas:

| Area | Example |
|---|---|
| Source code management (SCM) | Git plugin |
| Hosting-platform integration | GitLab plugin |
| Authentication / secrets | Credentials and SSH Credentials plugins |
| Pipeline steps | Pipeline-related plugins |
| Tool execution | Git client plugin |
| Notifications / reporting | Email and reporting plugins |

A plugin's **display name** can differ from its **plugin ID**. Plugin IDs help distinguish similarly named extensions.

## 2. Git vs Git client vs GitLab

### Git (`git`)

Adds Git SCM integration to Jenkins: checking out a repository, choosing branches, handling credentials, and using Git in job/pipeline SCM configuration.

### Git client (`git-client`)

Provides lower-level Git functionality used by Jenkins plugins. Depending on the configuration, operations can use command-line Git or supported Java implementations. It is a dependency used by the Git plugin, not a substitute for the full SCM integration.

### GitLab (`gitlab-plugin`)

Integrates GitLab events and build status with Jenkins. It can enable jobs to respond to events such as pushes and merge requests when a webhook and the appropriate job configuration are in place.

**Not equivalent:** GitLab plugin ≠ GitLab Branch Source plugin ≠ GitLab API plugin. Their use cases and IDs differ.

## 3. How a GitLab-triggered CI pipeline works

```text
(1) Developer commits/pushes
            |
            v
(2) GitLab stores commit
            |
            | HTTP webhook / event payload
            v
(3) Jenkins GitLab integration
            |
            | decides whether configured job should run
            v
(4) Jenkins schedules job on an agent
            |
            | Git plugin checks out repository
            v
(5) Build → Test → Package → Report
```

**Webhook:** An event-driven HTTP notification. It avoids relying solely on frequent SCM polling. For real integrations, secure endpoints, limit permissions, and validate applicable tokens/secrets.

## 4. Jenkins plugin installation lifecycle

```text
Available plugins
      |
      v
Resolve required dependencies
      |
      v
Download plugin files
      |
      v
Install / attempt to dynamically load
      |
      +--> Loaded successfully → Enabled
      |
      +--> Update requires restart
                 |
                 v
           Safe restart
                 |
                 v
           Dependencies reload
                 |
                 v
           Verify enabled plugins
```

Plugins may require other plugins and minimum dependency versions. The dependency graph can be **transitive**: A needs B, and B needs C. Failure in C can prevent both B and A from loading.

## 5. Plugin dependencies: understanding the Day 69 failure

The actual **Credentials** installation error stated that the loaded Bouncy Castle API plugin was below the required minimum:

```text
Loaded Bouncy Castle API:
2.30.1.82-277.v70ca_0b_877184

Required by Credentials:
2.30.1.83-289.v8426fcd19371 or higher
```

The installation progress page simultaneously said the Bouncy Castle API update had been installed but **needed a restart to take effect**. Downstream plugin failures included Credentials, SSH Credentials, Git client, Git, and GitLab.

**Diagnosis:** The core error was not that GitLab itself was misconfigured; a prerequisite plugin had not yet loaded its newer version.

**Fix taken:** Restart Jenkins after installation completed and jobs were idle. After rebooting, the UI showed Git, GitLab, and Git client installed and enabled.

**General rule:** Trace failure messages *upstream* until you find the first unmet version/dependency requirement.

## 6. Loaded vs installed vs enabled

- **Downloaded:** Plugin artifact was fetched; does not guarantee successful installation.
- **Installed:** Plugin files are present and Jenkins recognizes the installation.
- **Enabled:** Configured to be loaded. A plugin may still fail to load due to missing dependencies or compatibility issues.
- **Successfully loaded:** Jenkins has activated the plugin for use.
- **Healthy:** A Jenkins UI metric; **does not certify freedom from security vulnerabilities**.

A restart is sometimes required even when the installer says the new plugin package exists on disk.

## 7. Safe restart vs abrupt restart

**Safe restart** allows ongoing builds to finish before the controller shuts down. It reduces build interruption but still requires coordination with users of shared infrastructure.

A normal service restart may interrupt in-progress work. During the lab, the provided **Restart Jenkins when installation is complete and no jobs are running** checkbox was the best approach.

## 8. Common installation failures and diagnosis

| Symptom | Likely cause | First check |
|---|---|---|
| `Failed to load: ...` | Missing or incompatible dependency | Click **Details**; identify first unmet dependency |
| `Update required: ...` | Loaded plugin version is too old | Installed plugin version and whether restart is needed |
| Plugin stuck downloading | Network/update center issue | Jenkins update center connectivity and logs |
| Restart requested | Update cannot be dynamically applied | Wait until jobs complete; safely restart |
| Plugin installed but missing feature | Disabled plugin, dependent plugin issue, or missing configuration | Enabled state, logs, applicable job configuration |
| Plugin disabled | Admin choice or load problem | Installed plugins and relevant errors |
| Install succeeded but security warning appears | Known advisory in that version | Security notices and compatible patched versions |

### Recommended troubleshooting order

1. Record the exact top-level error.
2. Open **Details** on failing dependencies, not just the final plugin.
3. Identify the minimum version requirement and the loaded version.
4. Check whether an update is **awaiting restart**.
5. Complete pending installation work; restart safely if required.
6. Sign back in and verify actual loaded state.
7. Retry only plugins still missing or failing.
8. Consult Jenkins logs and compatibility documentation if unresolved.

Avoid blindly deleting artifacts or resetting Jenkins home directories.

## 9. Security: key lessons

The lab's final **GitLab 1.9.13** entry carried advisories for **cache confusion** and **improper access control allowing SSRF**.

**SSRF (Server-Side Request Forgery):** A flaw that can allow an attacker to make a server send unintended requests to internal or external destinations.

**Cache confusion:** Incorrect handling of cached content or access boundaries that can cause data to be served in an unintended context, depending on the specific vulnerability.

Operational best practices:

- Keep Jenkins core and plugins on compatible, supported versions.
- Review Jenkins security advisories before upgrades and installation.
- Test plugin upgrades on nonproduction Jenkins first.
- Back up Jenkins configuration and plan rollback procedures.
- Use least-privilege credentials; never hardcode secrets in Jenkinsfiles.
- Minimize unnecessary plugins and restrict who can install/manage them.
- Use protected webhooks and trusted source-code repositories.
- Treat a plugin health score as distinct from its security posture.

## 10. Next step: a conceptual Jenkinsfile

The Day 69 task required **plugin installation only**. The following is an **optional learning example**, not part of the lab and not a working GitLab integration without further configuration.

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Requires a reachable repository and suitable credentials if private.
                git branch: 'main', url: 'https://gitlab.example.com/team/project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Build stage placeholder"'
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Test stage placeholder"'
            }
        }
    }
}
```

### Pipeline vocabulary

| Term | Meaning |
|---|---|
| `pipeline` | Defines the Declarative Pipeline |
| `agent` | Where pipeline stages execute |
| `stages` | Ordered logical phases |
| `stage` | Named group of work, e.g., Test |
| `steps` | Actions performed inside a stage |
| `git` | SCM checkout step provided through Git integration |
| `sh` | Execute a shell command on a compatible agent |

## 11. Interview and review questions

**Q1. Why did Git and GitLab both fail?**  
They depended on plugins that could not load. The underlying Credentials plugin required a newer Bouncy Castle API version than the version currently loaded.

**Q2. Why did restarting help?**  
Jenkins had downloaded an updated Bouncy Castle API, but the running process needed a restart to activate it.

**Q3. Is Git client the same as Git plugin?**  
No. Git client provides supporting Git operations; Git adds higher-level Jenkins SCM integration.

**Q4. Does installing GitLab automatically configure a webhook?**  
No. GitLab integration requires appropriate GitLab/Jenkins configuration, endpoint access, and job trigger settings.

**Q5. Does plugin health 100 imply the version is secure?**  
No. The final UI reported health 100 alongside GitLab security advisories.

**Q6. How should you troubleshoot a plugin failure in production?**  
Capture errors, identify dependency and compatibility issues, plan changes, protect active builds, test upgrades, and verify after controlled restart.

**Q7. When is a Jenkins restart needed?**  
When the plugin update cannot be safely activated dynamically, especially when existing loaded dependencies are being updated.

---

**Bottom line:** Jenkins plugin management is not simply clicking **Install**. Effective administration requires dependency analysis, careful restarts, verification, and security awareness.
