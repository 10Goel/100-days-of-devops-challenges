# Day 99 — Commands Reference

> Run these in the KodeKloud VS Code integrated terminal as `bob`. The lab supplies `provider.tf`; preserve it. This guide assumes all project files from this repository are placed in `/home/bob/terraform`.

## 1. Inspect the project

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf
```

## 2. Create the configuration files

Place/copy `main.tf`, `variables.tf`, `outputs.tf`, and `terraform.tfvars` into the working directory. Check the exact names and values:

```bash
ls -l main.tf variables.tf outputs.tf terraform.tfvars
cat terraform.tfvars
```

Expected values:

```hcl
KKE_TABLE_NAME  = "nautilus-table"
KKE_ROLE_NAME   = "nautilus-role"
KKE_POLICY_NAME = "nautilus-readonly-policy"
```

## 3. Terraform lifecycle

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
terraform state list
terraform plan
```

On a fresh lab account, the first plan typically shows `4 to add`; after apply, the final plan should show `No changes. Your infrastructure matches the configuration.` The table may briefly be `CREATING` before becoming `ACTIVE`.

## 4. Query resource state

```bash
terraform state show aws_dynamodb_table.nautilus_table
terraform state show aws_iam_role.nautilus_role
terraform state show aws_iam_policy.nautilus_readonly
terraform state show aws_iam_role_policy_attachment.nautilus_attach
```

## 5. Verify DynamoDB

```bash
aws dynamodb describe-table   --region us-east-1   --table-name nautilus-table   --query 'Table.{Name:TableName,Status:TableStatus,Arn:TableArn,KeySchema:KeySchema}'   --output json
```

Expected: Name `nautilus-table`, `TableStatus` `ACTIVE`, partition key `id`.

## 6. Verify IAM role and trust policy

```bash
aws iam get-role   --role-name nautilus-role   --query 'Role.{Name:RoleName,Arn:Arn,Trust:AssumeRolePolicyDocument}'   --output json
```

Expected trusted service principal: `ec2.amazonaws.com`; allowed trust action: `sts:AssumeRole`.

## 7. Verify role-policy attachment

```bash
aws iam list-attached-role-policies   --role-name nautilus-role   --output table
```

Expected policy name: `nautilus-readonly-policy`.

## 8. Inspect exact policy JSON

```bash
POLICY_ARN=$(aws iam list-policies --scope Local   --query "Policies[?PolicyName=='nautilus-readonly-policy'].Arn | [0]"   --output text)

echo "$POLICY_ARN"

VERSION=$(aws iam get-policy   --policy-arn "$POLICY_ARN"   --query 'Policy.DefaultVersionId'   --output text)

aws iam get-policy-version   --policy-arn "$POLICY_ARN"   --version-id "$VERSION"   --query 'PolicyVersion.Document'   --output json
```

Verify exactly three allowed actions and `Resource` equals the DynamoDB table ARN. An AWS IAM simulation API may not be implemented in the KodeKloud sandbox; lack of simulation support does not itself prove the policy is wrong.

## 9. Troubleshooting decision table

| Symptom | Diagnostic command | Next step |
|---|---|---|
| Invalid HCL | `terraform validate` | Correct syntax / required attributes |
| Undefined variable | `terraform plan` | Check names in `variables.tf` / `terraform.tfvars` |
| Table already exists | `aws dynamodb describe-table --table-name nautilus-table --region us-east-1` | Inspect existing table/state; don't blindly delete |
| IAM role already exists | `aws iam get-role --role-name nautilus-role` | Inspect role/state; determine whether import is appropriate |
| Policy missing from role | `aws iam list-attached-role-policies --role-name nautilus-role` | Check attachment block and `terraform apply` |
| AWS authorization denied | Review error action | Check lab AWS account permissions |
| Pending plan changes | `terraform plan` | Resolve exact drift before checking |

## 10. Safe repetition

```bash
terraform fmt -check
terraform validate
terraform plan
terraform output -json
```

Avoid `terraform destroy` in the scored lab. Outside the lab, destroy only when approved and after reviewing the impact.
