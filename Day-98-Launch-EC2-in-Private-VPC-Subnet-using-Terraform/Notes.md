# Day 98 — Terraform and AWS Private Networking: Detailed Technical Notes

## 1. What this challenge proves

You can declaratively provision a complete private network slice using Terraform: custom VPC, subnet, security group, and EC2 instance, with explicit networking constraints, reusable variables, named outputs, and independent cloud verification. The KodeKloud challenge passed on the **second** attempt following a persistent Terraform plan update.

## 2. Infrastructure as Code (IaC)

IaC represents desired infrastructure in version-controlled configuration. Instead of manually creating objects through the AWS console, Terraform computes changes between configuration, state, and resources and invokes provider APIs to converge on the desired result.

**Declarative vs imperative:** Declarative code says *what should exist*; imperative automation describes *how to make it happen*. Terraform uses HCL, creates a dependency graph, plans changes and applies them through providers. It does not automatically correct every kind of drift, especially attributes configured with lifecycle exceptions.

## 3. Terraform core architecture

- **Terraform CLI/core:** reads HCL, evaluates expressions and creates the dependency graph.
- **Provider:** plugin that maps Terraform resources to platform APIs (here, AWS).
- **Resource:** a managed infrastructure object, e.g. `aws_subnet`.
- **Data source:** reads existing infrastructure, without managing its lifecycle.
- **State:** mapping of resource addresses to real-world objects and observed attributes.
- **Plan:** proposed actions after comparing configuration, state and provider refresh.
- **Apply:** execution of the approved actions.
- **Outputs:** named expressions evaluated from infrastructure attributes.
- **Modules:** reusable collections of Terraform configuration.

The lab used preconfigured `provider.tf` in `/home/bob/terraform`, and the files `main.tf`, `variables.tf`, and `outputs.tf` were created for the challenge.

## 4. Terraform workflow in depth

| Command | Purpose | Important limitation |
|---|---|---|
| `terraform init` | Backend/provider/module initialization | Does not deploy infrastructure |
| `terraform fmt` | HCL style normalization | Does not validate AWS correctness |
| `terraform validate` | Checks Terraform syntax and schema | Cannot guarantee permissions or cloud capacity |
| `terraform plan` | Shows proposed changes | A clean plan with ignored attributes can still conceal drift |
| `terraform apply` | Executes proposed changes | May partially succeed on API failures |
| `terraform output` | Prints output values | Outputs are not proof of live connectivity |
| `terraform state list` | Lists managed addresses | Does not necessarily prove resource health |

A clean plan is valuable, but it is not an end-to-end operational test. Always validate critical cloud settings directly.

## 5. AWS VPC fundamentals

A Virtual Private Cloud is a logically isolated networking boundary within an AWS region. A VPC has a CIDR block defining its private IP space. This challenge created `datacenter-priv-vpc` with `10.0.0.0/16`.

CIDR `/16` leaves 16 bits for host addresses: 2^16 = **65,536** IPv4 addresses in the range. This is the mathematical size of the IPv4 CIDR block, not a promise that all addresses can be assigned as AWS ENI addresses.

The VPC has a main route table and local routing for its own VPC CIDR. The challenge did not create an Internet Gateway, NAT Gateway or internet default route.

## 6. Subnet and CIDR design

The subnet `datacenter-priv-subnet` uses `10.0.1.0/24`, a subset of the parent `10.0.0.0/16` VPC CIDR. A `/24` contains 256 IPv4 addresses. AWS reserves five IPv4 addresses in each subnet, leaving **251** usable IP addresses under standard IPv4 subnet reservation rules.

`map_public_ip_on_launch = false` disables automatic public IPv4 assignment to instances launched in the subnet by default. The EC2 block additionally sets `associate_public_ip_address = false` explicitly. These controls are distinct from route-table policy.

**What makes a subnet private?** Its effective route table does not have a route to an Internet Gateway for internet destinations. Merely suppressing public IPs does not automatically guarantee private routing. A NAT Gateway can provide outbound-only internet access while instances remain private, but this lab intentionally omits it.

## 7. EC2 instance networking

`aws_instance.private_ec2` references the created subnet and security group:

```hcl
subnet_id              = aws_subnet.private_subnet.id
vpc_security_group_ids = [aws_security_group.private_sg.id]
associate_public_ip_address = false
```

EC2 receives a private IP from the subnet. It should not receive a public IPv4 address. The instance type is fixed as `t2.micro`. The lab AMI value reflects the previous challenge; images may not be available in other regions or AWS accounts.

**No direct internet connectivity:** Without suitable routes, NAT, or VPC endpoints, private EC2 instances cannot reach public package repositories. In production, choose controlled outbound mechanisms and Systems Manager access where appropriate.

## 8. Security group fundamentals

AWS security groups are **stateful**: return traffic for an allowed request is automatically permitted, regardless of whether a reciprocal inbound rule exists. Rules apply to network interfaces, not abstract subnets.

`protocol = "-1"` represents all protocols. In the lab, ingress uses `cidr_blocks = [var.KKE_VPC_CIDR]`, so sources must be within `10.0.0.0/16`. This is broad internal access across protocols and ports, not fine-grained least privilege. Outbound is also restricted to the VPC CIDR in the supplied example.

**CIDR limitations:** An IP range condition does not authenticate that the sender is an authorized VPC workload. Design secure routes, identities, SG-to-SG rules, NACLs, and application authentication as appropriate.

**Security groups vs NACLs:** SGs are stateful and associated with ENIs; NACLs are stateless and associated with subnets. Rules operate at different layers and combine with routing.

## 9. Why the first attempt failed

The initial configuration successfully created all four resources, but a subsequent `terraform plan` repeatedly displayed:

```text
Plan: 0 to add, 1 to change, 0 to destroy.
```

The proposed in-place change targeted `aws_instance.private_ec2.vpc_security_group_ids`. KodeKloud's checker failed with a message that the plan still showed pending changes. This is a specific observable symptom; the screenshot did not prove the underlying cause (e.g., simulator response, provider normalization, or actual association mismatch).

## 10. `lifecycle.ignore_changes` explained

The passing retry added:

```hcl
lifecycle {
  ignore_changes = [vpc_security_group_ids]
}
```

Terraform still considers the configured security group when **creating** the instance, but ignores future proposed updates based solely on changes to that attribute. This can produce a clean plan while a real drift remains.

**The safe verification pattern for this lab:**

1. Apply the configuration.
2. Check `terraform plan` reports no pending changes.
3. Query AWS for the actual instance's attached security groups.
4. Confirm `datacenter-priv-sg` is attached and has ingress from only `10.0.0.0/16`.
5. Confirm the instance has no public IPv4 address.

**Production recommendation:** Diagnose and eliminate the mismatch rather than ignoring security-critical attributes. Use `ignore_changes` only with a justified ownership boundary and compensating drift/security monitoring.

## 11. Variables and typed inputs

Two exact names were required:

```hcl
variable "KKE_VPC_CIDR" {
  type = string
  default = "10.0.0.0/16"
}
variable "KKE_SUBNET_CIDR" {
  type = string
  default = "10.0.1.0/24"
}
```

References (`var.KKE_VPC_CIDR`) use the values in `main.tf`. Defaults make the lab runnable without additional `-var` or `-var-file` flags.

In production, add validations (such as `can(cidrhost(...))`) and enforce subnet containment/non-overlap using design standards and suitable validation logic.

## 12. Outputs and dependency graph

Required output names:

- `KKE_vpc_name`: VPC Name tag.
- `KKE_subnet_name`: subnet Name tag.
- `KKE_ec2_private`: EC2 Name tag, **not** the instance's private IP address.

These output expressions reference managed resources, so Terraform knows their values after create/refresh. Every HCL reference like `aws_vpc.private_vpc.id` also creates an **implicit dependency**. Explicit `depends_on` is unnecessary for the core relationships shown in this project.

## 13. State, remote backends, and drift

State contains real-world identifiers and attributes. It can contain sensitive values, so avoid committing it or uploading it to public repositories. For teams, use a secured remote backend with encryption, access controls, locking or equivalent concurrency protection, audit logging, and backup/versioning. `terraform plan` performs refresh-based comparisons against the provider's view, but ignored fields are excluded from actionable change detection.

The `.terraform.lock.hcl` file records provider selections and checksums; it is generally appropriate to commit it for reproducibility, unlike `.terraform/` and `*.tfstate`.

## 14. IAM and least privilege

Terraform needs IAM permissions to create and inspect VPCs, subnets, security groups, and EC2 instances. In production, use least-privilege roles or workload identity federation rather than long-lived access keys. Separate plan/approval and apply permissions when feasible. Do not embed credentials in `provider.tf`, variables, shell history, or public docs.

## 15. Common failure modes

| Symptom | Investigation |
|---|---|
| `InvalidSubnet.Range` | Subnet must fit inside parent VPC CIDR |
| `InvalidAMIID.NotFound` | Check AMI availability in target region |
| `InvalidGroup.Duplicate` | Inspect existing SG name in VPC and Terraform state |
| Public IP assigned unexpectedly | Inspect subnet auto-assign and instance override |
| EC2 unreachable externally | Expected for truly private architecture; use private access paths |
| Persistent SG update in plan | Compare `vpc_security_group_ids` with AWS's attached groups |
| Terraform reports no changes but SG is wrong | Check whether lifecycle ignore masks the mismatch |
| Unexpected EC2 replacement | Inspect which attributes are marked `forces replacement` |
| Missing outputs | Check exact, case-sensitive names |
| `AccessDenied` | Inspect IAM permission scope, account, and credentials |

## 16. Production improvements beyond the lab

- Separate private subnets across at least two availability zones for resilience.
- Use tighter ingress protocols and ports, preferably SG references where appropriate.
- Add VPC endpoints for necessary AWS services or controlled NAT for outbound needs.
- Prefer Systems Manager for operational access instead of exposing SSH.
- Use customer-managed encryption and secure EBS configurations where required.
- Add tags, flow logs, monitoring, and policy checks.
- Replace the hardcoded AMI with a tested versioned image lookup or pinned image release process.
- Remove the `ignore_changes` workaround once underlying drift is resolved.
- Use modules and environment-separated state for scaling.

## 17. Interview questions and model answers

**Q1. What is a VPC?** A logically isolated virtual network in an AWS region.

**Q2. How do a VPC and subnet differ?** VPC defines a network boundary/IP range; a subnet partitions that range in a specific AZ.

**Q3. Why is `10.0.1.0/24` valid inside `10.0.0.0/16`?** The /24 is a narrower, contained subset of the /16 address space.

**Q4. How many IPv4 addresses are in a /24?** 256 total; AWS reserves five per subnet.

**Q5. What makes a subnet private?** The effective routing lacks a route to an Internet Gateway for public destinations.

**Q6. Does disabling public IPs alone make a subnet private?** No. Routes, subnet behavior and external connectivity determine the security posture.

**Q7. What is the purpose of a security group?** Stateful network-traffic control for ENIs.

**Q8. Difference between SG and NACL?** SG is stateful/ENI-scoped; NACL is stateless/subnet-scoped.

**Q9. What does `protocol = "-1"` mean?** All IP protocols; use sparingly with appropriate source/destination limits.

**Q10. Does `10.0.0.0/16` guarantee trustworthy origin?** No; it's an address-based restriction, not an authentication mechanism.

**Q11. Why use Terraform variables?** Reuse configurations without hardcoding environment-specific values.

**Q12. Why use Terraform outputs?** Expose selected computed attributes to users or downstream automation.

**Q13. What is an implicit dependency?** A dependency inferred from one resource referencing another's attributes.

**Q14. What is Terraform state?** The mapping and observed attributes Terraform uses to track managed infrastructure.

**Q15. What is drift?** A mismatch between real resources and the intended/configured infrastructure.

**Q16. What does `terraform plan` show?** Proposed actions from configuration and provider/state comparisons.

**Q17. What does `ignore_changes` do?** Suppresses planned updates arising from selected attribute differences after create.

**Q18. Why is ignoring `vpc_security_group_ids` risky?** Unauthorized or accidental SG changes may go undetected in Terraform plans.

**Q19. Why might a private EC2 be unable to install packages?** No NAT, internet route or required VPC endpoints.

**Q20. How should a DevOps engineer validate private EC2 deployment?** Check state/plan plus real subnet, route table, public IP, SG, and instance attributes through cloud APIs.

**Q21. Why keep `.terraform.lock.hcl`?** It pins provider selections/checksums for reproducible initialization.

**Q22. Why avoid committing `.tfstate`?** It can expose secrets, identifiers, and sensitive infrastructure details.

**Q23. Is the `KKE_ec2_private` output a private IP?** No: the challenge explicitly asked for the EC2's name, so it returns the Name tag.

**Q24. What is the difference between `map_public_ip_on_launch` and `associate_public_ip_address`?** One sets subnet default behavior; the other sets instance-specific launch behavior.

## 18. Conclusion

Day 98 connected Terraform provisioning, AWS network boundaries, subnet CIDR design, EC2 placement, security group rules, variables, outputs, and drift troubleshooting. The main operational lesson is to validate infrastructure both through IaC and the cloud control plane, especially when using lifecycle exceptions.

### References

- https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle
- https://developer.hashicorp.com/terraform/language/state
- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html
- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
