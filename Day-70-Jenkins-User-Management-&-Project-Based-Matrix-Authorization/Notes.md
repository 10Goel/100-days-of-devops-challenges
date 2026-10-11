# Day 70 — Jenkins Security and Matrix Authorization Notes

## 1. Authentication vs Authorization

- **Authentication:** How Jenkins determines *who* is signed in. For this lab the Jenkins internal user database authenticates `admin` and `yousuf`.
- **Authorization:** What a signed-in identity is allowed to do. The Matrix Authorization Strategy plugin evaluates those permissions.

```text
User enters credentials
        |
        v
Authentication (identity verified?)
        |
        v
Authorization (is requested action permitted?)
        |
        v
Render page / allow operation OR deny access
```

**Key distinction:** A user may log in successfully but still receive HTTP 403 because they lack permission to view an item. A failed password is an authentication failure; a permission-denied response is an authorization failure.

## 2. Why Matrix Authorization?

Matrix authorization lets administrators grant granular permissions to users or groups. Its grid consists of **identities (rows)** and **capabilities (columns)**.

Examples of capabilities:

| Category | Example permission | Purpose |
|---|---|---|
| Overall | Read | Access to basic Jenkins UI |
| Overall | Administer | Full administrative control |
| Job/Item | Read | View a job and its information |
| Job/Item | Build | Trigger a job build |
| Job/Item | Configure | Edit job definition |
| Job/Item | Delete | Delete the job |
| Job/Item | Workspace | Access build workspace data |
| Job/Item | Discover | Discover that a job exists, without full Read |
| Credentials | View/Manage/etc. | Credential-related access, depending on plugin/version |
| Agent | Build/Configure/etc. | Manage/use Jenkins build agents |

Exact labels and available capabilities vary by Jenkins version and installed plugins.

## 3. Global Matrix vs Project-based Matrix

**Global authorization** determines base permissions applied throughout Jenkins. **Project-based authorization** enables a separate permission matrix on individual jobs.

```text
GLOBAL MATRIX
  admin:   Overall/Administer
  yousuf:  Overall/Read
  guest:   (no permissions)
           |
           v
HELLOWORLD JOB MATRIX
  yousuf:  Job/Read
```

**Scope is important:** Job-specific permissions typically **add** capabilities at the item level. They do **not subtract** capabilities already inherited from the global matrix or broader-scope groups.

### Why not grant Job/Read globally?

Global Job/Read could permit viewing multiple Jenkins jobs, not just `Helloworld`. The exercise deliberately grants the user only the base Overall/Read globally and then grants access to one job locally.

## 4. Principle of Least Privilege (PoLP)

Grant only what an individual needs to do their work.

For this exercise:

- `admin`: Overall/Administer, to maintain Jenkins security and job configuration.
- `yousuf`: Overall/Read globally, to enter Jenkins.
- `yousuf` on `Helloworld`: Job/Read, to inspect that job without executing or changing it.
- Anonymous: no access.

**Security implications:** Excessive Build permission can execute job-defined operations; Configure permission can alter build steps; Workspace permission can expose artifacts or source files. Read permission itself may still reveal logs or build metadata, so even read-only access should be deliberate.

## 5. Users, Groups, and Effective Permissions

Jenkins permission evaluation can consider identities such as:

- An individual account (`yousuf`)
- `authenticated` (signed-in users)
- `anonymous` (not signed in)
- External groups/roles, if configured
- Permissions inherited from folders or parents

A restrictive row for `yousuf` does not neutralize a broad grant to `authenticated`. **Effective authorization requires reviewing all applicable entries**, not just one user row.

## 6. Admin Lockout Protection

Before saving a new authorization matrix, confirm at least one administrator retains **Overall/Administer**. A mistaken save can lock everyone out of managing the server. In a production deployment, use change control, backup, and a documented recovery procedure before altering authorization.

## 7. Jenkins CSRF Crumbs and HTTP 403

Jenkins uses anti-CSRF **crumbs** on state-changing requests, such as saving job configuration.

The earlier lab displayed:

```text
HTTP ERROR 403
No valid crumb was included in the request
/job/Helloworld/configSubmit
```

Potential reasons include a stale browser form, an expired session, changed session cookies, or a reverse-proxy mismatch.

**Response:** Reopen the Jenkins page through the canonical lab URL, authenticate, load a fresh Configure form, save, and verify. **Do not disable CSRF protection** merely to bypass the error.

### Why is a successful click not enough?

A frontend click is not evidence that the server accepted the write. Error pages or network failures can prevent persistence. Always reopen the form or inspect effective permissions after saving.

## 8. Jenkins Plugin Concepts

**Matrix Authorization Strategy** (plugin ID `matrix-auth`) provides the matrix authorization approach used in this lab. Installing a plugin and activating it can be separate steps; some changes require Jenkins to restart.

Safe plugin practice:

1. Check **Installed plugins** first.
2. Install only the needed plugin if missing.
3. Wait for installation to finish.
4. Restart only if required and when builds are not running.
5. Confirm the plugin is enabled and the relevant UI option appears.

## 9. Practical Verification Strategy

### Phase A — Identity

- Confirm user ID/full name in Jenkins Users list.
- Test login with the password from the **current** task attempt.
- Distinguish failed authentication from an authorization-denied page.

### Phase B — Global authorization

- Select project-based matrix authorization.
- `admin`: Overall/Administer.
- Target user: Overall/Read only.
- `Anonymous`: no permissions.
- Check broad groups and reopen Security after saving.

### Phase C — Job authorization

- Enable project-based security on `Helloworld`.
- Target user: Job/Read only.
- Save, reopen Configure, and check persistence.
- Log in as the target user: read the job, but cannot build or configure it.

### Phase D — Submit

Submit to the grader only when the effective behavior and saved configuration match the task.

## 10. Troubleshooting Case Study: Three Attempts

| Attempt | Grader result | Lesson |
|---|---|---|
| 1 | `kareem` did not have read-only permissions on `Helloworld` | The earlier 403 crumb error likely prevented a job-level configuration save; verify persistence |
| 2 | `mariyam` missing or password incorrect | Presence in Users list alone does not validate the password; test login |
| 3 | **Passed** with `yousuf` | Treat each lab as fresh and verify authentication, global matrix, job matrix, and effective access |

Note: The first two failure messages show what the grader reported; any causal explanation beyond those messages is diagnostic inference, not proven backend behavior.

## 11. Common Interview Questions

**Q1. What is the difference between authentication and authorization?**  
Authentication identifies the user; authorization decides what actions they can perform.

**Q2. Why use a project-based authorization strategy?**  
It permits global baseline permissions and additional permissions on specific Jenkins jobs, limiting access to designated projects.

**Q3. Does a job-level matrix override global permissions?**  
It cannot normally revoke permissions already granted at a broader applicable scope. Review global, group, folder, and inherited grants when diagnosing effective access.

**Q4. What is the minimum access needed to view one job?**  
Typically Overall/Read for basic Jenkins access, plus Item/Read on the job (and any necessary ancestor/folder access in a nested setup).

**Q5. Can a user with Job/Read execute a build?**  
Not solely with that permission. Build requires the applicable Job/Build permission, although other security and plugin-specific settings can affect access.

**Q6. Why preserve Overall/Administer before saving?**  
Removing the last administrator's access can cause a lockout.

**Q7. What does `No valid crumb` mean?**  
Jenkins rejected a state-changing request because the anti-CSRF token was missing or invalid.

**Q8. Why verify settings after saving?**  
To confirm that Jenkins accepted and persisted the configuration and that permissions work as intended.

**Q9. What is a danger of granting permissions to `authenticated`?**  
Every signed-in user might inherit them, making individual users more privileged than their row suggests.

**Q10. Does least privilege mean zero permissions?**  
No. It means granting the narrowest set of permissions necessary for the intended task.

## 12. Key Takeaways

- Separate identity creation and password verification from authorization setup.
- Use **Overall/Read globally** plus **Job/Read locally** for targeted view-only access.
- Preserve **Overall/Administer** for a trusted administrator.
- Avoid group-level access leaks and anonymous grants.
- Validate saved settings and real user experience before declaring success.
- CSRF failures must be diagnosed without weakening security controls.
