# IAM Least-Privilege Role Designs

Three IAM roles demonstrating least-privilege access patterns for common
enterprise use cases, each with a documented threat model covering blast
radius, attack scenarios, and design rationale.

Built and deployed to AWS using the CLI. Policy files are sanitized for
public sharing; account IDs have been replaced with ACCOUNT_ID_REDACTED.

---

## Roles

### SecurityAuditor
Read-only access across all AWS services for compliance review and security
assessment. Uses two AWS managed policies: SecurityAudit and ReadOnlyAccess.

Demonstrates: managed policy selection, auditor access pattern, zero write
permission design.

Threat model: docs/threat-models/security-auditor.md

---

### ScopedDeveloper
Write access scoped to a single named S3 bucket using a custom IAM policy,
with a permission boundary enforcing an S3-only ceiling.

Demonstrates: custom policy authoring, resource-level scoping, two-statement
S3 policy structure (bucket ARN for ListBucket, object ARN for GetObject and
PutObject), permission boundary as a meaningful ceiling distinct from the
identity policy, and verified access denial to other buckets and all non-S3
services.

Effective permissions are the intersection of the identity policy and the
boundary. The identity policy scopes access to one bucket. The boundary caps
the maximum to S3 only. Attaching AdministratorAccess to this role would
still yield no IAM, no EC2, no KMS. Only S3. The boundary proves this
through verified CLI output showing AccessDenied on a second bucket and
UnauthorizedOperation on EC2.

Threat model: docs/threat-models/scoped-developer.md

---

### CrossAccountReadOnly
Read-only access role designed for cross-account delegation, with an
ExternalId condition on the trust policy to prevent confused deputy attacks.

Demonstrates: cross-account trust pattern, ExternalId condition, defense
against confused deputy, temporary credential delegation.

Threat model: docs/threat-models/cross-account-readonly.md

---

## Background

This project is part of a broader cloud security portfolio built during my
transition from U.S. Army service to cloud security engineering. The access
control patterns demonstrated here reflect the same least-privilege principles
applied to classified systems in operational environments, translated into
AWS-native IAM constructs.

Active TS/SCI | AWS SAA | SSCP | Relocating to Raleigh, NC February 2027