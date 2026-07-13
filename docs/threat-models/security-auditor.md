# Threat Model: SecurityAuditor Role

## Purpose

This role is designed for personnel who need visibility into the AWS environment
for compliance review, security assessment, or audit purposes. It provides broad
read access with zero ability to modify any resource.

## What This Role Can Do

- Read configuration and metadata for all AWS services via ReadOnlyAccess
- Read security-relevant configuration including IAM policies, CloudTrail logs,
  security group rules, and S3 bucket policies via SecurityAudit
- List and describe resources across all AWS services
- View CloudWatch metrics and logs

## What This Role Cannot Do

- Create, modify, or delete any AWS resource
- Change IAM policies, roles, or users
- Disable logging or modify CloudTrail settings
- Launch or terminate EC2 instances

## Threat Scenarios

### Scenario 1: Credentials Compromised

If the temporary credentials for this role are stolen, an attacker gains read
access to environment configuration. They can enumerate resources, review IAM
policies, and understand the architecture. They cannot modify anything.

Blast radius: Information disclosure only. No data destruction, no privilege
escalation, no service disruption possible through this role alone.

Mitigating controls: Credentials are temporary and expire automatically.
CloudTrail logs all API calls made with these credentials, providing a full
audit trail of what was accessed.

### Scenario 2: Privilege Escalation Attempt

An attacker with these credentials attempts to attach additional policies to
escalate privileges.

Result: Denied. The role has no iam:AttachRolePolicy or any other IAM write
permission. Escalation through IAM modification is not possible with this role.

## Why Managed Policies Were Used

AWS managed policies are maintained by AWS and updated automatically when new
services launch. For a standard auditor role, managed policies are the correct
choice because the requirement is broad read access, not scoped access to
specific resources. Using custom policies here would require ongoing maintenance
to add new services as AWS expands.

## Trust Policy Design

The trust policy allows the iamadmin user to assume this role. In a production
environment, the principal would be a specific IAM group or federated identity
representing audit personnel, not an admin user. The current design reflects
a lab environment where a single admin user is the only identity available.