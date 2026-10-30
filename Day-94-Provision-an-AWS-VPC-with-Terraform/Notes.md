# Day 94 — Terraform Fundamentals to Advanced Study Notes

> **Theme:** Infrastructure as Code (IaC), Terraform workflow, the AWS provider, resource lifecycle, state, networking and practical troubleshooting.

## 1. What is Infrastructure as Code?
Infrastructure as Code (IaC) expresses infrastructure in version-controlled configuration files rather than relying on manual console operations. A team can review and reproduce cloud resources through automated workflows.

- **Declarative**: specify desired outcome (`aws_vpc` should exist); Terraform works out provider actions.
- **Imperative**: specify ordered operations (`create`, `configure`, etc.).
- **Idempotency / convergence**: applying unchanged desired configuration should result in no further infrastructure changes.
- **Drift**: real infrastructure changes outside Terraform, so actual state no longer matches configuration.

## 2. Terraform terminology
| Term | Meaning |
|---|---|
| Terraform CLI | Local tool that interprets HCL configuration, creates plans and applies changes |
| Provider | Plugin exposing a service's resources/data sources, e.g. AWS |
| Resource | Object Terraform manages, e.g. `aws_vpc` |
| Data source | Reads existing infrastructure without taking ownership of its lifecycle |
| Module | Reusable package of Terraform configuration; current directory is the root module |
| Root module | Configuration in the directory where Terraform runs |
| Child module | Reusable module called by another module |
| State | Map of Terraform resource addresses to remote objects and tracked attributes |
| Backend | Where state is stored (local by default; can be remote) |
| Plan | Proposed changes before execution |
| Apply | Executes the plan and updates state |
| Output | Exposes selected values from a root or child module |
| Variable | Parameterizes a configuration |
| Locals | Reusable derived expressions inside a module |
| Resource address | Identifier such as `aws_vpc.devops_vpc` |
| Drift | External changes to managed objects |
| Lock file | `.terraform.lock.hcl` pins selected provider versions/checksums |

## 3. Terraform architecture and execution
```text
main.tf + provider.tf + other *.tf files
                   │
                   ▼
             Terraform CLI
             ├─ init: prepare providers/backend
             ├─ validate: configuration consistency
             ├─ plan: refresh/read remote objects; calculate proposed changes
             └─ apply: provider API actions and state updates
                   │
                   ▼
          AWS Provider → AWS EC2/VPC API
                   │
                   ▼
          AWS VPC and Terraform state
```
All `.tf` files in a directory are combined into a single root module; their filenames typically organize code rather than dictate execution order.

### Core workflow
1. `terraform init`: initialize backend/modules and install provider dependencies.
2. `terraform fmt`: format HCL consistently.
3. `terraform validate`: check syntax and internal consistency, **not** remote AWS permissions or resource availability.
4. `terraform plan`: compare desired configuration and remote/state information and preview create/update/delete operations.
5. `terraform apply`: provision or alter infrastructure. Inspect plan before approval.
6. `terraform state list/show` and a fresh `terraform plan`: verify managed resources and convergence.

## 4. HCL basics
HashiCorp Configuration Language (HCL) is Terraform's declarative language. Blocks have a type, labels, and arguments.

```hcl
resource "aws_vpc" "devops_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "devops-vpc"
  }
}
```
- `resource`: declares a managed infrastructure object.
- `aws_vpc`: AWS provider resource type.
- `devops_vpc`: local label, used by references and state; not the cloud resource name.
- `cidr_block`: argument configuring the VPC network range.
- `tags.Name`: visible AWS Name tag, not an intrinsic VPC `name` argument.

### Types and expressions
- Primitive types: `string`, `number`, `bool`.
- Collections: `list`, `set`, `map`; structured `object` and `tuple`.
- References: `aws_vpc.devops_vpc.id`, `var.environment`, `local.common_tags`.
- Interpolation: expressions within strings are possible, but direct references are preferable when the whole value is an expression.
- Comments: `#`, `//`, and `/* ... */`.

## 5. Provider fundamentals
The AWS provider communicates with AWS APIs using credentials obtained from the lab/environment or configured identity. A provider's **region** selects where regional resources are created.

For illustration (the supplied lab's `provider.tf` was retained):
```hcl
provider "aws" {
  region = "us-east-1"
}
```
Provider version constraints belong in `terraform.required_providers`, e.g. a tested major/minor version range; the lock file records the chosen version. Never put AWS access keys in committed Terraform source.

## 6. AWS VPC deep dive
A Virtual Private Cloud (VPC) is a logically isolated IPv4/IPv6 network in an AWS region. VPCs are regional, while subnets live in availability zones.

### CIDR notation
`10.0.0.0/16` means that the first 16 bits identify the network, leaving 16 bits for addresses: `2^(32-16) = 65,536` IPv4 addresses in the block. That is the **VPC CIDR's total address count**; usable addresses per subnet are smaller because AWS reserves five IP addresses in each IPv4 subnet.

Common private IPv4 ranges (RFC 1918):
- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

Plan non-overlapping CIDR blocks when you expect VPC peering, transit gateways, VPNs, or connections to on-premises networks.

### DNS settings
- `enable_dns_support = true`: enables DNS resolution support in the VPC.
- `enable_dns_hostnames = true`: enables public DNS hostname assignment for qualifying instances, subject to further conditions.
- `instance_tenancy = "default"`: generally uses shared tenancy; dedicated tenancy can cost more.

### VPC alone is not a deployable website
An AWS VPC resource does not automatically provision application subnets, Internet Gateways, NAT Gateways, route table associations, security groups for a workload, or EC2 instances. These are separate resources.

Typical public-subnet architecture:
```text
VPC 10.0.0.0/16
 ├─ Public subnet 10.0.1.0/24
 │   ├─ route table → Internet Gateway (0.0.0.0/0)
 │   └─ EC2 instance + security group
 └─ Private subnet 10.0.2.0/24
     └─ optional NAT Gateway egress via routing
```

## 7. Terraform state — essential for DevOps
State maps `aws_vpc.devops_vpc` to the remote VPC ID and tracked attributes. Its purpose is to decide which actual object corresponds to your configuration.

### State commands
```bash
terraform state list
terraform state show aws_vpc.devops_vpc
terraform plan
```
State files can contain sensitive infrastructure data; do **not** commit `terraform.tfstate` or backups. For teams, use a secured remote backend and locking where supported; control access, enable encryption and versioning, and establish recovery procedures. Avoid manually editing state JSON.

### Lock file vs state file
- `.terraform.lock.hcl`: dependency selections (often **commit** to Git).
- `terraform.tfstate`: resource tracking and potentially secrets (**do not commit**).
- `.terraform/`: downloaded provider/module working files (**do not commit**).

## 8. Resource graph, dependencies, lifecycle
Terraform derives dependencies from references. For example:
```hcl
resource "aws_subnet" "example" {
  vpc_id     = aws_vpc.devops_vpc.id
  cidr_block = "10.0.1.0/24"
}
```
This subnet automatically depends on the VPC. `depends_on` is for hidden dependencies that Terraform cannot infer.

Lifecycle-related features worth learning:
- `create_before_destroy`: try to create a replacement before removing the old object (not always supported by constraints).
- `prevent_destroy`: guard against destruction in some plans; not a substitute for security policies.
- `ignore_changes`: suppress changes for designated attributes; can hide drift if used carelessly.
- `replace_triggered_by`: replace when tracked dependencies change.

## 9. Variables, locals, outputs and modules
Rather than hardcoding values in a reusable setup:
```hcl
variable "vpc_cidr" {
  type        = string
  description = "IPv4 CIDR for the VPC"
  default     = "10.0.0.0/16"
}

locals {
  project = "devops"
}

output "vpc_id" {
  value = aws_vpc.devops_vpc.id
}
```
A module packages resources, input variables, and outputs for reuse. Day 94 intentionally uses a simple root module to satisfy the lab's exact file instructions; these examples are conceptual extensions, not extra files to create for the lab.

## 10. Terraform vs Ansible vs CloudFormation
| Tool | Typical role |
|---|---|
| Terraform | Provision cloud resources across provider APIs |
| Ansible | Configure OS/services, deploy files/apps, orchestrate operational tasks |
| AWS CloudFormation | AWS-native declarative infrastructure definitions |

They can complement each other: Terraform provisions EC2/networking, then Ansible configures software, with care to manage secrets and boundaries between tools.

## 11. Terraform with CI/CD
Production workflow example:
```text
Pull request → fmt -check → validate → static/security checks → plan
           → review/approval → apply from protected pipeline
           → verify desired state and monitor drift
```
Avoid concurrent applies against the same state; use state locking, least-privilege IAM, protected branches, secret managers and review gates. Treat plan output as potentially sensitive.

## 12. Common failures and troubleshooting
| Symptom | Explanation / action |
|---|---|
| Provider authentication failure | Validate environment/role credentials; never add secrets to source |
| Wrong AWS region | Inspect provider config and `aws ec2 describe-vpcs --region ...` |
| VPC tag mismatch | Distinguish local label from `tags.Name` |
| Duplicate resource definition | Search all `.tf` files in same module for duplicate names |
| Unexpected deletion in plan | Stop and inspect state, module path, resources and configuration |
| Resource not found but state entry exists | Investigate remote drift and correct account/region |
| `terraform init` provider download error | Check DNS/network/proxy and provider registry access |
| VPC CIDR rejected | Check AWS VPC CIDR restrictions and overlap constraints |
| `terraform validate` succeeds but apply fails | Syntax correctness is not proof of IAM authorization or cloud-side validity |

## 13. Security and Git hygiene
Example `.gitignore` patterns for a real Terraform project:
```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
*.tfvars
!example.tfvars
```
Review variable files before committing; some may be safe examples, but real secrets must not be checked in. A team generally commits vetted `.tf` files and `.terraform.lock.hcl` and uses an encrypted remote backend for state.

## 14. Day 94 verified results
The lab confirmed the AWS VPC task passed; the terminal showed a follow-up `terraform plan` with **no changes** and the VPC Name tag set to **`devops-vpc`**. That is evidence of the tested desired state at the time of completion, not proof that the environment will remain unchanged indefinitely.

## 15. Interview questions and short answers
1. **What is a provider?** Plugin that lets Terraform interact with platform APIs and exposes resource/data source schemas.
2. **Why use state?** To map configuration resource addresses to actual managed objects and support change planning.
3. **What does `terraform init` do?** Initializes backend, modules and provider dependencies.
4. **Plan vs apply?** Plan previews intended actions; apply executes them and records the result in state.
5. **What is drift?** A mismatch between managed real-world infrastructure and declared state/configuration, often caused by external changes.
6. **What is a VPC?** An isolated virtual network within an AWS region.
7. **What is `/16`?** A CIDR prefix with 16 network bits and 16 address bits in IPv4.
8. **Is `Name` a VPC's Terraform resource label?** No; AWS Name is usually a tag, while the resource label identifies the block locally.
9. **Does VPC creation imply a public subnet?** No; subnets and routing are separate resources.
10. **What should go in Git?** Terraform source and normally the provider lock file; never state or credentials.
11. **How does Terraform know ordering?** Its dependency graph, usually inferred from references.
12. **Why keep a remote backend?** Shared secure state access, collaboration and concurrency control when supported.
13. **What does a no-change plan prove?** Terraform found no planned actions against the current checked state at that moment.
14. **When use `depends_on`?** For dependencies not expressed through resource attribute references.
15. **How do Terraform and Ansible differ?** Terraform primarily provisions infrastructure; Ansible primarily automates configuration and operations.

## 16. Suggested next hands-on progression
VPC → subnets → Internet Gateway/routes → security groups → EC2 → modules/variables → remote state → plan/apply pipeline → drift detection → automated validation.

### Official references
- [Terraform documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform language](https://developer.hashicorp.com/terraform/language)
- [Terraform state](https://developer.hashicorp.com/terraform/language/state)
- [AWS provider VPC resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
- [AWS VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
