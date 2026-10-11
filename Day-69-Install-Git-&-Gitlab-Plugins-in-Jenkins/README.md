# Day 69 — Install Git and GitLab Plugins in Jenkins

![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white) ![Challenge](https://img.shields.io/badge/KodeKloud-100%20Days%20of%20DevOps-2563EB) ![Topic](https://img.shields.io/badge/Topic-Plugin%20Management-059669)

## Overview

The Nautilus DevOps team requires the **Git** and **GitLab** Jenkins plugins to support source-code checkout and integration with GitLab in upcoming CI/CD jobs. This exercise covers installation through the Jenkins web UI, resolving plugin dependency failures, safely restarting Jenkins, and validating the installed plugins.

> **Outcome:** The final Jenkins **Installed plugins** screen showed **Git 5.11.0** and **GitLab 1.9.13** installed and enabled, along with **Git client 6.6.1**. Final KodeKloud grader confirmation was not captured in the available evidence.

## Requirements

| Item | Requirement |
|---|---|
| Target | Preconfigured Jenkins server, accessed via the lab's **Jenkins** button |
| Authentication | Use credentials supplied in the lab task (not committed to the repository) |
| Plugin 1 | **Git** (`git`) |
| Plugin 2 | **GitLab** (`gitlab-plugin`) |
| Restart | Restart Jenkins if required after installation; wait for the login page |
| Verification | Both required plugins appear under **Installed plugins** and are enabled; no ongoing plugin installations |

## Architecture and purpose

```text
Developer
    |
    | git push
    v
GitLab repository
    |
    | Webhook (configured in a later exercise)
    v
Jenkins controller
    |  GitLab plugin: receives/processes GitLab events
    |  Git plugin: retrieves Git repository contents
    v
Jenkins pipeline / build agent
    |
    +--> Build --> Test --> Package --> Deploy (future work)
```

> Installing plugins **does not by itself** configure GitLab credentials, webhooks, Jenkins jobs, or a deployment pipeline.

## Implementation

1. Open Jenkins from the lab header and sign in using the provided administrator account.
2. Go to **Manage Jenkins → Plugins → Available plugins**.
3. Select **Git** (plugin ID: `git`) and **GitLab** (plugin ID: `gitlab-plugin`), then start installation. If the UI does not retain selections between searches, install them one at a time.
4. Observe installation progress. Additional dependencies are installed automatically.
5. If the installation page reports that a dependency update needs a restart, first inspect failure details, then select **Restart Jenkins when installation is complete and no jobs are running**.
6. Wait until Jenkins finishes restarting and the login page returns. Sign in again.
7. Navigate to **Manage Jenkins → Plugins → Installed plugins**, search `git`, and confirm the required plugins are **enabled**.
8. Check **Download progress** to ensure no installation/update jobs remain before using the lab's **Check** button.

## Troubleshooting performed: dependency failure

The initial installation reported failures for **Credentials**, **SSH Credentials**, **Git client**, **Git**, and **GitLab**. Clicking **Details** on the Credentials failure exposed the root cause:

```text
Failed to load: Credentials Plugin
Update required: bouncycastle API Plugin
Installed/loaded: 2.30.1.82-277.v70ca_0b_877184
Required:        2.30.1.83-289.v8426fcd19371 or higher
```

The Download progress page also stated that **Bouncy Castle API was already installed but required a Jenkins restart for the update to take effect**. The newer plugin was present but the running Jenkins instance was still using the older loaded dependency.

```text
Bouncy Castle API (outdated loaded version)
                |
                v
        Credentials fails
                |
        +-------+--------+
        |                |
        v                v
    Git client       Git / GitLab
    fails            fail to load
```

**Resolution:** Initiated the restart from the plugin installation page after installation activity finished. When Jenkins came back, the Installed plugins page showed the required Git and GitLab plugins enabled and healthy. No manual deletion or forced reinstall was necessary.

### Why the restart mattered

Jenkins dynamically loads some plugins, but updates to already loaded plugins and their dependencies may require a full restart. A failed dependent plugin can be a **symptom**; diagnose the **first failing dependency** before changing unrelated components.

## Verification evidence

The final Installed plugins page displayed:

| Plugin | Version observed | Enabled | Health displayed |
|---|---|---|---|
| Git client | `6.6.1` | Yes | 100 |
| Git | `5.11.0` | Yes | 100 |
| GitLab | `1.9.13` | Yes | 100 |

**Security observation:** Jenkins displayed GitLab plugin advisories relating to **cache confusion** and **improper access control allowing SSRF**. Plugin health `100` does **not** mean the version has no known vulnerabilities. Review security advisories and evaluate patched alternatives before using a similar setup in production.

## Key takeaways

- Jenkins plugin names and plugin IDs matter: **Git**, **Git client**, and **GitLab** are distinct.
- Plugin installations often pull in multiple transitive dependencies.
- Inspect **Details** on the **earliest dependency failure**, not only the last failed plugin.
- A **restart** can activate an updated dependency that could not be safely hot-reloaded.
- Verify plugins under **Installed plugins** after Jenkins is back online.
- Treat Jenkins security advisories separately from successful installation or health indicators.

---

**Challenge:** KodeKloud 100 Days of DevOps — Day 69  
**Area:** Jenkins · CI/CD · Plugin Lifecycle · Dependency Troubleshooting
