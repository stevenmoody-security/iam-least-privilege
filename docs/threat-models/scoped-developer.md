# Threat Model: ScopedDeveloper Role

## Purpose

This role is designed for a developer who needs read and write access to a
specific S3 bucket for application work. It demonstrates how to scope
permissions to the minimum necessary resource rather than granting broad
S3 access across the entire account.

## What This Role Can Do

- List the contents of the stevenmoody-awslab-dev bucket
- Read objects from the stevenmoody-awslab-dev bucket
- Upload objects to the stevenmoody-awslab-dev bucket
- Delete objects from the stevenmoody-awslab-dev bucket

## What This Role Cannot Do

- Access any other S3 bucket in the account
- Modify bucket configuration, policies, or ACLs
- Access EC2, IAM, RDS, or any other AWS service
- Create or delete the bucket itself

## Threat Scenarios

### Scenario 1: Credentials Compromised

If the temporary credentials for this role are stolen, an attacker can read,
write, and delete objects in the stevenmoody-awslab-dev bucket only. They
cannot pivot to other services or other buckets.

Blast radius: Limited to one S3 bucket. No other AWS resources are reachable.
The attacker cannot enumerate the broader environment or access IAM.

Mitigating controls: Credentials are temporary and expire automatically.
CloudTrail data events can be enabled on the bucket to log every object-level
operation.

### Scenario 2: Lateral Movement Attempt

An attacker with these credentials attempts to access other S3 buckets or
other AWS services to move laterally through the environment.

Result: Denied. The resource block in the custom policy explicitly names only
the stevenmoody-awslab-dev bucket ARN and its contents. All other resource
ARNs are not covered by any Allow statement. AWS denies by default anything
not explicitly permitted.

### Scenario 3: Bucket Deletion Attempt

An attacker attempts to delete the bucket itself to cause a denial of service.

Result: Denied. The policy grants s3:DeleteObject, which deletes objects
inside the bucket, but not s3:DeleteBucket, which would remove the bucket
itself. This distinction is intentional and demonstrates why action-level
scoping matters even within a single service.

## Why a Custom Policy Was Used

AWS has no managed policy that scopes S3 access to a single named bucket.
Managed policies like AmazonS3FullAccess grant access to all buckets in the
account, which violates least privilege for this use case. A custom policy
is required any time permissions need to be scoped to a specific resource ARN.

## Trust Policy Design

The trust policy allows the iamadmin user to assume this role. In a production
environment, the principal would be the specific IAM identity of the developer
or the execution role of the application that needs bucket access.