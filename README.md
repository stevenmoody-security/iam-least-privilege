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
Write access scoped to a single named S3 bucket using a custom IAM policy.
No access to any other bucket or any other AWS service.

Demonstrates: custom policy authoring, resource-level scoping, action-level
scoping within a single service, least-privilege for application access.

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