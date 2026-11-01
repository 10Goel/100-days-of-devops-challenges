# Day 96 — Deep Notes: Terraform, EC2, RSA Keys, and AWS Networking

## 1. The learning objective

Day 96 extends Infrastructure as Code from a VPC (Day 94) and a security group (Day 95) to an actual **compute instance**. The desired state includes the OS image, instance size, login key registration, security-group association, and resource tagging. Terraform evaluates configuration, constructs a dependency graph, calls the AWS provider, and records resource identity in state.

## 2. Terraform's core architecture

- **Terraform configuration (HCL):** declares desired resources and values.
- **Terraform CLI:** reads files, loads providers, creates plans, and applies changes.
- **Provider:** plugin that maps resource/data-source operations to provider APIs; the AWS provider manages EC2 and VPC objects.
- **Resource:** infrastructure Terraform creates and manages (`aws_instance`, `aws_key_pair`).
- **Data source:** read-only query for existing infrastructure (`data.aws_vpc.default`, `data.aws_security_group.default`).
- **State:** persistent mapping of Terraform resource addresses to remote objects and selected attributes; may contain secrets.
- **Dependency graph:** derives creation/refresh ordering from references. Explicit `depends_on` is needed only for dependencies Terraform cannot infer.

`resource "aws_instance" "datacenter_ec2"` has resource **type** `aws_instance` and local **label** `datacenter_ec2`. Neither is the AWS-visible Name tag. The AWS console Name comes from `tags = { Name = "datacenter-ec2" }`.

## 3. Terraform execution lifecycle

| Command | Purpose | Common misconception |
|---|---|---|
| `terraform init` | Initialize backend, install provider plugins/modules | Does not create EC2 itself |
| `terraform fmt` | Reformat HCL | Does not prove configuration is deployable |
| `terraform validate` | Validate Terraform configuration | Does not prove the AMI exists or permissions are sufficient |
| `terraform plan` | Preview differences between configuration, state and discovered remote state | Is not a guarantee the apply will succeed |
| `terraform apply` | Execute approved changes, update state | Can partially succeed before an error |
| `terraform state list/show` | Inspect state addresses/attributes | Can expose sensitive values; avoid showing private key resource |

**Idempotency:** On unchanged configuration and infrastructure, an additional plan should show no changes. External changes create **drift** that can appear in later plans.

## 4. Understanding Amazon EC2

EC2 is virtual compute hosted by AWS. Key concepts:

- **AMI:** a region-scoped machine image containing an OS and launch configuration. The lab requires exact ID `ami-0c101f26f147fa7fd` in `us-east-1`.
- **Instance type (`t2.micro`):** defines a compute/memory profile. Compatibility and availability depend on image and region.
- **Name tag:** metadata used for identification; not an intrinsic EC2 instance name or unique constraint.
- **Instance ID:** AWS-generated identifier (`i-...`) unique to the instance in its regional account context.
- **State:** e.g., `pending`, `running`, `stopping`, `stopped`, `terminated`; an `apply` success is distinct from an application readiness check.
- **Networking:** instance runs in a subnet; VPC security groups act as stateful network-access controls.

The lab doesn't ask for a custom subnet, Elastic IP, public IP, user data, or web service. Do not add unrelated resources.

## 5. Default VPC and default security group

A **default VPC** is an AWS-provisioned VPC in a region (where available). The task requires the instance's **default security group**, not `datacenter-sg` from Day 95. `data "aws_vpc" "default" { default = true }` fetches the VPC; a security-group data source with `name = "default"` and its `vpc_id` ensures the correct SG.

An SG is **stateful**: response traffic to allowed connections is permitted without separate reverse-direction rules. Default SG rules are AWS-managed defaults and may not permit SSH from an external client. **Attaching a key pair does not open port 22.**

Terraform uses `vpc_security_group_ids`, not `security_groups` by name, for this VPC association; IDs avoid ambiguity and work reliably with VPC-scoped SGs.

## 6. RSA key pairs and authentication

RSA is an asymmetric algorithm: a mathematically related **private key** (kept secret) and **public key** (distributable). An EC2 key pair is an AWS registration of public-key material; the private key is not uploaded to AWS.

```hcl
resource "tls_private_key" "datacenter_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}

resource "aws_key_pair" "datacenter_kp" {
  key_name   = "datacenter-kp"
  public_key = tls_private_key.datacenter_key.public_key_openssh
}
```

The `tls_private_key` resource generates both components in Terraform's process. `public_key_openssh` is supplied to AWS. The EC2 instance references the created key by `key_name`, making an implicit dependency.

**Crucial security warning:** The private key is stored in **Terraform state**. `sensitive = true` or output redaction does **not** encrypt the state itself. Protect state with encrypted remote storage, least-privilege access, state locking where supported, and careful audit controls. Never commit `terraform.tfstate`, backups, or keys to GitHub. For production, consider generating/holding key material outside Terraform or using AWS Systems Manager Session Manager instead of SSH key management.

## 7. Understanding the HCL for Day 96

```hcl
data "aws_vpc" "default" { default = true }

data "aws_security_group" "default" {
  name   = "default"
  vpc_id = data.aws_vpc.default.id
}

resource "aws_instance" "datacenter_ec2" {
  ami                    = "ami-0c101f26f147fa7fd"
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.datacenter_kp.key_name
  vpc_security_group_ids = [data.aws_security_group.default.id]
  tags                   = { Name = "datacenter-ec2" }
}
```

A reference like `aws_key_pair.datacenter_kp.key_name` returns an attribute of another resource, which causes Terraform to order actions automatically. `data.aws_security_group.default.id` reads an existing group's ID. Terraform handles provider API calls and tracks the AWS-generated instance ID in state.

## 8. AWS vs Terraform naming

| Layer | Example | What it identifies |
|---|---|---|
| Terraform resource address | `aws_instance.datacenter_ec2` | Resource within Terraform state |
| AWS Name tag | `datacenter-ec2` | Display label in console/API tags |
| AWS Instance ID | `i-0123...` | Actual provisioned EC2 instance |
| AWS key-pair name | `datacenter-kp` | Registered EC2 key-pair object |
| AWS SG ID | `sg-0123...` | Security group association |

## 9. State, drift and repeatability

Terraform state helps connect HCL definitions with existing cloud resources. If someone edits the SG association manually, the next plan may detect drift and propose a correction. An `apply` that is interrupted can create a partially complete environment: inspect AWS resources and state before retrying, rather than blindly creating duplicates or deleting anything.

`.terraform/` is a provider/module cache. `terraform.tfstate` and its backups are **sensitive**. `.terraform.lock.hcl` is normally *committed*, since it pins selected provider versions and checksums; do not confuse it with state.

## 10. Common errors and their reasoning

| Symptom | Root cause to investigate |
|---|---|
| `No matching EC2 VPC found` | Default VPC absent or incorrect AWS region |
| `No matching EC2 Security Group found` | Default SG data-source filter incorrect / wrong VPC |
| `InvalidAMIID.NotFound` | Wrong region or unavailable lab AMI |
| `InvalidKeyPair.Duplicate` | AWS already has a key pair of that name, but Terraform isn't managing it |
| `Unsupported resource type tls_private_key` | TLS provider initialization/version problem |
| `UnauthorizedOperation` | Insufficient IAM permission |
| EC2 creation succeeds but cannot SSH | Inbound SG rules, route/public IP, key possession, or user mismatch |
| Repeated replacement plan | A ForceNew argument changed, or state drift/import mismatch |

Do not change fixed challenge values to bypass errors. Check provider region, actual AWS resources, and state first.

## 11. Production considerations beyond the lab

- Prefer remote encrypted state with access controls, lock coordination, versioning and backups.
- Apply least-privilege IAM for Terraform execution roles.
- Review plans in CI and require approval before production applies.
- Scope SSH/management access to trusted networks; don't open port 22 globally.
- Consider Session Manager for shell access without SSH ingress.
- Make reusable modules for compute instances after mastering resource/data-source relationships.
- Tag instances consistently for cost allocation and operations.
- Use tested images and control image updates through a release process.
- Avoid generated private keys in state when possible.

## 12. Interview questions and short answers

1. **What is IaC?** Infrastructure represented as version-controlled declarative or imperative code.
2. **What does `terraform init` do?** Initializes the working directory/backend and resolves providers/modules.
3. **Difference between resource and data source?** Resource manages an object; data source queries an existing one.
4. **Why use `data.aws_vpc.default`?** To reference the default VPC instead of creating or hardcoding one.
5. **Why filter the default SG by VPC ID?** SG names aren't universally unique across VPCs.
6. **What is an AMI?** AWS machine image defining the OS/root volume launch basis.
7. **What is an EC2 instance type?** Compute-memory/network capabilities/profile.
8. **What is the `Name` tag?** Metadata display label, not the EC2 instance ID.
9. **What is a key pair?** Public key registered with EC2 and corresponding private key for authentication.
10. **Does AWS receive the RSA private key here?** No, only the OpenSSH public key is registered.
11. **Where is the TLS-generated private key stored?** In Terraform state; protect it.
12. **Does `sensitive` encrypt state?** No, it primarily redacts ordinary CLI/UI display.
13. **How does Terraform infer resource dependencies?** Via expressions referencing other resources' attributes.
14. **When is `depends_on` needed?** When a dependency exists that Terraform can't infer from references.
15. **What does `terraform plan` do?** Previews proposed changes; doesn't apply them.
16. **What does `No changes` mean?** Terraform currently detects no changes necessary to reach declared state.
17. **Does default SG guarantee SSH access?** No; SG rules and connectivity still matter.
18. **How do you verify the deployed instance?** Check Terraform state and AWS CLI's instance, key and SG fields.
19. **What is drift?** Remote infrastructure has changed outside the declared Terraform configuration.
20. **What belongs in Git?** HCL modules/config and lockfile; not secrets, private keys, state or provider caches.

## 13. Further reading

- Terraform language: https://developer.hashicorp.com/terraform/language
- Terraform state: https://developer.hashicorp.com/terraform/language/state
- AWS EC2 key pairs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
- AWS EC2 security groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html
- Terraform TLS provider: https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/private_key
