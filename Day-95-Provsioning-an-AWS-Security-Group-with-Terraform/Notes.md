# Day 95 — Terraform and AWS Security Groups: Detailed Notes

## 1. Infrastructure as Code (IaC)

IaC expresses infrastructure in machine-readable configuration and uses tools to reconcile desired resources with current infrastructure. Terraform is primarily **declarative**: the configuration describes desired results, and Terraform works out a dependency-aware action plan. It is not an imperative shell script, and it does not automatically guarantee that every external system is idempotent.

Benefits: reviewable changes, version control, reproducibility, automated drift detection, reusable modules, standardized deployments. Costs: protecting state and credentials, provider version management, change review, and remote service limits.

## 2. Core Terraform architecture

- **Terraform CLI/core:** loads HCL, constructs the dependency graph, evaluates expressions, compares state with refreshed provider data, and plans operations.
- **Provider:** a plugin implementing resource/data-source operations for an API (here AWS).
- **Resource block:** declares infrastructure Terraform manages (e.g. `aws_security_group`).
- **Data block:** reads existing infrastructure (e.g. `data.aws_vpc.default`).
- **State:** maps configured resource addresses to real-world remote objects; stored locally or in a configured backend.
- **Backend:** determines where Terraform state is held and how operations/locking work, subject to backend capabilities.
- **Module:** a reusable set of Terraform files; every root working directory is a root module.

### Provider vs backend

A provider speaks to AWS APIs; a backend stores and retrieves Terraform state. They are different responsibilities. Credentials for providers and backends may be separate.

## 3. HCL syntax and resource addressing

```hcl
resource "aws_security_group" "datacenter_sg" {
  name        = "datacenter-sg"
  description = "Security group for Nautilus App Servers"
  vpc_id      = data.aws_vpc.default.id
}
```

`resource` is the block type, `aws_security_group` is the provider resource type, and `datacenter_sg` is the local logical name. The Terraform address is `aws_security_group.datacenter_sg`; the AWS-facing group name is `datacenter-sg`. Logical names are not AWS names.

## 4. Data sources vs resources

```hcl
data "aws_vpc" "default" {
  default = true
}
```

This queries the existing default VPC; it does **not** create one. `data.aws_vpc.default.id` is an expression resolving the selected VPC ID. The lookup can fail if the region has no default VPC or current credentials lack permission. Explicit references also establish implicit graph dependencies.

## 5. AWS VPC basics

A **VPC** is a logically isolated AWS networking environment scoped to a region. Its subnets exist in individual Availability Zones; routing, internet gateways, NAT, NACLs, and security groups determine traffic flow. A default VPC is commonly provisioned by AWS with basic subnet and gateway infrastructure, but it may be absent or deleted. A security group belongs to exactly one VPC.

Day 94 created a new VPC called `devops-vpc`. **Day 95 is different:** its SG belongs to the region's **default VPC**, not the Day 94 custom VPC.

## 6. Security groups — fundamentals

AWS security groups are stateful, allow-only virtual firewalls attached to supported network interfaces/resources. They have inbound (ingress) and outbound (egress) rules. There are no explicit deny rules. Traffic permitted for a connection is tracked so return traffic is allowed without adding the inverse rule.

A security group resource definition by itself does **not** expose any EC2 instance: it must be associated with an instance's interface or other supported resource before its rules control that resource.

### Requested ingress rules

| Purpose | Protocol | From | To | IPv4 source |
|---|---|---:|---:|---|
| HTTP | TCP | 80 | 80 | `0.0.0.0/0` |
| SSH | TCP | 22 | 22 | `0.0.0.0/0` |

`0.0.0.0/0` is the entire IPv4 address space (not an IPv6 source). HTTP is commonly TCP port 80, SSH TCP port 22. `from_port == to_port` allows precisely one port. AWS security group rule APIs use TCP/IP protocol identifiers; in Terraform we use `ip_protocol = "tcp"`.

### Security note

Public SSH (`0.0.0.0/0:22`) is specifically required for this exercise, but dangerous on internet-reachable machines. Use narrow allowlists, centralized access, managed administration paths (e.g. SSM), network segmentation, least-privilege IAM, and auditing in production.

## 7. Egress and default rule behavior

`aws_vpc_security_group_egress_rule` explicitly allows outbound IPv4 traffic with `ip_protocol = "-1"`. This represents all protocols; port fields are not used for an all-protocol rule. Egress requirements weren't specified in the lab, but explicitly managing outbound traffic makes the demonstration predictable.

Do not confuse *outbound* rules with inbound rules. Two inbound rules remain exactly two inbound rules. Do not assume the AWS default outbound rule will remain untouched when managing the SG with Terraform; provider behavior and resource types matter.

## 8. Standalone rules vs inline rules

The reference uses standalone resources:

- `aws_vpc_security_group_ingress_rule` — one per ingress rule.
- `aws_vpc_security_group_egress_rule` — one egress rule.

These have their own Terraform addresses and AWS rule identities. An older/common pattern is inline `ingress {}` and `egress {}` within `aws_security_group`, or older `aws_security_group_rule`. Avoid mixing inline and standalone management for the same SG rules, which can create conflicts/drift. Check provider compatibility; newer resource types require an appropriate AWS provider version.

## 9. Dependencies and ordering

```hcl
security_group_id = aws_security_group.datacenter_sg.id
```

This is an implicit dependency. Terraform creates the SG before rules referencing its ID. `depends_on` is useful only for dependencies that cannot be expressed through normal references; unnecessary explicit dependencies complicate plans.

## 10. Terraform commands: lifecycle

| Command | What it does |
|---|---|
| `terraform init` | Initializes working directory, downloads providers/modules, configures backend |
| `terraform fmt` | Canonical HCL formatting (no infrastructure mutation) |
| `terraform validate` | Checks syntax and internal configuration validity; does not prove AWS permissions |
| `terraform plan` | Builds a proposed change plan using state and provider data |
| `terraform apply` | Executes approved changes and updates state |
| `terraform state list` | Shows addresses Terraform tracks |
| `terraform state show ADDRESS` | Shows attributes tracked for a resource |
| `terraform providers` | Lists provider requirements |
| `terraform destroy` | Plans/removes managed resources; high-impact operation |

`terraform plan` is a preview, not a deployment. `terraform validate` does not require the actual infrastructure to exist, nor does it ensure a successful apply. `No changes` means Terraform sees the state and provider-reported reality as aligned at that moment, not a permanent guarantee.

## 11. Terraform state and security

`terraform.tfstate` includes resource attributes and can include secrets or sensitive operational data. Keep it out of public Git repos. `.terraform/` contains downloaded binaries and caches. The `.terraform.lock.hcl` dependency lock file is generally **safe and recommended to commit** for consistent provider selections. Depending on organizational requirements, use an appropriately secured remote backend with access control, encryption, and state locking when supported.

The provided `.gitignore` omits state, crash logs, caches, and secrets but does not ignore `.terraform.lock.hcl`.

## 12. Variables, outputs and reusable modules (next steps)

Variables parameterize configurations (`var.region`, `var.security_group_name`); locals reuse computed expressions; outputs expose selected values after apply. Modules group related resources with inputs and outputs. This lab intentionally uses literal names for clarity and because the exercise targets a single SG, but production code can wrap security group creation into a reusable module.

## 13. Verification strategy

Apply is not the only evidence:

1. Check Terraform plan for intended creates and no destruction.
2. Inspect `terraform state show aws_security_group.datacenter_sg` for name/description/VPC ID.
3. Query AWS CLI with `describe-security-groups` and check exactly two expected `IpPermissions` entries.
4. Match SG VPC ID to the default VPC ID in `us-east-1`.
5. Rerun `terraform plan` to check for pending drift.

An AWS rule list may appear in a different order; compare rule values, not array positions.

## 14. Common failures and root-cause approaches

- **No default VPC:** `data.aws_vpc.default` fails; inspect `describe-vpcs` and the region.
- **Duplicate group:** `InvalidGroup.Duplicate`; find the existing SG, and do not blindly create another.
- **Duplicate rule:** check AWS rule IDs and whether rules are already managed in state.
- **Authentication/authorization:** inspect provider configuration, identity (`aws sts get-caller-identity`), and EC2 permissions.
- **Unsupported resource type:** identify and confirm the installed AWS provider version.
- **Unexpected destroy:** stop and examine source/state drift, resource address changes, and workspaces.
- **AWS reachable but port not:** verify SG attachment, subnet routes, NACLs, OS firewall, application listener, and instance public addressing. A security group allow rule alone does not ensure end-to-end access.

## 15. Terraform in CI/CD

A common workflow is `terraform fmt -check` → `terraform validate` → `terraform plan` → peer review/approval → controlled `terraform apply`. Pin provider versions, protect credentials and state, restrict production apply permissions, publish plan artifacts securely, and guard against concurrent state operations. For large teams, use remote state/backend patterns and policy checks.

## 16. Interview questions and concise answers

1. **What is IaC?** Version-controlled definitions for provisioning/managing infrastructure via automation.
2. **What is a Terraform provider?** A plugin implementing operations against a platform API.
3. **Data source vs resource?** Data source reads an existing object; resource manages an object lifecycle.
4. **What does state do?** Maps Terraform addresses to remote objects and records tracked metadata.
5. **Does `terraform validate` contact AWS to verify permissions?** No, not as a complete AWS access/deployment test.
6. **What does `terraform plan` do?** Produces a proposed set of infrastructure changes.
7. **What is drift?** Deviation between managed configuration/state and real-world infrastructure.
8. **What is an implicit dependency?** Ordering inferred from expressions referencing another resource.
9. **Security group vs NACL?** SG is stateful and attached at resource/network-interface level; NACL is stateless at subnet boundary.
10. **Why is SSH 0.0.0.0/0 risky?** Any IPv4 source can attempt to connect to the SSH port if routing permits.
11. **What is `ip_protocol = "-1"`?** An AWS all-protocol rule selector.
12. **Does creating an SG secure an existing instance automatically?** No, it must be attached.
13. **Why commit `.terraform.lock.hcl`?** To record selected provider dependency versions/checksums for reproducibility.
14. **Why keep Terraform state private?** It can include sensitive resource details or credentials.
15. **Why avoid mixing inline and standalone SG rules?** Conflicting rule ownership can cause perpetual differences or removals.
16. **Where is a security group located?** In a specific VPC within an AWS region.
17. **Why use a VPC data source?** Reuse existing networking without accidentally provisioning a new VPC.
18. **Is `No changes` an absolute security guarantee?** No; it only indicates no tracked changes at plan time.

## 17. Day 95 recap

The core achievement is provisioning `datacenter-sg` inside the **default AWS VPC** in `us-east-1`, granting two narrowly defined *ports* (though very broad sources) and explicitly handling egress. This exercise reinforces Terraform data-source lookups, resource dependencies, network firewalls, state, and verification against AWS APIs.
