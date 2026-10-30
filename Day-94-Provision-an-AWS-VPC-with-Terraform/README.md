# Day 94 — Provision an AWS VPC with Terraform

![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-844FBA) ![AWS](https://img.shields.io/badge/AWS-VPC-FF9900) ![Challenge](https://img.shields.io/badge/KodeKloud-Day%2094%20Passed-success)

## Objective
Provision a virtual private cloud named **`devops-vpc`** in AWS region **`us-east-1`** using Terraform. The challenge requires creating **`/home/bob/terraform/main.tf`**, preserving the provided `provider.tf`, and using any valid IPv4 CIDR block.

## Architecture
```text
bob@iac-server
└── /home/bob/terraform/
    ├── provider.tf      # Existing lab provider configuration; not overwritten
    └── main.tf          # AWS VPC desired state
             │
             ▼
       Terraform AWS provider (us-east-1)
             │
             ▼
       VPC: devops-vpc (10.0.0.0/16)
```

## Task requirements
| Property | Value |
|---|---|
| Resource | `aws_vpc` |
| Name tag | `devops-vpc` |
| Region | `us-east-1` |
| IPv4 CIDR | `10.0.0.0/16` (chosen valid block) |
| Working directory | `/home/bob/terraform` |
| Required configuration | `main.tf` |

## Implementation
The `main.tf` file is included in this folder. The AWS provider was configured by the supplied lab's `provider.tf`; that provider configuration is intentionally not recreated here.

```hcl
resource "aws_vpc" "devops_vpc" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "devops-vpc"
  }
}
```

## Execution workflow
```bash
cd /home/bob/terraform
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply
terraform state list
terraform state show aws_vpc.devops_vpc
terraform plan
```
Review the plan before typing `yes` at the `terraform apply` prompt.

## Verification and outcome
The submitted lab displayed **challenge completed successfully**. Terraform's follow-up plan showed **`No changes. Your infrastructure matches the configuration.`**, indicating the managed VPC and declared configuration were in sync at verification time. The resource's Name tag was `devops-vpc`.

For optional cloud-side verification when credentials are available:
```bash
aws ec2 describe-vpcs --region us-east-1 \
  --filters 'Name=tag:Name,Values=devops-vpc' \
  --query 'Vpcs[].{VpcId:VpcId,CIDR:CidrBlock,State:State}' --output table
```

## Engineering takeaways
- A **resource block** defines desired infrastructure; `terraform apply` reconciles it with provider resources.
- Terraform's **resource label** (`devops_vpc`) is not the AWS **Name tag** (`devops-vpc`).
- **State** connects the configuration address to the remote VPC ID.
- A second `plan` with no changes is an important convergence check.
- This task provisions a VPC only; it does **not** create subnets, routing, internet gateways or compute instances.

> Security: never commit AWS credentials, `.terraform/`, `terraform.tfstate`, or plan files containing sensitive values to a public repository. Provider configuration and the original remote state are intentionally not included.
