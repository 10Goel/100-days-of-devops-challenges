# Day 76 — Jenkins Authorization: Fundamentals to Intermediate (`Notes.md`)

## 1. Authentication vs authorization

**Authentication** answers **“Who are you?”**: Jenkins verifies a user's identity through a configured *security realm* (for example, its built-in user database, LDAP, or another identity provider).

**Authorization** answers **“What may you do?”**: Jenkins checks the chosen *authorization strategy* and its permissions against the user and the resource being accessed.

```text
User signs in
     |
     v
Security realm authenticates identity
     |
     v
Authorization strategy checks permissions
     |
     +--> Allowed: perform the operation
     +--> Denied: do not perform it
```

A Jenkins user existing in the user database does **not** imply that they can build or configure every job.

## 2. What is an ACL?

An **Access Control List (ACL)** records which identities (users or groups) are granted which permissions on a protected resource. Jenkins evaluates these grants when users request operations such as opening a job, starting a build, or editing the configuration.

In a job-scoped matrix, each row represents an identity and each column represents a permission.

## 3. Global authorization vs project-based authorization

**Global authorization** applies broadly across Jenkins, while **project-based matrix authorization** allows jobs or folders to define additional, narrower-scoped grants.

```text
Global Jenkins permissions
           |
           v
Top-level Packages job
  Parent ACL inherited
  + explicit job-level grants
```

Project-based permissions are not an automatic security boundary if a parent/global ACL already grants broader access. A user with global administrative permission continues to have broad access.

## 4. Inheritance strategies

For Day 76, the required value is:

> **Inherit permissions from parent ACL**

This adds the parent's applicable grants to the job's explicit grants. At the top level, the parent is effectively Jenkins's global authorization context; folder-contained jobs instead inherit from the appropriate containing item.

Other inheritance modes may appear depending on the installed Matrix Authorization plugin version, such as not inheriting permissions or inheriting global configuration only. Do not change the task's specified strategy.

### Explicit vs effective permissions

**Explicit grants** are the checked permissions directly assigned on `Packages`.

**Effective permissions** include those grants **plus** any permissions inherited from parent ACLs, group memberships, and globally privileged roles.

Example:

```text
sam explicit job grants: Build, Configure, Read
sam inherited parent grants: Overall/Read
Effective set: explicit grants + applicable inherited grants
```

Unchecked boxes do not act as explicit denials of permissions inherited elsewhere.

## 5. Day 76 permissions explained

| Category | Permission | Meaning |
|---|---|---|
| Job | Build | Start a new build of the job |
| Job | Configure | Change the job's configuration |
| Job | Read | View the job |
| Job | Cancel | Stop a running build of the job |
| Run | Update | Edit supported build/run metadata, such as description where allowed |
| SCM | Tag | Perform tagging actions exposed by the installed SCM-related plugin |

Some permissions are provided by plugins. Therefore, available columns and names vary between Jenkins instances.

### What was granted?

**`sam`:** Job/Build, Job/Configure, Job/Read.

**`rohan`:** Job/Build, Job/Cancel, Job/Configure, Job/Read, Run/Update, SCM/Tag.

This is about *exact explicit job-level grants*. It is not evidence that these users have no additional access inherited from parent or group grants.

## 6. Least privilege and delegation

The **principle of least privilege** means granting only the permissions needed for an individual's task. It reduces the potential impact of errors and account compromise.

However, some of the task-required grants are powerful. **Job/Configure** can be sensitive because users who can edit a job may be able to influence the commands it executes or how credentials are used. Production security reviews should evaluate plugin behavior, credential scope, agent restrictions, and the actual trust level of users before granting Configure.

## 7. Jenkins permissions are not just Linux permissions

Jenkins authorization controls who can use Jenkins UI/API features. It does **not** directly replace Linux filesystem permissions on the Jenkins controller or build agents.

A job can be authorized to run while the OS user that executes the build still lacks permission to write a particular directory, for example.

## 8. Why user and group rows matter

- **User row:** Direct grants to an individual account, such as `sam`.
- **Group row:** Grants shared by members of that group.
- **Anonymous:** Requests not associated with an authenticated user.
- **Authenticated Users:** A broad group containing authenticated identities, where supported.

An empty Anonymous or Authenticated Users row in the **job-level** matrix means no **explicit** permissions are granted there; inherited permissions may still apply.

## 9. Common configuration pitfalls

### Pitfall A: Wrong inheritance mode

Selecting a different inheritance strategy can change whether global/parent permissions remain effective and may cause the challenge grader to reject the setup.

### Pitfall B: Wrong category

`Update` in the observed interface is **Run/Update**, not a generic job configuration permission. `Tag` is **SCM/Tag**. The grader may distinguish them.

### Pitfall C: Checking everything in a row

Avoid broad toggles when a task asks for exact permissions. Selecting extra grants is a functional and security difference.

### Pitfall D: Clicking Check before saving

A visible, correctly checked matrix is not proof that Jenkins stored it. Click **Save**, reopen the configuration, and compare permissions.

### Pitfall E: HTTP 403 and invalid crumb

Jenkins uses CSRF protection for state-changing requests. An expired browser session, stale form, or proxy inconsistency can cause:

```text
HTTP 403 — No valid crumb was included in the request
```

Use a fresh authenticated configuration form. Do not disable CSRF protection merely to make a lab save succeed.

## 10. Intermediate security practices

1. Review both **explicit and inherited** permissions.
2. Preserve at least one authorized administrative account before editing global authorization.
3. Prefer dedicated service accounts and narrowly scoped credentials for automation.
4. Treat **Job/Configure** as a sensitive capability.
5. Verify changes by reopening the configuration page; use an authorized second session for effective-access testing when appropriate.
6. Avoid modifying unrelated job configurations during a scoped request.
7. Use configuration history/audit tooling when supported and appropriately configured in real environments.
8. Review plugin security advisories and keep Jenkins/plugin versions supported.

## 11. Day 76 verification case study

Observed Jenkins UI state:

```text
Job: Packages
Project-based security: Enabled
Inheritance: Inherit permissions from parent ACL

rohan → Job: Build, Cancel, Configure, Read
         Run: Update
         SCM: Tag

sam   → Job: Build, Configure, Read
```

The final KodeKloud task screen reported **successful completion**, verifying the submitted lab configuration met the challenge requirements.

## 12. Interview questions and short answers

**Q1. What is the difference between a security realm and an authorization strategy?**  
A security realm verifies identity; an authorization strategy determines allowed actions.

**Q2. Why use project-based matrix authorization?**  
To assign distinct permissions to users/groups on individual jobs or folders rather than relying only on broad global grants.

**Q3. Do unchecked job-level permission boxes revoke global permissions?**  
No. Matrix grants are generally additive; inherited/global permissions may remain effective.

**Q4. What does `Inherit permissions from parent ACL` do?**  
It includes the parent's applicable grants in addition to direct project-level grants.

**Q5. What is the difference between Build and Configure?**  
Build starts an execution; Configure changes the job definition. Configure may be especially sensitive.

**Q6. What does Run/Update control?**  
Editing supported metadata of an individual build record, subject to Jenkins/plugin behavior.

**Q7. Why can a user have more access than the checked project matrix suggests?**  
Because of inherited permissions, group membership, or other broader grants.

**Q8. What does a Jenkins crumb protect?**  
State-changing web requests against cross-site request forgery (CSRF).

**Q9. What should you do after clicking Save on a security matrix?**  
Reopen the settings and verify that the expected values persisted.

**Q10. What was the final outcome of Day 76?**  
The `Packages` job received the task-specific grants for `sam` and `rohan`, inherited the parent ACL, and the KodeKloud challenge passed.

---

**Learning takeaway:** Proper Jenkins authorization requires understanding *where* a permission is granted, *what* it enables, and *how* inherited ACLs affect the actual authorization decision.
