# Day 100 — Commands Reference

> Commands were used or are suitable for reproducing the lab, depending on whether the AWS sandbox supports the relevant APIs. Run in `/home/bob/terraform`; preserve existing `provider.tf`. Do not paste a random AMI ID into `main.tf`.

## 1. Inspect workspace and existing resources

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf
cat main.tf
terraform state list
```

If Terraform state already manages unrelated resources, do not remove them from config without assessing the proposed destroys.

## 2. Confirm the existing SNS topic

```bash
aws sns list-topics --region us-east-1   --query "Topics[?contains(TopicArn, ':datacenter-sns-topic')].TopicArn"   --output text
```

It must return the ARN of `datacenter-sns-topic`; do not create another SNS topic.

## 3. Find an actually available Ubuntu AMI

Verify the example AMI from the task (observed to fail in the lab):

```bash
aws ec2 describe-images --region us-east-1   --image-ids ami-0c02fb55956c7d316   --query 'Images[].{ID:ImageId,Name:Name,State:State,Arch:Architecture}'   --output table
```

Observed issue: `InvalidAMIID.NotFound`. Retrieve available images and select a compatible Ubuntu `x86_64` image:

```bash
aws ec2 describe-images --region us-east-1   --owners 099720109477   --filters     'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*'     'Name=state,Values=available'   --query 'Images[].[ImageId,Name,Architecture,State]'   --output table
```

Some simulated KodeKloud environments do not implement owner-filtered image discovery; in that case inspect `aws ec2 describe-images` available in the lab, choose an appropriate Ubuntu AMI, and use its actual returned ID. The sample `main.tf` uses a Canonical data source appropriate for standard AWS; substitute a verified lab AMI if unsupported.

## 4. Create or inspect main.tf and outputs.tf

```bash
cat main.tf
cat outputs.tf
```

`main.tf` includes EC2 instance, CloudWatch alarm, SNS data source. `outputs.tf` exposes precisely `KKE_instance_name` and `KKE_alarm_name`.

## 5. Initialize, format, validate, plan

```bash
terraform init
terraform fmt
terraform validate
terraform plan
```

Fresh-state expectation: `2 to add, 0 to change, 0 to destroy` (data sources are read, not added). If the plan proposes unexpected deletes or replacements, review before proceeding.

## 6. Deploy

```bash
terraform apply
```

Confirm `yes` after reviewing the proposed changes. This deploys the EC2 instance and the CloudWatch metric alarm.

## 7. Inspect outputs and state

```bash
terraform output
terraform state list
terraform state show aws_instance.datacenter_ec2
terraform state show aws_cloudwatch_metric_alarm.datacenter_alarm
```

Expected named outputs:

```text
KKE_alarm_name = "datacenter-alarm"
KKE_instance_name = "datacenter-ec2"
```

## 8. Verify EC2 in AWS

```bash
aws ec2 describe-instances --region us-east-1   --filters 'Name=tag:Name,Values=datacenter-ec2'   --query 'Reservations[].Instances[].{ID:InstanceId,AMI:ImageId,State:State.Name,Type:InstanceType}'   --output table
```

## 9. Verify CloudWatch alarm parameters

```bash
aws cloudwatch describe-alarms --region us-east-1   --alarm-names datacenter-alarm   --query 'MetricAlarms[].{Name:AlarmName,Namespace:Namespace,Metric:MetricName,Statistic:Statistic,Period:Period,EvaluationPeriods:EvaluationPeriods,DatapointsToAlarm:DatapointsToAlarm,Threshold:Threshold,Comparator:ComparisonOperator,Dimensions:Dimensions,Actions:AlarmActions}'   --output json
```

Verify `AWS/EC2`, `CPUUtilization`, `Average`, `300`, `1`, `1`, `90`, `GreaterThanOrEqualToThreshold`, correct `InstanceId`, SNS ARN ending with `:datacenter-sns-topic`.

Inspect topic subscriptions if needed (may require permissions):

```bash
aws sns list-subscriptions-by-topic --region us-east-1   --topic-arn "$(aws sns list-topics --region us-east-1 --query "Topics[?contains(TopicArn, ':datacenter-sns-topic')].TopicArn | [0]" --output text)"
```

## 10. The mandatory final convergence check

```bash
terraform plan
```

Required before submitting:

```text
No changes. Your infrastructure matches the configuration.
```

A clean plan is a Terraform consistency check, not proof that an SNS subscriber actually received a notification.

## 11. Troubleshooting quick-reference

| Symptom | Diagnose | Response |
|---|---|---|
| `InvalidAMIID.NotFound` | Requested Ubuntu AMI unavailable in region/lab | Find a supported Ubuntu x86_64 AMI and update instance argument |
| `No matching SNS Topic found` | Incorrect name/account/region | Check `list-topics` and provider region |
| Wrong alert comparison | `GreaterThanThreshold` excludes exactly 90% | Use `GreaterThanOrEqualToThreshold` |
| Alarm monitors wrong instance | Dimension uses wrong ID | Reference `aws_instance.datacenter_ec2.id` |
| `INSUFFICIENT_DATA` | Missing or not-yet-arrived metrics | Check metric emission; not an automatic Terraform failure |
| SNS alert not received | Subscriptions/policies/actions | Inspect subscriptions, action ARN and CloudWatch alarm history |
| Persistent `terraform plan` diff | State/config/API drift | Read the exact diff; fix root cause rather than suppress with `ignore_changes` |

## 12. Resource cleanup (only in your own test account, never before grading)

```bash
terraform plan -destroy
# terraform destroy  # destructive; intentionally not run as part of the challenge
```

Do not remove the pre-existing SNS topic; it is not managed by this configuration.
