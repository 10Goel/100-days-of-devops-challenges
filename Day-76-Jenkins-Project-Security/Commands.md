# Day 76 — Configuration Runbook (`Commands.md`)

> This is a **Jenkins web UI task**, not a Linux-shell task. No SSH or system package installation is required when the necessary authorization plugin and strategy are already available. The procedures below are the configuration equivalents of commands, with optional *read-only* validation snippets.

## 1. Pre-flight checks

1. Open the Jenkins interface through the KodeKloud lab toolbar.
2. Log in using the **administrator credentials given in the lab** (do not write passwords into documentation).
3. Confirm that the existing **`Packages`** job is present.
4. Confirm that the existing user IDs are spelled exactly `sam` and `rohan`.
5. Check **Manage Jenkins → Plugins → Installed plugins** for **Matrix Authorization Strategy** (`matrix-auth`). Install/enable only if necessary; restart Jenkins only if required by the plugin installation.
6. Check **Manage Jenkins → Security** to verify that the current authorization strategy allows **project-based matrix permissions**. Avoid modifying global authorization if it already does.

## 2. Open the existing job configuration

```text
Jenkins Dashboard
  → Packages
    → Configure
      → General
        → Enable project-based security [✓]
```

Do **not** create another job with a similar name or copy over the existing configuration.

## 3. Configure inheritance

Select the exact dropdown value:

```text
Inheritance Strategy:
Inherit permissions from parent ACL
```

This means permissions assigned on the parent are inherited, *in addition to* grants made specifically on `Packages`.

## 4. Add the two identities

In the job security matrix:

```text
Add user → sam
Add user → rohan
```

Use **Add user**, not **Add group**.

## 5. Set explicit permissions

### `sam`

```text
Job → Build       [✓]
Job → Configure   [✓]
Job → Read        [✓]
All other boxes  [ ]
```

### `rohan`

```text
Job → Build       [✓]
Job → Cancel      [✓]
Job → Configure   [✓]
Job → Read        [✓]
Run → Update      [✓]
SCM → Tag         [✓]
All other boxes  [ ]
```

**Category matters:** `Update` is under **Run**, while `Tag` is under **SCM** in the lab UI. The exact UI column layout depends on installed plugins. Scroll or expand the appropriate row to find the labels.

Do not change `Anonymous` or `Authenticated Users` unless the task explicitly requires changing them. In the observed job-level configuration, both rows had **no explicit grants**.

## 6. Save and independently verify

1. Click **Save**.
2. Verify Jenkins navigates away from the edit form without error.
3. Reopen **Packages → Configure**.
4. Check:
   - Project-based security is enabled.
   - Inheritance reads **Inherit permissions from parent ACL**.
   - `sam` has **only** Build, Configure, Read explicitly granted.
   - `rohan` has **only** Build, Cancel, Configure, Read, Update, Tag explicitly granted.
5. Confirm unrelated job configurations were untouched.
6. Return to the KodeKloud challenge and click **Check**.

Expected result:

```text
You're done!
You have successfully completed the challenge.
```

## 7. Optional read-only validation through Jenkins Script Console

**Advanced / optional.** This is not required for the lab. Requires an authorized administrator and should be used only when permitted by the environment. The snippet *reads* one job's authorization property and does not modify security configuration.

```groovy
import jenkins.model.Jenkins
import hudson.security.AuthorizationMatrixProperty

def job = Jenkins.get().getItemByFullName('Packages')
assert job != null : 'Packages job was not found'

def property = job.getProperty(AuthorizationMatrixProperty)
println "Job: ${job.fullName}"
println "Project matrix property present: ${property != null}"
if (property != null) {
    println "Inheritance strategy: ${property.inheritanceStrategy?.class?.simpleName}"
    property.grantedPermissionEntries.each { permission, entries ->
        println "${permission.group.title} / ${permission.name}: ${entries}"
    }
}
```

The internal APIs and classes can differ by Matrix Authorization plugin release; if this snippet is unsupported, use the Jenkins UI as the source of truth rather than attempting to alter the plugin or configuration during the challenge.

## 8. Troubleshooting quick reference

| Symptom | Likely issue | Safe next action |
|---|---|---|
| `Enable project-based security` missing | Matrix plugin or strategy not active | Inspect installed plugin and global authorization strategy |
| User row missing after saving | Form submission failed or wrong identity type | Reload configuration; use **Add user** |
| Unexpected additional access | Parent/global or group grants are inherited | Inspect effective permissions and parent ACL |
| `HTTP 403 No valid crumb` | Stale CSRF token or session/proxy issue | Reopen a fresh authenticated form; do **not** disable CSRF |
| Permission unavailable | Plugin version or missing feature | Inspect plugin and exact permission category |
| Grader fails despite visible ticks | Unsaved configuration, wrong inheritance or permission category | Reopen job and compare each exact grant |

## 9. Final comparison

```text
[✓] Existing job Packages
[✓] Project-based security enabled
[✓] Inherit permissions from parent ACL
[✓] sam: Job/Build, Job/Configure, Job/Read
[✓] rohan: Job/Build, Job/Cancel, Job/Configure,
           Job/Read, Run/Update, SCM/Tag
[✓] No unrelated job changes
[✓] Configuration saved and verified
[✓] KodeKloud result: Passed
```
