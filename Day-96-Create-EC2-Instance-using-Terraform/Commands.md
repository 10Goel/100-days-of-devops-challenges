# Day 96 — Command Reference

> **Context:** KodeKloud iac-server, `bob`, `/home/bob/terraform`, AWS `us-east-1`. The lab provides `provider.tf`. Do not publish any keys, state, or credentials.

## 1. Inspect the workspace

```bash
cd /home/bob/terraform
pwd
ls -la
cat provider.tf
terraform version
```

Inspect rather than overwrite provider settings. Add the repository's [`main.tf`](main.tf) to the lab workspace.

## 2. Format, initialize, validate

```bash
terraform init
terraform fmt main.tf
terraform validate
```

If necessary, inspect installed provider versions with:

```bash
terraform providers
```

## 3. Plan and deploy

```bash
terraform plan
terraform apply
# Review the resource changes and type yes if appropriate.
```

For a clean lab, the plan typically shows `3 to add, 0 to change, 0 to destroy`. Avoid applying unexpected destroys or replacements.

## 4. Inspect managed infrastructure

```bash
terraform state list
terraform state show aws_instance.datacenter_ec2
terraform state show aws_key_pair.datacenter_kp
terraform plan
```

**Never print or share `terraform state show tls_private_key.datacenter_key` or raw state files.** Those can contain private key material.

## 5. Verify in AWS

```bash
# Check instance attributes and status
aws ec2 describe-instances \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-ec2' \
            'Name=instance-state-name,Values=pending,running' \
  --query 'Reservations[].Instances[].{ID:InstanceId,AMI:ImageId,Type:InstanceType,Key:KeyName,State:State.Name}' \
  --output table

# Confirm the RSA key registration
aws ec2 describe-key-pairs \
  --region us-east-1 --key-names datacenter-kp \
  --query 'KeyPairs[].{Name:KeyName,Type:KeyType}' --output table

# Check attached security group: expected default
aws ec2 describe-instances \
  --region us-east-1 \
  --filters 'Name=tag:Name,Values=datacenter-ec2' \
  --query 'Reservations[].Instances[].SecurityGroups[].{Name:GroupName,Id:GroupId}' \
  --output table

# Optional: view default VPC ID
aws ec2 describe-vpcs \
  --region us-east-1 \
  --filters 'Name=isDefault,Values=true' \
  --query 'Vpcs[].VpcId' --output text
```

## 6. Troubleshooting commands

```bash
terraform validate
terraform providers
terraform plan
aws sts get-caller-identity
aws ec2 describe-vpcs --region us-east-1 --filters 'Name=isDefault,Values=true'
aws ec2 describe-key-pairs --region us-east-1 --key-names datacenter-kp
```

| Error | Diagnostic focus |
|---|---|
| `No matching EC2 VPC found` | Does this region/account have a default VPC? |
| `InvalidAMIID.NotFound` | Confirm fixed AMI ID and `us-east-1` region |
| `InvalidKeyPair.Duplicate` | Existing unmanaged `datacenter-kp`; inspect before changes |
| `No valid credential sources` | Use lab-supplied provider settings and credentials |
| `UnauthorizedOperation` | IAM access to EC2 actions |
| `Failed to query available provider packages` | Registry connectivity or provider availability |

**Lab verification:** Confirm correct instance, key, and SG; then click KodeKloud **Check**. Avoid `terraform destroy` unless explicitly instructed to clean up a disposable environment.
