# Day 100 — Deep Notes: Terraform + EC2 + CloudWatch + SNS

> Study notes for the final KodeKloud 100 Days of DevOps challenge. Covers fundamentals, details of the actual task, operational design, troubleshooting, and interview preparation.

## 1. What was automated?

A Terraform configuration provisions an Ubuntu EC2 instance named **`datacenter-ec2`** and a CloudWatch metric alarm named **`datacenter-alarm`**. The alarm watches the **average** `AWS/EC2` **`CPUUtilization`** metric for that exact instance. If average CPU is **greater than or equal to 90%** for **one five-minute period**, the alarm transitions to ALARM and invokes an **existing** SNS topic named **`datacenter-sns-topic`**. Outputs expose resource names as `KKE_instance_name` and `KKE_alarm_name`.

## 2. Observability vocabulary

- **Metric:** numeric time-series measurement; EC2 CPUUtilization is expressed in percent.
- **Namespace:** logical AWS metric grouping (`AWS/EC2`).
- **Dimensions:** key/value qualifiers identifying a metric stream; here `InstanceId` is critical.
- **Statistic:** aggregation of observations within a period (`Average`, not `Maximum`).
- **Period:** duration of each metric aggregation/evaluation window, in seconds (`300`).
- **Threshold:** numeric limit against which a statistic is compared (`90`).
- **Evaluation period:** number of periods considered (`1`).
- **Datapoints to alarm:** number of breaching datapoints needed (`1`).
- **Alarm action:** something AWS triggers upon an alarm state transition, here SNS notification.
- **Alert vs alarm:** a CloudWatch alarm evaluates a time series and changes state; an SNS message is a notification mechanism.

### Numeric example

If the five-minute average CPU is **89.9%**, the datapoint is not breaching. If it is **90.0%**, it *is* breaching with `GreaterThanOrEqualToThreshold`. With `evaluation_periods=1` and `datapoints_to_alarm=1`, one breaching evaluation is enough to move to `ALARM` once evaluation has the required data. Minute-by-minute spikes do not necessarily imply a five-minute average above 90%.

## 3. CloudWatch alarm states

- **OK:** evaluated metric does not satisfy the alarm condition.
- **ALARM:** evaluated metric satisfies the condition.
- **INSUFFICIENT_DATA:** not enough usable data to determine alarm status, depending on the missing-data treatment.

The SNS action is usually invoked on transitions into ALARM, not continuously for every high-CPU sample. A new alarm may legitimately start in INSUFFICIENT_DATA. `treat_missing_data = "missing"` avoids falsely classifying missing samples as breaching; monitoring data availability is a separate operational concern.

## 4. Basic vs detailed EC2 monitoring

EC2 basic monitoring commonly publishes data at five-minute intervals; detailed monitoring can provide one-minute granularity. **Period=300** requests five-minute aggregation and is compatible with the challenge's five-minute threshold window. More frequent source data alone does not change the configured 300-second alarm period.

## 5. Terraform data source vs managed resource

```hcl
data "aws_sns_topic" "datacenter" { name = "datacenter-sns-topic" }
```

This reads an existing SNS topic ARN. It **does not create, modify or own** that topic. In contrast, `resource "aws_instance" ...` and `resource "aws_cloudwatch_metric_alarm" ...` create Terraform-managed infrastructure. This distinction prevents duplicate topic creation and accidental ownership of shared notification resources.

## 6. Dependency graph and references

```hcl
dimensions = { InstanceId = aws_instance.datacenter_ec2.id }
alarm_actions = [data.aws_sns_topic.datacenter.arn]
```

The first expression ties the alarm to the created instance and establishes an implicit dependency on the EC2 instance. The second uses the existing topic's ARN. Terraform constructs a directed acyclic graph, orders operations, and tracks managed resources in state. `depends_on` is not needed for these references.

## 7. Terraform lifecycle and drift

1. `terraform init` initializes the directory, downloads providers and configures backends.
2. `terraform fmt` normalizes HCL formatting.
3. `terraform validate` checks configuration structure and provider schema; it is not a successful-deployment guarantee.
4. `terraform plan` compares desired configuration with refreshed infrastructure and state, proposing actions.
5. `terraform apply` executes the approved plan and records state.
6. A **final `terraform plan`** should produce **No changes** to demonstrate the configuration has converged.

**Lesson from Day 98:** An API request or `apply` succeeding does not necessarily mean Terraform will produce a clean subsequent plan. A repeated diff could indicate eventual consistency, a provider bug, a forced/computed attribute, or real configuration drift. Avoid a reflexive `ignore_changes` workaround; it may conceal security or configuration changes. Investigate the diff and verify the live resource.

## 8. AWS EC2 basics

An **AMI** identifies an image used to boot an EC2 instance. It is region-specific and can be retired, deregistered, or missing from restricted training simulators. The originally suggested Ubuntu AMI `ami-0c02fb55956c7d316` was rejected with **`InvalidAMIID.NotFound`** in the observed lab. Because the task allowed another appropriate Ubuntu AMI, the deployment could use a different image after verifying availability. The actual final AMI ID was not supplied, so this repository uses a Canonical AMI lookup for standard AWS deployments; restricted labs may require a direct verified ID.

**EC2 type:** the example code uses `t2.micro`, a small lab-friendly size. The Day 100 task did not mandate a particular instance type. Production selection should consider availability, CPU credits, architecture, cost and workload requirements.

## 9. IAM and SNS delivery

Three separate concerns must be satisfied for useful alerts:

1. **CloudWatch action is configured**: `alarm_actions` includes the SNS topic ARN and `actions_enabled=true`.
2. **Topic permits publication**: appropriate SNS topic permissions and AWS service behavior allow the alarm action.
3. **Notifications are subscribed and confirmed**: email, Lambda, HTTPS, SQS or other subscription endpoints must be configured and healthy.

A correct Terraform alarm does not guarantee an email is delivered without these extra prerequisites. SNS is a messaging fan-out service, not a replacement for CloudWatch metric evaluation. The task did not require creating SNS subscriptions.

## 10. IAM and least privilege

- Scope the Terraform deployer to required EC2, CloudWatch and SNS read actions where feasible.
- Avoid static credentials committed to Git. Prefer short-lived AWS role credentials.
- Scope alarm action destinations and topic policies to trusted principals and resources.
- Protect Terraform state because it can expose infrastructure details and sometimes secrets.
- Prefer encrypted, locked remote state for team environments; retain `.terraform.lock.hcl` in version control.

## 11. HCL details and exact challenge settings

```hcl
alarm_name          = "datacenter-alarm"
namespace           = "AWS/EC2"
metric_name         = "CPUUtilization"
statistic           = "Average"
period              = 300
threshold           = 90
comparison_operator = "GreaterThanOrEqualToThreshold"
evaluation_periods  = 1
datapoints_to_alarm = 1
dimensions          = { InstanceId = aws_instance.datacenter_ec2.id }
alarm_actions       = [data.aws_sns_topic.datacenter.arn]
```

Why not use `GreaterThanThreshold`? Because it excludes exactly 90%. Why specify `datapoints_to_alarm=1`? Explicitly documents the intended one-of-one evaluation, although one evaluation period implies the same effective behavior in many configurations. Why use `Average`? It directly matches the grader specification and measures sustained CPU use rather than isolated peaks.

## 12. Validation and evidence

- **Terraform state**: `terraform state list`, `terraform state show ...`.
- **Terraform outputs**: exactly `KKE_instance_name`, `KKE_alarm_name`.
- **AWS EC2**: `aws ec2 describe-instances` verifies instance, AMI, type and state.
- **CloudWatch**: `aws cloudwatch describe-alarms` verifies `Period`, `Threshold`, `ComparisonOperator`, `Dimensions`, SNS ARN and evaluation periods.
- **SNS**: `aws sns list-topics` checks the topic exists.
- **Final consistency**: `terraform plan` must report **No changes**.

The KodeKloud Day 100 challenge was reported passed by the learner. The repository's sample configuration is a portable, documented reference and is not asserted to be the exact byte-for-byte final code from the lab.

## 13. Practical troubleshooting

### Ubuntu AMI missing

**Symptom:** `InvalidAMIID.NotFound` after `describe-images` or `apply`.

**Cause:** Invalid, deregistered, region-mismatched or simulator-unavailable AMI.

**Fix:** List available Ubuntu images for the current region/account, verify architecture and state, then update the selected AMI. Do not guess an unverified AMI ID.

### Missing SNS topic

**Symptom:** data lookup fails.

**Fix:** Confirm the topic name, region, account and permissions. The topic is pre-existing; don't create a duplicate.

### Alarm stays INSUFFICIENT_DATA

**Cause:** EC2 metric not yet published or metric dimensions incorrect; potentially expected immediately after creating a new alarm.

**Fix:** Verify the instance ID, CloudWatch namespace, period and metric emission. Don't arbitrarily change the threshold.

### No notifications

**Cause:** No confirmed subscriptions, SNS topic policy, or missing alarm state transition.

**Fix:** Verify `AlarmActions`, SNS topic policy, subscription status and alarm history. A notification isn't guaranteed merely because the alarm exists.

### Terraform plan still has changes

**Cause:** Drift, inconsistent provider behavior, or actual unapplied configuration.

**Fix:** Inspect the diff, verify live AWS attributes, repair and apply deliberately; never submit with pending changes when the grader requires a clean plan.

## 14. Interview questions and answers

**Q1. What is CloudWatch?** AWS observability service for collecting metrics, logs, events and evaluating alarms.

**Q2. What is the difference between a metric and an alarm?** A metric is measured data; an alarm evaluates a condition on that data and changes state.

**Q3. Why use `AWS/EC2`?** It identifies the namespace containing EC2 metrics such as CPUUtilization.

**Q4. Why use the `InstanceId` dimension?** It limits monitoring to a specific instance's metric series.

**Q5. What does `Average` mean?** The mean of the metric samples included in the evaluation period.

**Q6. Does 90.0% trigger this alarm?** Yes, with `GreaterThanOrEqualToThreshold`.

**Q7. What does `period = 300` mean?** A 300-second, five-minute metric evaluation period.

**Q8. What do `evaluation_periods = 1` and `datapoints_to_alarm = 1` mean?** One breaching five-minute datapoint is enough to meet the alarm condition.

**Q9. What is SNS?** AWS managed pub/sub messaging and notification fan-out service.

**Q10. Does an SNS topic automatically send emails?** No; a confirmed email subscription and successful delivery are required.

**Q11. What is a Terraform data source?** A read-only lookup of a resource outside this config's ownership, such as an existing SNS topic.

**Q12. Why reference the EC2 ID in the alarm?** Ensures the alarm monitors the instance created by Terraform and creates an implicit dependency.

**Q13. What are Terraform outputs?** Named values surfaced after an apply; here the required resource names.

**Q14. Is `terraform validate` enough to prove success?** No. It doesn't verify that AWS created the required live resources.

**Q15. Why run `terraform plan` after apply?** To verify there are no remaining managed-resource differences.

**Q16. What is infrastructure drift?** A divergence between deployed resources and Terraform's desired configuration or recorded state.

**Q17. What is `INSUFFICIENT_DATA`?** Alarm state indicating insufficient usable datapoints to evaluate the condition.

**Q18. What happens if CPU spikes briefly above 90%?** It may not alarm if the five-minute average remains below the threshold.

**Q19. Does creating an alarm install a CloudWatch agent?** No; EC2 CPUUtilization is an AWS-supplied metric and the alarm does not install any agent.

**Q20. How do you prevent accidental secret commits?** Protect credentials, use `.gitignore`, remote encrypted state, secret scanning and short-lived identities.

**Q21. Why might an AMI ID fail?** AMIs differ by region, account access, lifecycle and sandbox support.

**Q22. When is `ignore_changes` dangerous?** When it suppresses legitimate configuration or security drift from Terraform's plan.

**Q23. Does the alarm automatically reboot EC2?** No. This alarm has an SNS notification action, not an EC2 recovery/reboot action.

**Q24. How would you extend this in production?** Add reviewed network/IAM design, dashboards, multiple alarms, runbooks, incident routing, tagging, remote state, CI/CD validation and cost controls.

## 15. Takeaway

Day 100 connects the full DevOps loop: **provision → observe → detect → notify → verify convergence**. The main technical lesson is not merely creating an alarm, but correctly targeting the instance, choosing the right evaluation semantics, using existing notification infrastructure, and validating the deployed state before delivery.
