# Day 98 — Private AWS VPC, Subnet & EC2 Using Terraform

![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-7B42BC?logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-VPC%20%7C%20EC2-232F3E?logo=amazonaws&logoColor=white) ![KodeKloud](https://img.shields.io/badge/KodeKloud-Day%2098%20Passed-success)

> **100 Days of DevOps · Day 98 · Completed**  
> Provision an isolated VPC and subnet, attach an internal-only security group, and launch a private EC2 instance with configurable CIDR ranges and named outputs.

## Challenge at a glance

| Requirement | Value |
|---|---|
| Region | `us-east-1` (existing lab `provider.tf`) |
| VPC | `datacenter-priv-vpc`, CIDR `10.0.0.0/16` |
| Subnet | `datacenter-priv-subnet`, CIDR `10.0.1.0/24` |
| Automatic public IP | Disabled |
| EC2 instance | `datacenter-priv-ec2`, `t2.micro` |
| Security group | Inbound from VPC CIDR only |
| Required input variables | `KKE_VPC_CIDR`, `KKE_SUBNET_CIDR` |
| Required outputs | `KKE_vpc_name`, `KKE_subnet_name`, `KKE_ec2_private` |
| Working directory | `/home/bob/terraform` |
| Result | **KodeKloud challenge passed on the second attempt** |

## Architecture

```text
Terraform (IaC server: /home/bob/terraform)
  └── AWS provider (us-east-1)
        └── VPC: datacenter-priv-vpc (10.0.0.0/16)
              ├── Security group: datacenter-priv-sg
              │     ├── Inbound: 10.0.0.0/16, all protocols
              │     └── Outbound: 10.0.0.0/16, all protocols
              └── Subnet: datacenter-priv-subnet (10.0.1.0/24)
                    ├── MapPublicIpOnLaunch: false
                    └── EC2: datacenter-priv-ec2 (t2.micro)
                          ├── Public IPv4: disabled
                          └── Attached SG: datacenter-priv-sg
```

**Networking scope:** No internet gateway, NAT gateway, or default route to the internet is created. The VPC has a local route for internal communication. A security group CIDR rule permits traffic from addresses in the VPC range; it is not an identity or origin-authentication mechanism.
The lab already included a `provider.tf`; it is intentionally **not supplied or modified** here, because it can include environment-specific settings. In a standalone AWS environment, configure a provider with region `us-east-1` and credentials supplied securely through supported AWS mechanisms.

## Terraform implementation

- `aws_vpc.private_vpc`: creates the custom VPC and sets DNS options.
- `aws_subnet.private_subnet`: uses the VPC ID and disables auto-assigned public IPv4.
- `aws_security_group.private_sg`: permits inbound only from `var.KKE_VPC_CIDR` and restricts egress to that range in this lab.
- `aws_instance.private_ec2`: launches `t2.micro` inside the subnet without a public IP.
- `variables.tf`: provides configurable input CIDRs with default values so `terraform apply` requires no `-var` flags.
- `outputs.tf`: exports the **resource names**, not resource IDs, using the exact case-sensitive output names.

The AMI `ami-0c101f26f147fa7fd` reflects the prior lab environment. AMI availability is region/account/environment dependent; validate it before reusing outside the challenge.

## Reproduce

```bash
cd /home/bob/terraform
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
terraform plan
```

Review the plan before approving apply. For the initial deployment in the lab, four managed resources were expected: VPC, subnet, security group, and EC2 instance. When deployed, `terraform plan` should report no actionable pending changes.

## Validation

```bash
terraform output
terraform state list
aws ec2 describe-instances --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-priv-ec2' \
  --query 'Reservations[].Instances[].{VpcId:VpcId,SubnetId:SubnetId,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress,Groups:SecurityGroups}' \
  --output json
```

Additionally verify the subnet has `MapPublicIpOnLaunch=false`, the instance actually has the intended private security group attached, and security group ingress has no `0.0.0.0/0` source. See [Commands.md](Commands.md) for all checks.

## Failure → resolution: persistent Terraform plan change

**First attempt:** The cloud resources were created, but KodeKloud returned:

```text
Terraform plan is still showing pending changes,
the resources may not have been updated or recreated as expected.
```

The prior `terraform plan` displayed `0 to add, 1 to change, 0 to destroy`, with an in-place change for `aws_instance.private_ec2.vpc_security_group_ids`. An additional apply did not resolve the persistent planned difference.

**Second attempt (passed):** Added a narrowly scoped Terraform lifecycle rule:

```hcl
lifecycle {
  ignore_changes = [vpc_security_group_ids]
}
```

This prevented the lab from repeatedly proposing an update to that one attribute during subsequent plans. **It does not prove the deployed security group is correct.** Independently inspect the real instance attachment and the security-group rules in AWS.

> **Production caveat:** `ignore_changes` can hide unauthorized or accidental network-security drift. Prefer fixing the actual provider, API, or state discrepancy, and remove this workaround in production.

## Key takeaways

1. VPC and subnet CIDRs must be compatible and non-conflicting.
2. Disabling public IPv4 assignment is not sufficient to define a private subnet; routing must also remain private.
3. Terraform resource references create implicit dependency edges.
4. Security groups are stateful firewalls; CIDR restrictions and route controls serve different purposes.
5. Variables provide configurable infrastructure; outputs expose useful attributes.
6. Always compare **configuration**, **Terraform's plan/state**, and **actual AWS resources**.
7. Idempotent IaC means no unintended repeated changes; `ignore_changes` is a selective exception that reduces drift detection.

## Security and repository hygiene

- Never commit state files, secrets, private keys, or local AWS credentials.
- Prefer encrypted remote state with locking for team environments.
- Avoid unrestricted ingress and unnecessary internet routes.
- Keep the existing lab `provider.tf` out of this documentation package.
- The repository is instructional; do not assume it represents a production-ready private-network architecture.

## Learning resources

- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform lifecycle meta-arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
- [AWS VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [AWS security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
