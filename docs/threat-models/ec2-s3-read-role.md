# Threat Model: EC2S3ReadRole

## Purpose

This role is designed for an EC2 instance running an application that needs
read access to a specific S3 bucket. It demonstrates the non-human identity
pattern: instead of storing access keys in code or environment variables, the
role is attached to the instance and AWS issues temporary credentials
automatically through the instance metadata service.

## What This Role Can Do

- List the contents of the stevenmoody-awslab-dev bucket
- Read objects from the stevenmoody-awslab-dev bucket

## What This Role Cannot Do

- Write or delete objects in any S3 bucket
- Access any other S3 bucket in the account
- Access EC2, IAM, or any other AWS service

## How Credentials Are Issued

When an EC2 instance has this role attached, AWS makes temporary credentials
available at 169.254.169.254, a link-local address reachable only from within
the instance itself. The application calls this address to retrieve credentials,
uses them to make S3 API calls, and the credentials rotate automatically. No
access keys are stored in code, environment variables, or configuration files.

## Threat Scenarios

### Scenario 1: Instance Compromise and Credential Theft

An attacker achieves code execution on the EC2 instance and calls the instance
metadata service to retrieve the temporary credentials. They export those
credentials and use them from an external system.

Blast radius: Read access to one S3 bucket only. The attacker cannot write or
delete objects, cannot access other buckets, and cannot reach any other AWS
service. The credentials expire automatically, limiting the window of exposure.

Mitigating controls: CloudTrail data events on the bucket log every GetObject
call, including the source identity. Unusual access patterns, such as high
volume reads or access from unexpected IP addresses after credential export,
are detectable. IMDSv2 can be enforced to require a session token before
the metadata service responds, adding a layer of protection against
server-side request forgery attacks that attempt to reach the metadata service.

### Scenario 2: Lateral Movement Attempt

An attacker with the stolen credentials attempts to access other S3 buckets
or other AWS services.

Result: Denied. The permissions policy names only the stevenmoody-awslab-dev
bucket ARN. All other resources are denied by default.

## Why No Permission Boundary

The ScopedDeveloper role uses a permission boundary because it has write
access and the boundary enforces an S3-only ceiling as a second layer of
control. This role has read-only access scoped to a single bucket already.
A permission boundary would add no meaningful additional constraint given
the permissions are already at their minimum necessary scope.

## Trust Policy Design

The trust policy names ec2.amazonaws.com as the principal rather than a
specific IAM user or account. This means any EC2 instance in this account
can be launched with this role attached. In a production environment, the
role would be scoped further using condition keys such as
aws:ResourceTag to limit which instances can use it.
