# Day 72 — Jenkins Parameterized Freestyle Job

![Jenkins](https://img.shields.io/badge/Jenkins-Parameterized%20Builds-D24939?logo=jenkins&logoColor=white) ![Challenge](https://img.shields.io/badge/KodeKloud-Day%2072-blue) ![Status](https://img.shields.io/badge/Status-Passed-brightgreen)

## Overview

Created and executed a **parameterized Jenkins Freestyle job** that accepts a free-form build stage and a controlled environment selection. The job displays both input values in its console output. This exercise demonstrates how build-time inputs make CI/CD automation reusable without editing job configuration between runs.

**Outcome:** KodeKloud Day 72 passed; Jenkins build **#1** finished with `SUCCESS`, using `Stage=Build` and `env=Staging`.

## Requirements

| Item | Required value |
|---|---|
| Job name | `parameterized-job` |
| Job type | Freestyle project |
| String parameter | `Stage` |
| String default | `Build` |
| Choice parameter | `env` |
| Choices (in order) | `Development`, `Staging`, `Production` |
| Build step | Execute shell; echo both values |
| Acceptance run | At least one successful build with `env=Staging` |

> Parameter identifiers are **case-sensitive**: `Stage` is not `stage` and `env` is not `ENV`.

## Workflow

```text
Jenkins Dashboard
       |
       v
New Item → Freestyle project → parameterized-job
       |
       v
General → This project is parameterized
       |                     |
       v                     v
String: Stage=Build    Choice: env
                        Development
                        Staging
                        Production
       \                     /
        \                   /
         v                 v
            Build with Parameters
             Stage=Build
             env=Staging
                  |
                  v
         Build step: Execute shell
                  |
                  v
         Jenkins Console Output
         Stage: Build
         Environment: Staging
                  |
                  v
             SUCCESS
```

## Implementation

### 1. Create job

Sign in to the Jenkins UI with your lab-provided admin credentials. Select **New Item**, enter `parameterized-job`, choose **Freestyle project**, then click **OK**.

### 2. Add parameters

Under **General**, check **This project is parameterized** and add:

- **String Parameter**: Name `Stage`, Default Value `Build`.
- **Choice Parameter**: Name `env`, choices on separate lines: `Development`, `Staging`, `Production`.

The first choice is the default in Jenkins unless overridden at build time. For this challenge explicitly select **Staging**.

### 3. Add shell build step

Go to **Build Steps → Add build step → Execute shell**, and paste:

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

Save the job.

### 4. Run and verify

Open **Build with Parameters**, enter `Stage=Build`, choose **Staging**, and click **Build**. Open **Build History → #1 → Console Output**.

Observed key output:

```text
Started by user admin
Running as SYSTEM
Jenkins Parameterized Job
Stage: Build
Environment: Staging
Finished: SUCCESS
```

The KodeKloud grader subsequently confirmed **Day 72 passed**.

## Validation checklist

- [x] Job `parameterized-job` created
- [x] `Stage` defined as a String Parameter, default `Build`
- [x] `env` defined as a Choice Parameter with all three exact choices
- [x] Execute shell prints both parameters
- [x] Build #1 executed with `env=Staging`
- [x] Console output contained the expected values
- [x] Jenkins build succeeded
- [x] KodeKloud validation passed

## Key takeaways

1. **Parameterization** separates configuration at run time from a job's reusable logic.
2. Jenkins passes Freestyle build parameter values to shell build steps as environment variables.
3. A **choice parameter** constrains inputs; a **string parameter** accepts free-form text.
4. Console output and each build's **Parameters** page provide evidence of the actual values used.
5. A successful process exit (`0`) produces `SUCCESS`; the shell script should fail explicitly when validation fails in more advanced jobs.

## Security

The sample prints only non-sensitive parameters. **Never echo secrets or passwords** into Jenkins logs. For secrets, use Jenkins Credentials and credential-binding mechanisms rather than ordinary String Parameters.
