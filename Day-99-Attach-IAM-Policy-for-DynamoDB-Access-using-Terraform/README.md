# Day 99 — Secure DynamoDB Access with Terraform & AWS IAM

![Terraform](https://img.shields.io/badge/Terraform-Infrastructure_as_Code-623CE4?logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-DynamoDB_%2B_IAM-232F3E?logo=amazonwebservices&logoColor=white) ![KodeKloud](https://img.shields.io/badge/100_Days_of_DevOps-Day_99-success) ![Status](https://img.shields.io/badge/Challenge-PASSED-brightgreen)

> **Challenge status:** Passed on KodeKloud. This repository reproduces the working lab approach and documents its security implications. 

## Overview

Provision an AWS DynamoDB table, an assumable IAM role, and a customer-managed permission policy granting **only `dynamodb:GetItem`, `dynamodb:Scan`, and `dynamodb:Query`** on **that table's ARN**. Attach the policy to the role. Define the required Terraform input variables, outputs, and values file.

### Architecture

```text
                            AWS account (lab provider)
                                     |
                       Terraform /home/bob/terraform
                                     |
                +--------------------+--------------------+
                |                                         |
       DynamoDB: nautilus-table                    IAM: nautilus-role
       Partition key: id (S)                      Trust: EC2 service
       Billing: PAY_PER_REQUEST                           |
                |                                  policy attachment
                |                                         |
                +---------> nautilus-readonly-policy <----+
                            Allowed: GetItem/Scan/Query
                            Resource: table ARN only
```

**Trust vs authorization:** The role trust policy permits `ec2.amazonaws.com` to request `sts:AssumeRole`. The attached permissions policy limits the assumed role's DynamoDB operations. This configuration **does not** attach the role to an EC2 instance; an instance profile and association would be required to use it from EC2.

## Exact challenge requirements

| Item | Value |
|---|---|
| DynamoDB table | `nautilus-table` |
| IAM role | `nautilus-role` |
| IAM policy | `nautilus-readonly-policy` |
| Allowed DynamoDB API actions | `GetItem`, `Scan`, `Query` only |
| IAM resource scope | ARN of `nautilus-table` only |
| Terraform directory | `/home/bob/terraform` |
| Inputs | `KKE_TABLE_NAME`, `KKE_ROLE_NAME`, `KKE_POLICY_NAME` |
| Outputs | `kke_dynamodb_table`, `kke_iam_role_name`, `kke_iam_policy_name` |
| Required files | `main.tf`, `variables.tf`, `outputs.tf`, `terraform.tfvars` |

## Repository layout

```text
Day-99-Terraform-DynamoDB-IAM/
├── README.md
├── Commands.md
├── Notes.md
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── .gitignore
```

`provider.tf` is **not included**: the KodeKloud environment provides it. The code assumes that an appropriate AWS provider is available in the working directory and uses the region configured by that provider (in the lab, `us-east-1`). In a standalone deployment, create your own suitable AWS provider configuration separately.

## Solution design

1. **DynamoDB table** — on-demand billing, string partition key `id`, tagged with the required name.
2. **IAM role** — trusted principal `ec2.amazonaws.com` with `sts:AssumeRole`.
3. **IAM permissions policy** — `jsonencode` generates valid JSON, granting three read APIs with `Resource = aws_dynamodb_table.nautilus_table.arn`.
4. **Role-policy attachment** — connects the managed policy to the role.
5. **Variables and outputs** — exact names required by the challenge; values loaded from `terraform.tfvars`.

The policy grants no write operations (`PutItem`, `UpdateItem`, `DeleteItem`, etc.). Note that read-only operations can still expose data and that a separate identity policy, permissions boundary, SCP, or explicit deny could affect effective permissions.

## Reproduce the lab

From the provisioned KodeKloud terminal:

```bash
cd /home/bob/terraform
# Retain the lab-provided provider.tf; place the four .tf files here.
terraform init
terraform fmt -check
terraform validate
terraform plan
terraform apply
terraform output
terraform state list
terraform plan
```

On a fresh workspace, expect four managed resources: `aws_dynamodb_table.nautilus_table`, `aws_iam_role.nautilus_role`, `aws_iam_policy.nautilus_readonly`, and `aws_iam_role_policy_attachment.nautilus_attach`. A clean final plan should report **No changes**. Resource counts can vary if pre-existing state is present.

### Verification commands

```bash
aws dynamodb describe-table --region us-east-1 --table-name nautilus-table   --query 'Table.{Name:TableName,Status:TableStatus,ARN:TableArn}' --output table

aws iam get-role --role-name nautilus-role   --query 'Role.{Name:RoleName,Trust:AssumeRolePolicyDocument}' --output json

aws iam list-attached-role-policies --role-name nautilus-role --output table

POLICY_ARN=$(aws iam list-policies --scope Local   --query "Policies[?PolicyName=='nautilus-readonly-policy'].Arn | [0]" --output text)
VERSION=$(aws iam get-policy --policy-arn "$POLICY_ARN"   --query 'Policy.DefaultVersionId' --output text)
aws iam get-policy-version --policy-arn "$POLICY_ARN" --version-id "$VERSION"   --query 'PolicyVersion.Document' --output json
```

Inspect the policy document to confirm the three actions and that the resource is exactly the table ARN, never `"*"`.

## Validation evidence and outcome

- **Confirmed:** The Day 99 challenge passed KodeKloud grading.
- **Expected resource checks:** DynamoDB `ACTIVE`, role and policy names exact, policy attached, output names exact, no pending Terraform changes.
- **Not claimed:** No real AWS CLI output or screenshots were uploaded for this documentation; the commands above are reproduction and verification instructions, not fabricated logs.

## Security and Git hygiene

- No credentials or private keys are included.
- Commit `.terraform.lock.hcl` when available in a real project; never commit `.terraform/` or `terraform.tfstate`.
- The `terraform.tfvars` in this demo contains only public lab names; treat other `.tfvars` files containing secrets as sensitive.
- Production: prefer a purpose-specific trusted service and attach the role through the appropriate resource. Audit effective policies and protect Terraform state.

## Lessons learned

- Difference between **IAM role trust policy** and **permissions policy**.
- DynamoDB table keys and on-demand billing.
- IAM resource ARN scoping and least-privilege design.
- Terraform resource dependency graph, `jsonencode()`, `.tfvars`, outputs, and state verification.
- Passing a lab requires the actual attachments and resources, not just syntactically valid HCL.
