# Day 97 — Deep-Dive Notes: Terraform and AWS IAM

## 1. What is IAM?

AWS Identity and Access Management (IAM) controls **authentication-related identities and authorization** for AWS API operations. It lets teams manage users, groups, roles, and permissions. AWS account-root credentials are not an operational IAM user. Production applications generally use IAM roles and temporary credentials rather than long-lived access keys.

### Important nouns

- **Principal:** The authenticated entity making a request (user, role session, federated identity, service principal).
- **IAM user:** Long-lived account-local identity; prefer roles for workloads.
- **IAM group:** Collection of IAM users for permission administration.
- **IAM role:** Assumable identity that produces temporary credentials, often used by workloads or cross-account operators.
- **Policy:** JSON document describing permission rules. A standalone policy grants no access until it is attached or otherwise applied appropriately.
- **AWS API action:** Operation such as `ec2:DescribeInstances` or `ec2:RunInstances`.
- **Resource ARN:** AWS resource identifier; IAM policies sometimes support resource-level restrictions.

## 2. Policy document anatomy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2ReadOnlyAccess",
      "Effect": "Allow",
      "Action": ["ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

- `Version` identifies the IAM policy language version, **not** the Terraform or AWS provider version.
- `Statement` is an array of permission entries.
- `Sid` is an optional human-readable statement identifier.
- `Effect` is `Allow` or `Deny`.
- `Action` specifies exact API actions or wildcard patterns.
- `Resource` lists applicable ARNs or `*` where resource-scoped control is unsupported.
- `Condition` (not used here) can refine authorization using context such as source IP, MFA, or resource tags when supported.

## 3. How IAM evaluates permissions

IAM begins from implicit denial. An applicable identity or resource-based allow may grant an action, subject to permission boundaries, service control policies, session policies, resource policies, and other applicable restrictions. An applicable **explicit deny overrides an allow**. The full authorization model is context-dependent: not every policy type combines the same way in every scenario.

**Critical distinction:** The Day 97 policy does not grant write actions; it also does not explicitly deny them. If an identity has another policy allowing `ec2:TerminateInstances`, that action may remain available.

## 4. What does `ec2:Describe*` permit?

It matches EC2 actions beginning with `Describe`, including:

| API action | Typical information |
|---|---|
| `ec2:DescribeInstances` | Instance IDs, states, network and instance metadata |
| `ec2:DescribeImages` | AMI attributes and availability |
| `ec2:DescribeSnapshots` | EBS snapshot metadata |
| `ec2:DescribeVolumes` | EBS volume information |
| `ec2:DescribeSecurityGroups` | Security groups and security group rules |
| `ec2:DescribeVpcs` | VPC details |
| `ec2:DescribeSubnets` | Subnet details |

It does not include `RunInstances`, `StopInstances`, `TerminateInstances`, `DeleteSnapshot`, or `CreateVolume`.

**Least privilege consideration:** The wildcard is broader than the three named resources. In production, prefer an explicit allow-list when feasible. Also, some AWS Console pages call APIs outside the `Describe*` family, so a Describe-only policy may not make every console feature fully functional.

## 5. Why `Resource = "*"`?

Many EC2 Describe actions do not support per-resource authorization. Their policy statements must use `Resource: "*"`. This does **not** mean all API actions are allowed: scope is still constrained by `Action`, and other policy restrictions may apply.

## 6. IAM global scope versus AWS provider region

IAM identities and customer-managed policies are account-global, whereas resources such as EC2 instances and VPCs are regional. The lab specifies AWS provider region `us-east-1` for its infrastructure environment, **not** as a region constraint on this IAM policy.

If a specific EC2 API call should be constrained by region, examine whether its action supports relevant condition keys such as `aws:RequestedRegion`; policy design must respect IAM's action and condition compatibility.

## 7. Terraform fundamentals refreshed

Terraform is declarative IaC: configuration declares desired infrastructure and providers map it to cloud APIs. A typical workflow:

```text
Write HCL -> terraform init -> terraform fmt -> terraform validate
          -> terraform plan -> review -> terraform apply
          -> inspect state and drift
```

- **Provider:** Plugin implementing AWS API operations.
- **Resource:** Object Terraform manages (`aws_iam_policy`).
- **Data source:** Reads an object without creating it.
- **State:** Mapping between Terraform addresses and remote objects; can contain sensitive data.
- **Plan:** Proposed actions based on config, state, and provider refresh.
- **Apply:** Executes the planned changes.
- **Drift:** Remote changes made outside Terraform; detected when state is refreshed and compared.
- **Idempotence/convergence:** Reapply unchanged configuration and expect no substantive changes.

## 8. Why use `jsonencode()`?

IAM policy documents are JSON. Terraform's `jsonencode()` converts a native HCL value into JSON with correct quoting and escaping. This is easier and safer than manually formatting JSON in a heredoc.

```hcl
policy = jsonencode({
  Version = "2012-10-17"
  Statement = [{
    Effect   = "Allow"
    Action   = ["ec2:Describe*"]
    Resource = "*"
  }]
})
```

An alternative is `aws_iam_policy_document` data source, useful for composing larger policies with multiple statements and conditions.

## 9. Customer-managed vs AWS-managed vs inline policies

| Type | Owner | Reuse |
|---|---|---|
| AWS-managed policy | AWS | Can be attached to multiple identities; AWS may update it |
| Customer-managed policy | Your AWS account | Reusable and versioned IAM policy |
| Inline policy | Bound to one IAM identity | Not separately reusable |

Day 97 creates a **customer-managed** policy. It does not create an attachment.

## 10. Policy creation vs attachment

**Creation** produces a reusable IAM policy ARN. **Attachment** would associate it with an IAM user, role, or group; this was **not requested** in the KodeKloud exercise. A role could be granted this policy later using the appropriate Terraform attachment resource, after considering the intended principal and security model.

## 11. Verification strategy

1. `terraform validate` checks syntax and configuration structure.
2. `terraform plan` previews change scope; for the original challenge it should propose one policy.
3. `terraform apply` creates the policy.
4. `terraform state list` confirms Terraform tracks `aws_iam_policy.siva`.
5. `aws iam get-policy-version` confirms the live policy document if the lab supports the call.
6. `terraform plan` reporting no changes establishes a clean Terraform comparison at that time.
7. **Effective authorization** requires testing the permissions for an attached principal in a real environment. No-Change does not prove that an unattached user can browse the console.

## 12. Lab-specific IAM simulator failure

During the completed challenge, `aws iam simulate-custom-policy` returned:

```text
InternalFailure: The simulate_custom_policy action has not been implemented
```

The lab emulates selected AWS APIs, and this simulator method was not supported. This is a **capability limitation of the lab API endpoint**, not evidence that the Terraform IAM policy definition is invalid. The Terraform plan showed `No changes` and the KodeKloud grader passed the task.

**Troubleshooting rule:** Distinguish a policy-simulation service/API failure from a simulation result that explicitly says `implicitDeny` or `explicitDeny`.

## 13. Security and operations best practices

- Prefer temporary role credentials and MFA for privileged access.
- Apply least privilege; review wildcard action patterns periodically.
- Use `Condition` keys cautiously and test their compatibility.
- Enforce organizational guardrails using SCPs and permission boundaries when applicable.
- Store Terraform state securely; state may include secrets and identifiers.
- Commit `.terraform.lock.hcl` for provider reproducibility; exclude state and `.terraform/`.
- Run `plan` in CI; require review/approval before applying IAM changes.
- Use CloudTrail and appropriate access-analysis tools to audit permissions.
- Avoid introducing user/role/policy attachment resources when the ticket requests only a policy.

## 14. Troubleshooting quick reference

| Symptom | Investigation |
|---|---|
| `EntityAlreadyExists` | Policy name duplicates an existing policy; inspect account and state |
| `MalformedPolicyDocument` | Verify JSON syntax, valid actions, and resource scope |
| `AccessDenied` | Check identity and account policy controls; do not bypass guardrails |
| `No valid credential sources found` | Inspect the existing AWS provider configuration |
| State and AWS disagree | Refresh and inspect, then plan carefully; avoid blind deletion |
| IAM simulator reports Not Implemented | Skip unsupported lab API; verify via supported reads and grading |

## 15. Interview questions and answers

1. **What is an IAM policy?** A JSON document defining authorization statements.
2. **What is the difference between IAM authentication and authorization?** Authentication verifies identity; authorization determines allowed actions.
3. **What is `Effect: Allow`?** It permits matching actions unless overridden by applicable denials/restrictions.
4. **What is an explicit deny?** A matching Deny that overrides permissions granted by an Allow.
5. **What is an implicit deny?** An action is denied because no applicable Allow grants it.
6. **Does a policy grant access merely by existing?** No, it must be applied/attached to an appropriate identity or resource-policy context.
7. **What does `ec2:Describe*` mean?** It matches EC2 action names beginning with Describe.
8. **Does that pattern allow termination?** No, `ec2:TerminateInstances` is not a Describe action.
9. **Why use Resource `*`?** Many Describe API actions lack resource-level permission support.
10. **Is IAM regional?** IAM customer-managed policies are global within an AWS account.
11. **Why use Terraform `jsonencode`?** To generate valid JSON from HCL safely.
12. **What is the difference between resource and data source?** A resource manages lifecycle; a data source reads existing information.
13. **What does `terraform plan` do?** Calculates and displays proposed infrastructure changes.
14. **What does a no-change plan prove?** Terraform sees no pending changes under its current state/configuration/refresh; it does not prove all effective IAM permissions.
15. **What are customer-managed policies?** Reusable account-owned IAM policy objects.
16. **When choose explicit API actions over wildcards?** When least privilege and predictable scope matter.
17. **Why is state sensitive?** It can contain security-relevant identifiers, generated credentials, or resource attributes.
18. **How would you test a policy?** Review document, simulate where supported, then test as intended identity in a controlled account.
19. **What if policy simulation is unavailable?** Use supported read-only API checks and report test limitations without assuming success.
20. **How would you manage IAM in CI/CD?** Version-control code, plan, review, gate apply, and audit changes.

## 16. Takeaway

The success condition of this challenge was a correctly named, account-managed IAM policy granting read-only EC2 visibility. The deeper DevOps lesson is to **separate the policy document, its attachment to an identity, and the effective permissions calculated at request time**.
