# Day 72 — Jenkins Commands and UI Runbook

This challenge is **UI-driven**. There are no mandatory Linux terminal commands: the shell snippet below runs *inside Jenkins* as a Freestyle build step.

## 1. Create job

```text
Jenkins → New Item
Name: parameterized-job
Type: Freestyle project
Click: OK
```

## 2. Configure parameters

```text
Configure → General
[x] This project is parameterized

Add Parameter → String Parameter
  Name: Stage
  Default Value: Build
  Description (optional): Build stage to display

Add Parameter → Choice Parameter
  Name: env
  Choices (one choice per line):
    Development
    Staging
    Production
  Description (optional): Target environment
```

**Verification:** `Stage` begins with capital `S`; `env` is lowercase. Do not use comma-separated choices.

## 3. Configure the shell step

```text
Configure → Build Steps → Add build step → Execute shell
```

Paste:

```bash
#!/bin/bash
set -eu

echo "================================="
echo "Jenkins Parameterized Job"
echo "================================="
echo "Stage: $Stage"
echo "Environment: $env"
echo "================================="
```

Click **Save**.

### Command explanation

| Expression | Purpose |
|---|---|
| `#!/bin/bash` | Request Bash for the script |
| `set -e` | Stop on a failing simple command (subject to Bash exceptions) |
| `set -u` | Treat an unset variable as an error |
| `echo "Stage: $Stage"` | Print the current String Parameter value |
| `echo "Environment: $env"` | Print the current Choice Parameter value |

## 4. Execute the required build

```text
parameterized-job → Build with Parameters
Stage: Build
env: Staging
Click: Build
```

Then open:

```text
Build History → #1 → Console Output
```

Expected meaningful lines:

```text
Jenkins Parameterized Job
Stage: Build
Environment: Staging
Finished: SUCCESS
```

In the actual challenge, build **#1** displayed those values and finished `SUCCESS`.

## 5. Check recorded parameters

```text
parameterized-job → Build #1 → Parameters
```

Confirm:

```text
Stage = Build
env   = Staging
```

## 6. Optional: Test locally (NOT required for KodeKloud)

To understand environment variables on a personal Linux machine:

```bash
export Stage='Build'
export env='Staging'

printf 'Stage: %s\nEnvironment: %s\n' "$Stage" "$env"
```

Expected:

```text
Stage: Build
Environment: Staging
```

Do **not** confuse this local test with running the actual Jenkins job.

## 7. Troubleshooting reference

| Symptom | Check / fix |
|---|---|
| **Build with Parameters** missing | Enable **This project is parameterized**, save the job |
| `Stage` prints blank | Confirm parameter name and script capitalization |
| Wrong environment selected | Choose **Staging**, not the default **Development** |
| Choices appear as one combined value | Put each choice on a separate line |
| `unbound variable` | Correct missing or misspelled parameters; `set -u` exposes this |
| Job saves but no output | Ensure **Execute shell** is in Build Steps and saved |
| Job status is FAILURE | Open **Console Output**, inspect errors and exit codes |
| Jenkins Save gives CSRF crumb error | Reopen a fresh authenticated configuration page; do not disable CSRF protection |

## 8. Acceptance checklist

```text
[ ] Correct job name
[ ] String: Stage / Build
[ ] Choice: env / Development, Staging, Production
[ ] Shell echoes both inputs
[ ] A build runs with env=Staging
[ ] Console reports Finished: SUCCESS
```

**Security reminder:** Never paste credentials into the Execute shell script or output sensitive parameter values.
