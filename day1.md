# AWS Day 1 Notes

## Topics

- AWS Regions
- Availability Zones (AZs)
- IAM users and roles
- IAM policies
- Multi-factor authentication (MFA)
- Least privilege

## 1. Regions and Availability Zones

### AWS Regions

An AWS **Region** is a geographic area, such as:

- `us-east-1` - Northern Virginia
- `ap-south-1` - Mumbai

Each Region is a fully independent and isolated set of AWS infrastructure. Data does not leave a Region unless you explicitly move or replicate it.

### Availability Zones

An **Availability Zone (AZ)** is one or more physically separate data centers within a Region. Each AZ has independent:

- Power
- Cooling
- Networking

Availability Zones within the same Region are connected by low-latency links. A Region typically contains three or more AZs, for example:

```text
AWS
`-- Region (for example, ap-south-1)
  |-- AZ-a (data center cluster)
  |-- AZ-b
  `-- AZ-c
```

## 2. IAM: Identity and Access Management

IAM is AWS's system for controlling:

> Who (identity) can do what (action) on which resource, under what conditions.

### IAM Users

An **IAM user** is a persistent identity for a person or, occasionally, an application. It can have long-term credentials such as:

- A password for the AWS Management Console
- Access keys for the AWS CLI or API

Think of an IAM user as a login account.

#### Root Account Best Practice

Do not use the root account for daily work. The root account is the email address used to create the AWS account and has unrestricted access.

For normal work:

1. Use IAM Identity Center (SSO) when possible.
2. Otherwise, create an IAM user with only the permissions it needs.
3. Protect the root account with MFA and keep it for tasks that specifically require root access.

### IAM Roles

An **IAM role** is not tied to a specific person. It is a set of permissions that can be temporarily assumed by:

- An IAM user
- An AWS service, such as EC2
- An external identity

Roles do not have permanent credentials. Instead, AWS Security Token Service (STS) provides short-lived, automatically rotating temporary credentials.

#### Why Use Roles Instead of Long-Lived Access Keys?

An EC2 instance running an application should not have a hardcoded access key stored on disk. If the key is exposed, an attacker could use it.

Instead, attach an IAM role to the EC2 instance. AWS automatically provides temporary credentials to the instance.

Roles are also useful when:

- One AWS account needs to access resources in another account.
- A human needs to switch into elevated permissions only when necessary.

### IAM Policies

An **IAM policy** is a JSON document that defines permissions. A policy commonly specifies:

- `Effect`: Whether the permission is `Allow` or `Deny`
- `Action`: The operation being allowed or denied, such as `s3:GetObject`
- `Resource`: The resource ARN to which the policy applies
- `Condition`: Optional rules that must be met

## 3. Core Security Principles

### MFA

**Multi-factor authentication (MFA)** adds an extra verification step beyond a password. Enable MFA on the root account and on identities with elevated permissions.

### Least Privilege

Grant each identity only the permissions it needs to perform its job. Review and reduce permissions over time rather than granting broad access by default.

#### Example Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-example-bucket",
        "arn:aws:s3:::my-example-bucket/*"
      ]
    }
  ]
}
```
