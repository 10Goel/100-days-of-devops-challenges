# Day 98 — Commands and Verification Runbook

> Execute in the KodeKloud **IaC server** terminal. Do not publish live AWS credentials, full Terraform state, or SSH keys.

## 1. Inspect project and provider

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf
```

Inspect but do not overwrite the preconfigured `provider.tf`.

## 2. Create the configuration files

Copy `main.tf`, `variables.tf`, and `outputs.tf` from this repository into `/home/bob/terraform/`. The filenames and `KKE_*` input/output names are case-sensitive.

```bash
ls -l main.tf variables.tf outputs.tf provider.tf
terraform fmt -check
```

## 3. Initialize, validate, review

```bash
terraform init
terraform fmt
terraform validate
terraform plan
```

For a fresh lab environment, expect `4 to add, 0 to change, 0 to destroy`. Do not apply plans with unexpected deletions or changes.

## 4. Provision

```bash
terraform apply
# Type yes only after inspecting the plan.
```

## 5. Check outputs and state

```bash
terraform output
terraform output -raw KKE_vpc_name
terraform output -raw KKE_subnet_name
terraform output -raw KKE_ec2_private
terraform state list
```

Expected output values:

```text
KKE_vpc_name      = "datacenter-priv-vpc"
KKE_subnet_name   = "datacenter-priv-subnet"
KKE_ec2_private   = "datacenter-priv-ec2"
```

## 6. Verify the VPC

```bash
aws ec2 describe-vpcs \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-priv-vpc' \
  --query 'Vpcs[].{ID:VpcId,CIDR:CidrBlock,State:State}' \
  --output table
```

Expected CIDR `10.0.0.0/16`.

## 7. Verify the subnet

```bash
aws ec2 describe-subnets \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-priv-subnet' \
  --query 'Subnets[].{ID:SubnetId,VpcId:VpcId,CIDR:CidrBlock,AutoPublicIP:MapPublicIpOnLaunch}' \
  --output table
```

Expect `10.0.1.0/24`, `AutoPublicIP=False`, and VPC ID matching the new VPC.

## 8. Verify the EC2 instance

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-priv-ec2' \
  --query 'Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,VpcId:VpcId,SubnetId:SubnetId,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress,State:State.Name,SecurityGroups:SecurityGroups}' \
  --output json
```

Expect `t2.micro`, `PublicIP: null`, the desired subnet/VPC, and a group with `GroupName: datacenter-priv-sg`.

## 9. Verify inbound/outbound rules

```bash
aws ec2 describe-security-groups \
  --region us-east-1 \
  --filters 'Name=group-name,Values=datacenter-priv-sg' \
  --query 'SecurityGroups[].{VpcId:VpcId,Ingress:IpPermissions,Egress:IpPermissionsEgress}' \
  --output json
```

The ingress rule must allow only the VPC CIDR `10.0.0.0/16`; there should be no `0.0.0.0/0` ingress rule. The shown lab configuration also limits egress to the same CIDR.

## 10. Inspect routes

```bash
aws ec2 describe-route-tables \
  --region us-east-1 \
  --filters 'Name=vpc-id,Values='"$(aws ec2 describe-vpcs --region us-east-1 --filters Name=tag:Name,Values=datacenter-priv-vpc --query 'Vpcs[0].VpcId' --output text)" \
  --query 'RouteTables[].Routes[]' \
  --output json
```

For this private lab VPC, no route to `0.0.0.0/0` via an Internet Gateway should be configured.

## 11. Final plan (critical KodeKloud check)

```bash
terraform plan
```

Required final state: **No changes** and actual AWS resources verified independently.

## 12. Troubleshooting repeated security-group updates

```bash
terraform plan
terraform state show aws_instance.private_ec2
aws ec2 describe-instances --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-priv-ec2' \
  --query 'Reservations[].Instances[].SecurityGroups' --output json
```

If the plan repeatedly proposes updates to `vpc_security_group_ids`, investigate actual AWS attachment and provider behavior. The challenge's passing workaround used:

```hcl
lifecycle {
  ignore_changes = [vpc_security_group_ids]
}
```

This suppresses drift detection for that attribute; verify actual security-group attachment independently. **Do not reflexively apply repeated plans or use the workaround as a production default.**

## 13. Additional diagnostics

```bash
terraform version
terraform providers
terraform validate
terraform state list
aws ec2 describe-images --region us-east-1 \
  --image-ids ami-0c101f26f147fa7fd \
  --query 'Images[].{ID:ImageId,State:State,Architecture:Architecture}' \
  --output table
```

`terraform destroy` is intentionally not part of the lab execution sequence. In a separate disposable environment, destruction should only be performed after reviewing its plan and confirming ownership of every affected resource.
