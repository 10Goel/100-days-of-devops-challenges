# Day 95 — Commands and Verification

> Run these in the KodeKloud VS Code integrated terminal as `bob` on the IaC server. Do not commit credentials or Terraform state. The original lab session is ephemeral.

## 1. Inspect the working directory

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf  # Review; do not replace
terraform version
```

## 2. Inspect the default VPC (if AWS CLI is available)

```bash
aws ec2 describe-vpcs \
  --region us-east-1 \
  --filters 'Name=isDefault,Values=true' \
  --query 'Vpcs[].{VpcId:VpcId,CIDR:CidrBlock}' \
  --output table
```

Terraform's `data.aws_vpc.default` performs the lookup even if the AWS CLI is unavailable. It fails if no default VPC is present.

## 3. Write the configuration

Place the repository's [`main.tf`](main.tf) at `/home/bob/terraform/main.tf`. Keep supplied `provider.tf` unchanged. The only lab-provided file to create is `main.tf`.

## 4. Terraform lifecycle

```bash
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply
```

Review the plan before approving. With an empty state and the reference configuration, typically **four AWS resources** are added: one security group, two ingress rules, one egress rule; the existing VPC is only read.

## 5. State inspection

```bash
terraform state list
terraform state show aws_security_group.datacenter_sg
terraform state show aws_vpc_security_group_ingress_rule.http
terraform state show aws_vpc_security_group_ingress_rule.ssh
terraform plan
```

Expected resource addresses:

```text
aws_security_group.datacenter_sg
aws_vpc_security_group_ingress_rule.http
aws_vpc_security_group_ingress_rule.ssh
aws_vpc_security_group_egress_rule.all_outbound
```

A second `terraform plan` should normally report `No changes. Your infrastructure matches the configuration.`

## 6. Validate cloud configuration

```bash
# Default VPC ID
aws ec2 describe-vpcs \
  --region us-east-1 \
  --filters 'Name=isDefault,Values=true' \
  --query 'Vpcs[0].VpcId' \
  --output text

# Security group properties and inbound permissions
aws ec2 describe-security-groups \
  --region us-east-1 \
  --filters 'Name=group-name,Values=datacenter-sg' \
  --query 'SecurityGroups[].{Name:GroupName,Description:Description,VpcId:VpcId,Ingress:IpPermissions}' \
  --output json
```

Check that `Description` equals `Security group for Nautilus App Servers` and ingress contains exactly TCP 80 and TCP 22 from `0.0.0.0/0`. If there are several groups of the same name in different VPCs, compare the VPC ID.

## 7. Troubleshooting cheatsheet

```bash
terraform providers
terraform validate
terraform plan
terraform state list
aws sts get-caller-identity
```

| Symptom | Check |
|---|---|
| `no matching EC2 VPC found` | Region and default VPC existence |
| `Unsupported resource type` | AWS provider version supporting newer standalone rules |
| `InvalidGroup.Duplicate` | Group already exists; avoid creating another blindly |
| `InvalidPermission.Duplicate` | Duplicate ingress rule already present |
| `UnauthorizedOperation` | Current role's EC2 permissions |
| `No valid credential sources` | Lab's existing provider and credential setup |
| Unexpected destroy | Stop and investigate state/configuration mismatch |

Do **not** use `terraform destroy` simply to clear errors. Protect any shared infrastructure and inspect `plan` before `apply`.
