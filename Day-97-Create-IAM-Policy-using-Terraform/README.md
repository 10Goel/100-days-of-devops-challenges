# Day 97 — AWS IAM Read-Only Policy Using Terraform

![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-IAM-FF9900?logo=amazonaws&logoColor=white) ![KodeKloud](https://img.shields.io/badge/KodeKloud-Day%2097%20Passed-brightgreen)

> **100 Days of DevOps · Day 97 · Completed successfully**

## Overview

Created an AWS customer-managed IAM policy called **`iampolicy_siva`** using Terraform. The policy grants read-only EC2 Describe access, including the ability to view EC2 instances, AMIs, and EBS snapshots. The exercise demonstrates permission documents, least privilege, and Terraform-managed identity infrastructure.

## Task requirements

| Item | Requirement |
|---|---|
| Policy name | `iampolicy_siva` |
| Cloud | AWS IAM |
| Lab provider region | `us-east-1` |
| Access | Read-only visibility in EC2 console |
| Covered resources | Instances, AMIs, snapshots |
| Working directory | `/home/bob/terraform` |
| File to author | `main.tf` |
| Existing file | Preserve lab-provided `provider.tf` |

**IAM is global per AWS account.** The lab's provider uses `us-east-1`, but the managed policy is not a regional object.

## Architecture

```text
Developer (bob@iac-server)
       |
       v
 /home/bob/terraform
   |-- provider.tf  (provided by lab; do not overwrite)
   `-- main.tf
       |
       v
 AWS Provider -> AWS IAM
       |
       v
 Customer-managed policy: iampolicy_siva
       |
       `-- Allow ec2:Describe* on Resource "*"
              |-- DescribeInstances
              |-- DescribeImages (AMIs)
              `-- DescribeSnapshots
```

## Implementation

The solution uses `aws_iam_policy` and Terraform's `jsonencode` function. Complete configuration is in [`main.tf`](main.tf):

```hcl
resource "aws_iam_policy" "siva" {
  name        = "iampolicy_siva"
  description = "Read-only access to EC2 instances, AMIs, and snapshots"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Sid      = "EC2ReadOnlyAccess"
      Effect   = "Allow"
      Action   = ["ec2:Describe*"]
      Resource = "*"
    }]
  })
}
```

`ec2:Describe*` grants a broad set of **Describe** operations. It does **not grant** `RunInstances`, `TerminateInstances`, or other write operations. It does not explicitly deny them either; effective access also depends on other identity policies, SCPs, permission boundaries, etc.

## Reproduce the lab

1. Work in `/home/bob/terraform` with the existing `provider.tf`.
2. Add the contents of [`main.tf`](main.tf).
3. Run:

```bash
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply
```

4. Confirm the managed resource in Terraform state:

```bash
terraform state list
terraform state show aws_iam_policy.siva
terraform plan
```

The final plan in the completed lab reported:

```text
No changes. Your infrastructure matches the configuration.
```

5. Optionally verify the IAM policy and document using AWS CLI commands from [`Commands.md`](Commands.md).

## Outcome and verification

- **Challenge result:** Passed (KodeKloud Day 97).
- Terraform tracked the policy `aws_iam_policy.siva`.
- The policy ARN appeared in state refresh (lab account ID is intentionally omitted here).
- Final `terraform plan` returned **no changes**.
- `aws iam simulate-custom-policy` returned `InternalFailure: simulate_custom_policy action has not been implemented` in this lab. This is an unsupported lab API operation, **not evidence of a misconfigured policy**; the simulator test was not completed.

## Security and production guidance

- The challenge only creates a policy: **it does not attach it** to a user, group, or role.
- For stricter least-privilege use, enumerate the needed Describe actions instead of using the wildcard. Console features can also require additional read-only actions.
- `Resource = "*"` is necessary for many Describe APIs that do not support resource-level permissions.
- Avoid granting write privileges, and test effective access for the intended identity before production use.
- Keep Terraform state and credentials out of Git; use an encrypted, access-controlled remote backend when appropriate.
- Do not commit a real `provider.tf` containing any lab credentials. Use this repository as a portable example, not an exact copy of sensitive lab configuration.

## Key takeaways

1. IAM policy documents declare permissions; attachments determine which identities receive them.
2. Terraform `jsonencode()` safely produces IAM JSON from HCL objects.
3. `Describe` is a read-only API family, but wildcard scopes deserve review.
4. IAM policies are account-global, even if the Terraform AWS provider is set to a region.
5. A successful Terraform apply and clean plan do not substitute for validating real effective IAM permissions.
