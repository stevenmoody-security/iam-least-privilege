# IAM Least-Privilege Role Designs

Four IAM roles demonstrating least-privilege access patterns for common
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

### EC2S3ReadRole
Read-only access role for an EC2 instance, demonstrating the non-human
identity pattern. The trust policy names ec2.amazonaws.com as the principal
rather than a human IAM user. AWS issues temporary credentials automatically
through the instance metadata service at 169.254.169.254, a link-local
address reachable only from within the instance itself. No access keys are
stored anywhere.

Demonstrates: service principal vs user principal in trust policy design,
instance metadata service credential issuance, non-human identity access
pattern, and why this pattern eliminates the stored credential risk present
in access key based authentication. This is the same principle applied to
AI agent identity through Amazon Bedrock AgentCore Identity: the agent
receives a role, AWS handles credential issuance automatically, and no
secrets are stored anywhere in the system.

Threat model: docs/threat-models/ec2-s3-read-role.md


---

## Authentication Decision Matrix

When securing access in AWS environments, the authentication mechanism depends
on where the resource being accessed lives and what kind of identity is making
the request.

### IAM Roles (SigV4)
Use when: the resource lives inside AWS. Lambda functions, DynamoDB tables,
S3 buckets, and any other AWS-native service authenticate through IAM roles.
AWS handles credential issuance and signing automatically. No secrets are
stored anywhere.

Example: EC2S3ReadRole in this repository. The EC2 instance receives a role,
AWS issues temporary credentials through the instance metadata service, and
the application makes S3 API calls without storing any access keys.

### API Keys
Use when: accessing an API Gateway endpoint where the data is not sensitive
and low friction matters more than strong authentication guarantees. API keys
are simple shared secrets passed in request headers. Appropriate for
non-sensitive integrations where OAuth overhead is not justified.

### JWT Two-Legged OAuth (2LO)
Use when: the resource lives outside your AWS account in a separate trust
domain. IAM does not reach external systems, so a standards-based token is
required. Two-Legged OAuth is machine-to-machine with no human user involved.
The client authenticates, receives a signed JSON Web Token, and presents it
to the external service. Any standards-compliant system can verify the token
without calling back to AWS.

Example: an AI agent in AWS needing to call an external partner API. The
agent authenticates using 2LO, receives a JWT, and presents it to the
partner service. This is the pattern demonstrated in the AWS Summit SEC307
workshop for cross-trust-domain tool access.

### The EC2 to AgentCore Connection
The EC2 instance role pattern is the direct conceptual precursor to Amazon
Bedrock AgentCore Identity. Both assign a role to a compute resource. Both
allow AWS to handle credential issuance automatically. Both eliminate stored
credentials entirely. The difference is that EC2 instance roles attach to
servers, while AgentCore Identity attaches to AI agents. The security
principle is identical.
---
## Background

This project is part of a broader cloud security portfolio built during my
transition from U.S. Army service to cloud security engineering. The access
control patterns demonstrated here reflect the same least-privilege principles
applied to classified systems in operational environments, translated into
AWS-native IAM constructs.

Active TS/SCI | AWS SAA | SSCP | Relocating to Raleigh, NC February 2027

