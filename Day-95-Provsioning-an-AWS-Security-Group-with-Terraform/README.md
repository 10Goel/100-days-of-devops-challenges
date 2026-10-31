# Day 95 — Provision an AWS Security Group with Terraform

![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-7B42BC?logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-VPC%20Security%20Group-232F3E?logo=amazonaws) ![Challenge](https://img.shields.io/badge/KodeKloud-Day%2095%20Passed-brightgreen)

## Overview

The Nautilus DevOps team is migrating infrastructure to AWS incrementally. In this exercise, Terraform provisions a security group named `datacenter-sg` **in the existing default VPC** in `us-east-1`, exposing only the two specifically requested inbound ports.

**Challenge status:** Passed (reported by the participant). The commands and configuration below document the lab solution; they are not a live deployment.

## Acceptance criteria

| Property | Required value |
|---|---|
| AWS Region | `us-east-1` |
| VPC | Existing **default** VPC |
| Security group | `datacenter-sg` |
| Description | `Security group for Nautilus App Servers` |
| HTTP inbound | TCP `80`, source `0.0.0.0/0` |
| SSH inbound | TCP `22`, source `0.0.0.0/0` |
| Working directory | `/home/bob/terraform` |
| Terraform configuration | `main.tf`; retain supplied `provider.tf` |

> **Security warning:** `0.0.0.0/0` exposes SSH to all IPv4 addresses. This is a lab requirement, not a recommended production policy. Restrict SSH to trusted source IP ranges, use SSM Session Manager where appropriate, and apply least privilege.

## Architecture

```text
bob@iac-server — /home/bob/terraform/
    ├── provider.tf       (existing; region + credentials configuration)
    └── main.tf           (this exercise)
             |
             v
AWS provider — us-east-1
             |
     data.aws_vpc.default  (lookup; creates no VPC)
             |
             v
 aws_security_group.datacenter_sg
       ├── inbound HTTP TCP/80  ← 0.0.0.0/0
       ├── inbound SSH  TCP/22  ← 0.0.0.0/0
       └── outbound all IPv4 traffic → 0.0.0.0/0
```

## Implementation

1. Change into `/home/bob/terraform` and inspect `provider.tf` without overwriting it.
2. Add the resources from [`main.tf`](main.tf).
3. Initialize, format, validate, and inspect the plan.
4. Apply the reviewed plan.
5. Verify the group and ingress rules through state inspection and AWS CLI.

```bash
cd /home/bob/terraform
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply
terraform state list
terraform plan
```

The configuration uses `data "aws_vpc" "default"` to look up the existing VPC, and **separate** `aws_vpc_security_group_ingress_rule` resources to manage the two ingress permissions. The egress rule is explicit to avoid ambiguity in outbound behavior. It is not an extra ingress rule.

## Verify the result

```bash
aws ec2 describe-security-groups \
  --region us-east-1 \
  --filters 'Name=group-name,Values=datacenter-sg' \
  --query 'SecurityGroups[].{Name:GroupName,Description:Description,VpcId:VpcId,Ingress:IpPermissions}' \
  --output json
```

Confirm the group VPC ID matches the default VPC and that inbound TCP ports **80** and **22** both accept **`0.0.0.0/0`**. The final `terraform plan` should normally show **no changes**.

## Notes on compatibility and lifecycle

- Separate `aws_vpc_security_group_ingress_rule` and `aws_vpc_security_group_egress_rule` resources require a compatible modern AWS provider version. Keep the lab's existing `provider.tf` configuration intact and check provider support if validation fails.
- Do **not** mix inline rules inside `aws_security_group` with independently managed rule resources for the same group.
- Security groups are **stateful**: permitted request traffic gets its return traffic without an explicit reverse ingress rule.
- A newly created security group protects **no EC2 instance** until attached to that instance's network interface or another supported resource.
- Never commit `terraform.tfstate`, `.terraform/`, `.tfvars` files containing secrets, AWS keys, or debug logs.

## Lessons learned

Terraform data sources are for querying existing resources; managed resources express desired infrastructure. Resource references such as `aws_security_group.datacenter_sg.id` create implicit dependencies. `terraform plan` offers review before changes, while Terraform state tracks the resulting resources.

See [`Commands.md`](Commands.md) for a runnable command checklist and [`Notes.md`](Notes.md) for deeper Terraform, AWS networking, and troubleshooting explanations.
