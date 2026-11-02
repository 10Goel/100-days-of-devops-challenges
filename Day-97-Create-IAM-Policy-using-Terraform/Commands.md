# Day 97 — Commands and Verification Runbook

> Run on the KodeKloud IaC server as `bob`. The lab provides `provider.tf` and authentication. **Do not copy credentials into GitHub.**

## 1. Inspect the project

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf  # inspect locally; never publish credentials
```

Confirm the existing provider points at the lab region `us-east-1`. IAM itself is global.

## 2. Create the required file

```bash
vi main.tf
```

Paste the Terraform configuration from [`main.tf`](main.tf). Save using `Esc`, `:wq`, and Enter.

## 3. Terraform lifecycle

```bash
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply
```

A fresh apply normally yields `Plan: 1 to add, 0 to change, 0 to destroy`. Enter `yes` when the plan matches the requirements.

## 4. State-based checks

```bash
terraform state list
terraform state show aws_iam_policy.siva
terraform plan
```

Expect the state list to include `aws_iam_policy.siva` and the final plan to say:

```text
No changes. Your infrastructure matches the configuration.
```

## 5. AWS CLI checks (if supported)

```bash
aws sts get-caller-identity

POLICY_ARN=$(aws iam list-policies   --scope Local   --query "Policies[?PolicyName=='iampolicy_siva'].Arn | [0]"   --output text)

echo "$POLICY_ARN"
aws iam get-policy --policy-arn "$POLICY_ARN"

VERSION=$(aws iam get-policy   --policy-arn "$POLICY_ARN"   --query 'Policy.DefaultVersionId' --output text)

aws iam get-policy-version   --policy-arn "$POLICY_ARN"   --version-id "$VERSION"   --query 'PolicyVersion.Document' --output json
```

Expected policy fields: `Version = 2012-10-17`, `Effect = Allow`, `Action = ec2:Describe*`, `Resource = *`.

## 6. Optional IAM simulation (not supported by this lab)

If a real AWS account supports the operation **and your identity has IAM simulation permission**, `simulate-custom-policy` can evaluate example actions. In the Day 97 lab this failed with:

```text
An error occurred (InternalFailure) when calling the SimulateCustomPolicy operation:
The simulate_custom_policy action has not been implemented
```

This indicates **the lab endpoint does not implement the simulation API**. It does not mean the `aws_iam_policy` resource failed. Use state, the policy document, and lab verification instead. On real AWS, supplement policy simulation with actual role/user testing.

## 7. Diagnostic commands

```bash
terraform version
terraform providers
terraform validate
terraform plan
terraform state list
```

For API permission errors, inspect `aws sts get-caller-identity` and the **existing lab provider settings**. For duplicate-name errors, check for an existing IAM policy and whether it is tracked in Terraform state; avoid deleting resources indiscriminately.

## 8. Git hygiene

```bash
git status --short
git check-ignore .terraform/ terraform.tfstate
```

Never commit `terraform.tfstate`, credentials, local secret variables, or provider caches. Commit `.terraform.lock.hcl` for real reproducible Terraform projects when it has been generated and reviewed.
