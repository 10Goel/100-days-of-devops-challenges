# Day 96 — Provision an AWS EC2 Instance with Terraform and an RSA Key Pair

![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-7B42BC?logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white) ![KodeKloud](https://img.shields.io/badge/KodeKloud-Day%2096%20Passed-2EA44F)

## Challenge result

**Status: Passed.** Provision an EC2 instance named `datacenter-ec2`, use the precise Amazon Linux AMI `ami-0c101f26f147fa7fd` and `t2.micro`, create a **new RSA** key pair named `datacenter-kp`, and attach the **existing default** security group in `us-east-1`. The lab working directory is `/home/bob/terraform`; only `main.tf` is to be added, preserving the lab-provided `provider.tf`.

> **Scope and reproducibility:** This repository documents the configuration and checks for the completed KodeKloud task. It does not include the ephemeral lab AWS credentials, original `provider.tf`, Terraform state, private keys, or a live AWS instance.

## Architecture

```text
/home/bob/terraform/
├── provider.tf                  # supplied by lab, not reproduced
└── main.tf                      # Terraform configuration
           |
           +--> data.aws_vpc.default
           |        |
           |        +--> data.aws_security_group.default
           |
           +--> tls_private_key.datacenter_key (RSA 2048)
                    |
                    +--> aws_key_pair.datacenter_kp (public key only)
                              |
                              v
                    aws_instance.datacenter_ec2
                     AMI: ami-0c101f26f147fa7fd
                     Type: t2.micro
                     SG: default, in default VPC
                     Name: datacenter-ec2
```

## Requirements

| Setting | Required value |
|---|---|
| Region | `us-east-1` |
| Instance Name tag | `datacenter-ec2` |
| AMI ID | `ami-0c101f26f147fa7fd` |
| Instance type | `t2.micro` |
| New key-pair name and algorithm | `datacenter-kp`, RSA |
| Security group | Existing `default` security group |
| Working directory | `/home/bob/terraform` |
| New Terraform file | `main.tf` |

## Implementation

See [`main.tf`](main.tf). The configuration:

1. Reads the default VPC via `data "aws_vpc"`.
2. Locates the `default` security group **within that VPC**.
3. Generates RSA key material with `tls_private_key`.
4. Registers only its public key with EC2 using `aws_key_pair`.
5. Provisions `aws_instance` with the requested AMI, instance type, key name, security group, and AWS Name tag.

Terraform automatically derives ordering from references to the VPC, security group, and key pair. The lab's existing `provider.tf` configures the AWS region and authentication; `terraform init` downloads the additional TLS provider.

## Runbook

```bash
cd /home/bob/terraform
cat provider.tf                 # inspect only; don't overwrite
terraform init
terraform fmt main.tf
terraform validate
terraform plan
terraform apply                 # review the plan, then approve
terraform state list
terraform state show aws_instance.datacenter_ec2
terraform plan                  # expect no changes after convergence
```

A new workspace with this configuration should plan **three managed resources**: `tls_private_key`, `aws_key_pair`, and `aws_instance`. The VPC and security group are data sources, not newly managed infrastructure.

## Verification

```bash
aws ec2 describe-instances --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-ec2' \
            'Name=instance-state-name,Values=pending,running' \
  --query 'Reservations[].Instances[].{ID:InstanceId,AMI:ImageId,Type:InstanceType,Key:KeyName,State:State.Name}' \
  --output table

aws ec2 describe-key-pairs --region us-east-1 \
  --key-names datacenter-kp \
  --query 'KeyPairs[].{Name:KeyName,Type:KeyType}' --output table

aws ec2 describe-instances --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-ec2' \
  --query 'Reservations[].Instances[].SecurityGroups[].GroupName' \
  --output text
```

Check that the AMI, type, key name, and SG are exact; the final query should return `default`. A later `terraform plan` should ideally say **No changes**. The KodeKloud grader accepted the completed exercise.

## Security considerations

**Important:** `tls_private_key` saves the *private key* in Terraform state, even if its output is marked sensitive. Never commit state or private keys. In a production environment, generate/manage keys outside Terraform when possible, secure remote state with encryption and access controls, and prefer AWS Systems Manager Session Manager over broad SSH access. The task's default security group is reused without modifying its ingress rules.

## What I learned

- Terraform resource blocks versus data sources
- AWS VPC and default security group association
- RSA private/public key separation and EC2 key-pair registration
- HCL references and implicit resource dependencies
- Terraform init/plan/apply/state workflows and state sensitivity
- Instance verification through Terraform state and AWS CLI
