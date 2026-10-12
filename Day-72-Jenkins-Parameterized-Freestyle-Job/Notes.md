# Day 72 — Jenkins Parameterized Builds: Learning Notes

## 1. What is Jenkins?

**Jenkins** is an extensible automation server used to build, test, package, and deploy software. It can run tasks manually, on schedules, through SCM triggers, or based on external events. Plugins extend its capabilities.

### Core vocabulary

| Term | Meaning |
|---|---|
| **Job** | Saved automation configuration in Jenkins |
| **Build** | One execution of a job, with its own number, status, logs, and metadata |
| **Freestyle project** | UI-configured job with build steps and optional build parameters |
| **Pipeline** | CI/CD workflow expressed as code (often a `Jenkinsfile`) |
| **Parameter** | Input supplied to a job for a particular build |
| **Workspace** | Directory Jenkins uses when executing a job |
| **Executor** | Capacity unit that runs one build at a time |
| **Agent** | Machine/environment where workload executes |
| **Console Output** | Log of commands, messages, and results from a build |

## 2. Why use parameterized jobs?

Without parameters, a job might have a hard-coded environment such as `Development`; changing it could require editing job configuration. With parameters, the same job can run with different inputs:

```text
Same job: parameterized-job
    |
    +-- Stage=Build, env=Development
    +-- Stage=Build, env=Staging   ← Day 72 acceptance run
    +-- Stage=Test,  env=Production
```

This improves reuse, manual testing, and consistency. In a real deployment, production actions still require appropriate controls and approvals.

## 3. Common Jenkins parameter types

| Type | Typical use |
|---|---|
| String Parameter | Branch, build stage, image tag, version |
| Choice Parameter | Allowed environments or regions |
| Boolean Parameter | Toggle a test or optional stage |
| Text Parameter | Multiline text inputs |
| Password Parameter | Masks input in certain UI contexts, but **is not a replacement for Jenkins Credentials** |

Day 72 required exactly **one String Parameter** and **one Choice Parameter**.

## 4. How parameters reach the shell

For Freestyle **Execute shell** steps, Jenkins exposes build parameter values as environment variables.

```text
Jenkins Build with Parameters
    Stage = Build
    env   = Staging
          |
          v
Build environment
    Stage="Build"
    env="Staging"
          |
          v
Shell process
    echo "$Stage"   → Build
    echo "$env"     → Staging
```

**Case sensitivity:** Linux environment variable names are case-sensitive. `$Stage`, `$STAGE`, and `$stage` refer to different names.

Use quotes such as `"$Stage"` to preserve spaces and avoid accidental word splitting or glob expansion.

## 5. String vs Choice Parameters

**String**: Allows free text and can carry unexpected or empty input. Apply validation before using it in a command with side effects.

**Choice**: Restricts users to configured values when using the standard Jenkins UI, reducing accidental environment selection. This is an input convenience, **not an authorization boundary**.

For this task the choice list was:

```text
Development
Staging
Production
```

The first entry normally appears as the default; the required validation build explicitly used `Staging`.

## 6. Freestyle build lifecycle

```text
User clicks Build with Parameters
          |
          v
Jenkins records input values
          |
          v
Build queues / executor assigned
          |
          v
Environment variables populated
          |
          v
Execute shell starts
          |
          v
Commands execute and produce logs
          |
          v
Exit code determines step success/failure
          |
          v
Build status, duration and logs stored
```

A job can have multiple build steps; their order and failure behavior matter. For Day 72, one shell step was sufficient.

## 7. The actual Bash script

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

- **Shebang** requests Bash.
- **`set -e`** usually exits on a failing command, with exceptions in conditionals and other Bash constructs.
- **`set -u`** errors on unset variables, which can expose mistakes such as `$stage` instead of `$Stage`.
- **`echo`** writes to stdout, captured by the Jenkins console.

In an interactive Linux shell, `echo "$env"` works the same way only if `env` is defined. Jenkins defines it for the build from its parameter.

## 8. How to validate a Jenkins build properly

Do not check only whether the console contains a reassuring message. Verify:

1. **Job configuration** matches the requested names and defaults.
2. **Build parameters** recorded for the selected build match the input.
3. **Console output** includes both expected values.
4. **Build result** is `SUCCESS`.
5. **External grading** passes if the training platform uses a separate checker.

### Day 72 verified output

```text
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
Jenkins Parameterized Job
Stage: Build
Environment: Staging
Finished: SUCCESS
```

The KodeKloud Day 72 challenge was subsequently reported **passed**.

## 9. Troubleshooting: from symptom to root cause

| Symptom | Why it happens | Remedy |
|---|---|---|
| `Stage:` empty | Wrong/missing parameter value | Inspect saved parameters and selected build |
| Shell reports unset variable | Parameter/script case mismatch | Match `Stage` and `env` exactly |
| Choice list incorrect | Values entered on one line | Put one choice per line |
| Output shows `Development` | First choice selected by default | Explicitly select `Staging` |
| Build not triggered | Job not saved or parameter form not submitted | Save, then use Build with Parameters |
| CSRF crumb 403 while saving | Expired or invalid request token/session | Reauthenticate and load a fresh form; don't disable CSRF |
| Console ends in FAILURE | Script error or nonzero command exit | Examine the first relevant error in the console |

## 10. Intermediate concepts

### 10.1 Build reproducibility and traceability

Each build records its number, initiator, status, and parameter values. Two builds of the same job can behave differently if their inputs or workspace state differ. Logging selected **non-sensitive** parameters helps trace outcomes.

### 10.2 Parameter validation

The Day 72 script only prints inputs, so its risk is low. For real-world automation, validate allowed inputs before using them in deployment or system commands.

```bash
case "$env" in
  Development|Staging|Production) ;;
  *) echo "Invalid environment" >&2; exit 1 ;;
esac
```

This is a reusable example, **not part of the required Day 72 configuration**.

### 10.3 Avoid confusing a stage *parameter* with a Pipeline *stage*

In this challenge, `Stage` is just a user-defined string input such as `Build`. It does **not** create a Jenkins Pipeline stage. Pipeline stages are defined in Pipeline-as-Code.

### 10.4 Freestyle vs Pipeline

| Freestyle | Pipeline |
|---|---|
| Steps configured through the UI | Workflow stored as code |
| Suitable for small, simple jobs | Better for complex, versioned CI/CD workflows |
| Easier introductory learning | Supports stages, parallelism, approvals, and more |

For learning, the same behavior could later be expressed in a Declarative Pipeline:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'Stage', defaultValue: 'Build', description: 'Build stage')
        choice(name: 'env', choices: ['Development', 'Staging', 'Production'], description: 'Environment')
    }
    stages {
        stage('Print inputs') {
            steps {
                echo "Stage: ${params.Stage}"
                echo "Environment: ${params.env}"
            }
        }
    }
}
```

**This Jenkinsfile is an extension exercise, not a substitute for the Freestyle job the grader required.**

### 10.5 Security and input handling

- Build parameters are not a secure place for confidential credentials.
- Avoid logging access tokens, passwords, private keys, or secret values.
- Use Jenkins Credentials and appropriate bindings for secrets.
- Parameters control job *inputs*, not user *permissions*. Configure security and job-level authorization separately.
- Avoid constructing shell commands from untrusted free-text parameters without validation and safe quoting.

## 11. Interview questions with concise answers

**Q1. What is a parameterized Jenkins job?**  
A job that accepts values at build time rather than relying solely on hard-coded configuration.

**Q2. String vs Choice Parameter?**  
A String Parameter accepts typed text; a Choice Parameter presents predefined options.

**Q3. How do Freestyle build steps access parameters?**  
Typically as environment variables, e.g. `$Stage` and `$env` in Execute shell.

**Q4. Are Jenkins parameters case-sensitive?**  
Their names are case-sensitive in Linux shell execution, so capitalization must match.

**Q5. Why was Staging explicitly selected?**  
The acceptance criteria required at least one successful build with `env=Staging`; the first listed choice was Development.

**Q6. What does `Finished: SUCCESS` prove?**  
That Jenkins considered the build successful. Confirm job parameters and actual output separately for requirement-level verification.

**Q7. What does `set -u` do?**  
It makes references to unset shell variables fail instead of silently expanding to empty values.

**Q8. Is a `Stage` string parameter the same as `stage('Build')` in a Jenkinsfile?**  
No. One is a runtime input; the other declares a structural Pipeline stage.

**Q9. How should production credentials be supplied?**  
Through Jenkins Credentials with the appropriate permissions and binding, not plain String Parameters.

**Q10. Why use parameters at all?**  
They make a job reusable and let each build record the inputs behind its execution.

## 12. Final takeaway

Day 72 moves from a fixed Jenkins job to a reusable, parameter-driven job. The same job configuration accepted `Stage=Build` and `env=Staging`, printed both values, completed build #1 successfully, and passed the challenge.

---

**Related files:** [README.md](README.md) | [Commands.md](Commands.md)
