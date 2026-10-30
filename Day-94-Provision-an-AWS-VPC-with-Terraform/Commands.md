# Day 94 — Commands Reference

## 1. Inspect supplied lab files
```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf   # Inspect only; do not overwrite provided config
```

## 2. Create the requested file
```bash
cat > main.tf <<'EOF'
resource "aws_vpc" "devops_vpc" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "devops-vpc"
  }
}
EOF
```

## 3. Initialize and check
```bash
terraform init
terraform fmt main.tf
terraform validate
terraform plan
```
Expected before first apply: `Plan: 1 to add, 0 to change, 0 to destroy.` if no VPC is yet managed by this configuration.

## 4. Apply
```bash
terraform apply
# Inspect proposed actions, then type yes when prompted.
```

## 5. Verify state and resource attributes
```bash
terraform state list
terraform state show aws_vpc.devops_vpc
terraform plan
```
Successful convergence: `No changes. Your infrastructure matches the configuration.`

Optional AWS check:
```bash
aws ec2 describe-vpcs \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=devops-vpc' \
  --query 'Vpcs[].{VpcId:VpcId,CIDR:CidrBlock,State:State}' \
  --output table
```

## 6. Troubleshooting commands
```bash
terraform version
terraform providers
terraform validate
terraform plan
terraform state list
terraform state show aws_vpc.devops_vpc
aws sts get-caller-identity  # Only if lab credentials permit
```

| Symptom | Investigation |
|---|---|
| `No valid credential sources found` | Check lab-provided AWS authentication, not hardcoded keys |
| Region incorrect | Inspect `provider.tf` and AWS verification region |
| Resource label exists but Name tag wrong | Verify `tags = { Name = "devops-vpc" }` |
| Provider download fails | Check network/proxy access; retry `terraform init` |
| Plan shows unexpected destruction | **Do not apply**; review state, working directory, and plan |
| State says object exists but AWS doesn't | Investigate drift and credentials before changing state |

## 7. Useful commands for later study (not required for this challenge)
```bash
terraform plan -out=tfplan    # Saved binary plan (may contain secrets; do not commit)
terraform show tfplan
terraform output
terraform workspace list
terraform fmt -check
```

`terraform destroy` would delete managed infrastructure and was **not** part of the passed lab workflow. Use only in a disposable account with explicit approval and a reviewed plan.
