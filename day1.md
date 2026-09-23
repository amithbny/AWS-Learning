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

# AWS IAM Interview Questions and Answers

## Part 1: Conceptual Questions

### 1. What's the difference between an IAM user and an IAM role?

- **IAM user:** A permanent identity that can have long-term credentials, such as a password or access keys. It is typically used for a person or application that needs a persistent identity.

- **IAM role:** An identity with permissions that provides temporary security credentials when assumed by a trusted entity.

Roles are generally preferred for EC2 instances, Lambda functions, and cross-account access because they avoid the need to manage long-term access keys.

### 2. Why would you use a role instead of hardcoding access keys on an EC2 instance?

Hardcoding access keys creates a security risk because permanent credentials can be exposed if the instance is compromised.

Instead, I would attach an IAM role to the EC2 instance. AWS provides temporary credentials through the instance metadata service, and those credentials are automatically refreshed.

This reduces the risk of credential leakage and eliminates manual key rotation.

### 3. Explain the shared responsibility model. What does AWS manage versus what do you manage?

The shared responsibility model divides security responsibilities between AWS and the customer.

- **AWS - security of the cloud:** AWS manages physical data centers, hardware, networking infrastructure, and the underlying virtualization infrastructure.

- **Customer - security in the cloud:** The customer manages IAM permissions, data protection, application security, and configuration of AWS resources.

For EC2, the customer is also responsible for operating system patching, firewall configuration, and instance-level security.

### 4. What's the difference between an identity-based policy and a resource-based policy?

- **Identity-based policy:** Attached to an IAM user, group, or role. It specifies which actions that identity can perform on resources.

- **Resource-based policy:** Attached directly to a resource, such as an S3 bucket or SQS queue. It specifies which principals can access that resource and what actions they can perform.

Resource-based policies can also be used to grant cross-account access.

### 5. What happens when one policy allows an action and another explicitly denies it?

An explicit `Deny` always overrides an `Allow`, regardless of how many policies grant that permission.

If no applicable policy allows an action, the request is implicitly denied by default.

### 6. What is least privilege, and how would you implement it practically?

Least privilege means giving a user, role, or application only the permissions required to perform its tasks.

I would implement it by:

- Starting with minimal permissions and adding only what is required.
- Restricting resources to specific ARNs instead of using `*` wherever possible.
- Using IAM Access Analyzer and the IAM Policy Simulator to review permissions.
- Using roles and temporary credentials instead of long-term access keys.
- Reviewing last-accessed information through IAM Access Advisor and removing unused permissions.

### 7. What's the difference between a managed policy and an inline policy? When would you use inline?

- **Managed policy:** A standalone policy that can be attached to multiple IAM identities. It can be centrally maintained and reused.

- **Inline policy:** A policy embedded directly into a single IAM user, group, or role. It has a one-to-one relationship with that identity.

I would generally use managed policies because they are easier to reuse and manage centrally. Inline policies are useful when a permission must remain uniquely associated with one identity and should not be reused independently.

### 8. What is STS, and what role does it play when a role is assumed?

AWS Security Token Service (STS) provides temporary security credentials.

When an entity assumes a role using `AssumeRole`, STS issues temporary credentials consisting of:

- Access key ID
- Secret access key
- Session token

These credentials have a defined expiration time and allow the entity to access AWS resources according to the assumed role's permissions.

### 9. Can an IAM role be assumed by a user in another AWS account? How?

Yes. This is called cross-account role assumption.

The process is:

1. Create a role in the target AWS account.

2. Configure its trust policy to trust the source AWS account or a specific principal from that account.
3. Allow the source IAM user or role to call `sts:AssumeRole` on the target role.
4. The source principal calls `AssumeRole` with the target role's ARN.
5. STS returns temporary credentials for accessing the target account.

An external ID can be required for third-party access, and MFA can be required through trust policy conditions.

### 10. What's the difference between authentication and authorization in IAM?

- **Authentication:** Verifies who you are. Examples include logging in with a password, using access keys, or signing in through federation.

- **Authorization:** Determines what you are allowed to do after your identity has been established. IAM policies are used to evaluate permissions.

In simple terms, authentication answers "Who are you?" while authorization answers "What are you allowed to do?"

## Part 2: Scenario-Based Questions

### 11. A Lambda function needs to read from an S3 bucket and write to a DynamoDB table. How do you set this up securely?

I would create an IAM execution role for the Lambda function.

1. Configure the role's trust policy to allow `lambda.amazonaws.com` to assume it.

2. Attach a permissions policy allowing only the required actions, such as:

- `s3:GetObject` on the required S3 bucket objects.
- `dynamodb:PutItem` and `dynamodb:GetItem` on the specific DynamoDB table, if those operations are needed.

3. Assign the execution role to the Lambda function.

4. Avoid storing AWS access keys inside the function code or environment variables.

This follows least privilege and uses temporary credentials.

### 12. A developer's IAM user has AdministratorAccess. What's wrong with this, and what would you do instead?

AdministratorAccess grants extensive permissions across the AWS account. This violates least privilege when the developer does not need full administrative access.

I would:

- Identify the services and actions required for the developer's job.
- Create a least-privilege policy or appropriate role.
- Require MFA for privileged access.
- Provide a separate elevated-access role for occasional administrative tasks.
- Log and review privileged activity using CloudTrail.

This reduces the potential impact of compromised credentials or accidental changes.

### 13. You run `aws sts get-caller-identity` and the ARN isn't what you expected. What are the possible causes?

Possible causes include:

- The wrong AWS CLI profile was selected.
- Environment variables such as `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, or `AWS_SESSION_TOKEN` are overriding the expected credentials.
- The shell is using credentials from a previously assumed role.
- The CLI is using an EC2 instance profile instead of local credentials.
- The `AWS_PROFILE` environment variable or default profile is configured incorrectly.
- The credentials belong to a different AWS account or IAM identity.

I would inspect the active profile, credential sources, and role-assumption configuration to identify the cause.

### 14. How would you grant a third-party vendor temporary, time-limited access to one S3 bucket without creating an IAM user for them?

I would create an IAM role in my AWS account.

1. Configure its trust policy to allow the vendor's AWS account or designated principal to assume the role.

2. Require an external ID in the trust policy to help mitigate the confused deputy problem.

3. Attach a permissions policy scoped to the required S3 bucket and necessary actions.

4. Set an appropriate role session duration and apply additional conditions where needed.

5. Have the vendor assume the role using STS to receive temporary credentials.

This avoids creating a permanent IAM user for the vendor.

### 15. Your CI/CD pipeline needs to deploy to AWS. Would you use access keys or a role? Why?

I would prefer an IAM role with OIDC federation, where supported by the CI/CD platform.

For example, GitHub Actions or a suitably configured Azure DevOps pipeline can obtain a federated token and use it to assume an IAM role.

The role would have only the permissions required for deployment.

This avoids storing long-lived AWS access keys in pipeline secrets. If OIDC is unavailable, I would use an appropriate short-lived credential or role-assumption mechanism rather than relying on permanent access keys wherever possible.

## Part 3: Practical and CLI Questions

### 16. How do you check which identity your current AWS CLI session is using?

Run:

```bash
aws sts get-caller-identity
```

Example output:

```json
{
  "UserId": "AIDAXXXXXXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/developer"
}
```

It returns the account ID, user ID, and ARN of the identity associated with the current credentials.

For an assumed role, the ARN will generally contain `assumed-role` and the role session name.

### 17. Write a policy that allows read-only access to a single S3 bucket and denies deletion, even if another attached policy allows it.

Example policy for a bucket named `my-bucket`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
    },
    {
      "Sid": "ExplicitlyDenyDeletion",
      "Effect": "Deny",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteObjectVersion",
        "s3:DeleteBucket"
      ],
      "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
    }
  ]
}
```

The explicit Deny overrides applicable Allows for those deletion actions. This policy does not grant write access, and it does not prevent every possible change to the bucket, such as modifying its policy or configuration.

### 18. How do you audit which permissions an IAM role actually has, beyond just reading the attached policy JSON?

I would use several methods:

- **IAM Policy Simulator:** Test specific actions and resources to see whether they are allowed or denied by the evaluated policies.
- **IAM Access Analyzer:** Review access findings, including external access, and use policy validation and unused-access analysis where available.
- **IAM Access Advisor:** Review the services accessed by the role and their last-accessed timestamps to identify potentially unused permissions.
- **AWS CloudTrail:** Review actual API activity, including which identity performed an action, when it happened, and whether the request succeeded.

I would also consider permissions boundaries, session policies, service control policies (SCPs), and resource-based policies because attached identity policies alone do not necessarily describe the role's effective permissions.

## Part 4: Important Fintech and Trading-Platform Follow-Up Questions

These are useful topics to prepare for DevOps interviews involving separate development, UAT, and production environments.

### 19. How would you enforce MFA for privileged IAM roles?

I would require MFA for privileged access by configuring the appropriate identity and role-assumption controls.

For IAM users assuming roles, I can require MFA through trust policy conditions using `aws:MultiFactorAuthPresent`, where applicable.

I would also use centralized identity federation and enforce MFA through the identity provider when possible, restrict privileged role access, and monitor role assumptions through CloudTrail.

### 20. How would you implement cross-account role assumption between development, UAT, and production?

I would separate the environments into different AWS accounts and create environment-specific IAM roles.

- Developers would receive access to development resources through development roles.
- UAT deployments would use a dedicated UAT deployment role.
- Production access would be restricted to explicitly authorized principals and deployment processes.

Each target role would have a trust policy specifying who can assume it, and its permissions policy would grant only the actions needed in that environment.

I would use temporary credentials, MFA or federation controls where appropriate, and CloudTrail to audit access.

### 21. How would you use CloudTrail to audit who assumed a role and when?

I would use CloudTrail to inspect the `AssumeRole` API events.

I would review fields such as:

- `eventTime` - when the role was assumed.
- `userIdentity` - the identity that initiated the request.
- `requestParameters.roleArn` - the role being assumed.
- `requestParameters.roleSessionName` - the requested session name.
- `sourceIPAddress` - the source IP address.
- `responseElements` - details returned by the operation, when present.

I would also review subsequent API events performed using the assumed-role session to understand what actions were taken.

## Quick Revision Table

| Topic                 | Key point                                                   |
| --------------------- | ----------------------------------------------------------- |
| IAM user              | Persistent identity, potentially with long-term credentials |
| IAM role              | Assumed identity that provides temporary credentials        |
| STS                   | Issues temporary security credentials                       |
| Least privilege       | Grant only the permissions required                         |
| Explicit deny         | Overrides applicable allows                                 |
| Identity-based policy | Attached to an IAM identity                                 |
| Resource-based policy | Attached to a resource                                      |
| Managed policy        | Standalone and reusable                                     |
| Inline policy         | Embedded in one identity                                    |
| Authentication        | Verifies identity                                           |
| Authorization         | Determines permissions                                      |
| Cross-account access  | Trust policy plus permission to assume the role             |
| EC2 security          | Prefer instance roles over hardcoded access keys            |
| Lambda security       | Use an execution role with scoped permissions               |
| CI/CD security        | Prefer OIDC federation and temporary credentials            |
| CloudTrail            | Records AWS API activity for auditing                       |
