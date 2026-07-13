# Threat Model: CrossAccountReadOnly Role

## Purpose

This role demonstrates a cross-account access pattern where an external AWS
account is delegated read-only access to resources in this account. This is
a common pattern in enterprise environments where a centralized security or
audit account needs visibility into workload accounts without having standing
credentials in each one.

## What This Role Can Do

- Read configuration and metadata for all AWS services via ReadOnlyAccess
- List and describe resources across all AWS services
- View CloudWatch metrics and logs

## What This Role Cannot Do

- Create, modify, or delete any AWS resource
- Change IAM policies, roles, or users
- Be assumed without providing the correct ExternalId value
- Be assumed by any identity outside the trusted principal account

## Cross-Account Pattern Explained

In a real deployment this role would work as follows:

1. This account creates the role with a trust policy naming the external
   account ID as the trusted principal
2. The external account grants its own IAM identities permission to call
   sts:AssumeRole targeting this role's ARN
3. When an identity in the external account needs access, it calls AssumeRole
   with the correct ExternalId and receives temporary credentials
4. Those temporary credentials expire automatically after the session ends

This pattern means the external account never holds standing credentials in
the workload account. Access is granted on demand and expires automatically.

## Note on Account ID in This Lab

This is a single-account lab environment. AWS does not allow placeholder or
nonexistent account IDs in trust policy Principal fields, so the trust policy
uses this account's own ID rather than a separate external account ID. In a
production cross-account deployment, the Principal would reference a different
AWS account ID belonging to the trusted external account.

## Threat Scenarios

### Scenario 1: Role ARN Exposed

An attacker discovers the ARN of this role and attempts to assume it from
their own AWS account.

Result: Denied. The trust policy restricts the Principal to a specific AWS
account. An identity in any other account cannot assume this role regardless
of whether they know the ARN.

### Scenario 2: Confused Deputy Attack

A malicious third party tricks a legitimate service that has permission to
assume this role into doing so on the attacker's behalf.

Result: Mitigated by the ExternalId condition. The ExternalId is a shared
secret known only to this account and the trusted external account. Even if
a malicious party tricks the external service into calling AssumeRole, they
cannot provide the correct ExternalId and the assumption is denied.

### Scenario 3: Credentials Compromised

If the temporary credentials issued by this role are stolen, an attacker
gains read access to the environment configuration identical in scope to
the SecurityAuditor role.

Blast radius: Information disclosure only. No write permissions exist on
this role. Credentials expire automatically.

## Trust Policy Design

The Condition block uses StringEquals to require an exact match on the
sts:ExternalId key. In a production deployment the ExternalId value would
be a randomly generated string stored securely and shared only with the
trusted external account through an out-of-band channel.