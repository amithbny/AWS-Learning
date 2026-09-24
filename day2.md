# AWS VPC Notes

## Overview

A VPC (Virtual Private Cloud) is your own private network inside AWS. It lets you launch resources such as EC2 instances, databases, and load balancers in a logically isolated environment with your own IP ranges, routing rules, and security controls.

Think of it as your own "private internet" inside the larger AWS cloud. You control how resources communicate with each other, with the internet, and with on-premises networks.

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/12cc10b6-724c-42c9-b07b-d8a7ce124e24)

> By default, AWS creates a default VPC in each region for new accounts, but in real projects you usually create custom VPCs for application-specific networking.

## 1. What is a VPC?

A VPC is a logically isolated virtual network in AWS. Within a VPC, you can:

- define the IP address range
- create subnets
- attach gateways and route tables
- enforce security using security groups and NACLs
- connect to other networks using VPN or Direct Connect

A good interview answer is:

> A VPC is a logically isolated section of the AWS cloud where you launch resources in a virtual network you define, with control over IP ranges, subnets, routing, and gateways.

## 2. VPC Components

The main building blocks of a VPC are:

### 2.1 Subnets

A subnet is a range of IP addresses inside a VPC.

- A subnet belongs to only one Availability Zone (AZ)
- A VPC can span multiple AZs
- You place resources in subnets based on their networking needs

### 2.2 IP Addressing

You can assign IPv4 and IPv6 addresses to your VPCs and subnets.

- IPv4 is still the most common
- CIDR notation is used to define ranges
- Example: `10.0.0.0/16`

### 2.3 Route Tables

Route tables determine where network traffic is directed.

- A subnet is associated with a route table
- Traffic can be routed to the internet, NAT gateway, peering connection, or other network devices

### 2.4 Internet Gateway (IGW)

An Internet Gateway allows a VPC to communicate with the public internet.

- Public subnets use an IGW
- Instances in public subnets can receive traffic from the internet if security allows

### 2.5 NAT Gateway

A NAT Gateway allows private subnet instances to access the internet for outbound traffic without exposing the instances to inbound internet connections.

- Usually placed in a public subnet
- Private subnets route outbound internet traffic to the NAT

### 2.6 Security Groups

A security group is a virtual firewall at the instance level.

- Stateful
- Only allows rules
- Evaluates all rules together
- Used for EC2, RDS, ALB, and many other services

### 2.7 Network ACLs (NACLs)

A Network ACL is a stateless firewall at the subnet level.

- Operates on subnets, not individual instances
- Supports allow and deny rules
- Rules are evaluated in order
- Common for controlling traffic entering and leaving a subnet

### 2.8 VPC Endpoints

A VPC endpoint lets private instances access AWS services without using the internet or NAT.

Example:

- S3 gateway endpoint
- Interface endpoint for other AWS services

### 2.9 VPC Peering

A VPC peering connection connects two VPCs so they can route traffic between each other.

- One-to-one connectivity
- Not transitive
- CIDR ranges cannot overlap

### 2.10 Transit Gateway

A Transit Gateway is a central hub for connecting many VPCs and hybrid networks.

- Better than using many peer connections
- Useful in large AWS environments

### 2.11 Flow Logs

VPC Flow Logs capture information about traffic to and from network interfaces.

- Useful for troubleshooting and security monitoring
- Can be sent to CloudWatch Logs or S3

### 2.12 VPN Connections

A VPN connects your VPC to your on-premises network over an encrypted tunnel.

- Site-to-site VPN
- Used for hybrid cloud connectivity

## 3. Public vs Private Subnet

| Subnet Type    | Route to Internet         | Typical Use                    |
| -------------- | ------------------------- | ------------------------------ |
| Public subnet  | Yes, via Internet Gateway | Web servers, load balancers    |
| Private subnet | No direct internet route  | Databases, application servers |

A subnet is public when its route table contains a route like:

```text
0.0.0.0/0 -> igw-xxxx
```

A private subnet usually routes traffic to a NAT Gateway instead of the internet directly.

## 4. Default VPC

Each AWS account gets a default VPC in each region.

It includes:

- default subnets
- default route tables
- default security groups
- default network ACLs

You can delete it, but custom VPCs are recommended for real workloads because default VPCs are only meant as a starting point.

## 5. CIDR Basics

CIDR is a method of defining a range of IP addresses.

Examples:

- `10.0.0.0/16`
- `172.31.0.0/16`
- `192.168.1.0/24`

In a VPC interview, remember:

- smallest VPC size is typically `/28` (16 IP addresses)
- largest common VPC size is `/16`
- each subnet reserves a few IP addresses for AWS use

## 6. Security Groups vs NACLs

| Feature            | Security Group                   | NACL                         |
| ------------------ | -------------------------------- | ---------------------------- |
| Scope              | Instance level                   | Subnet level                 |
| Stateful/Stateless | Stateful                         | Stateless                    |
| Rules              | Allow only                       | Allow and deny               |
| Evaluation         | All rules are checked as a group | Rules are evaluated in order |

Important points:

- Security groups are the main firewall for instances
- NACLs provide an extra layer of control at the subnet level
- NACLs require you to allow return traffic on ephemeral ports

## 7. NAT Gateway vs Internet Gateway

### Internet Gateway

- Provides connectivity between a VPC and the public internet
- Used by public subnets
- Supports two-way communication for public resources

### NAT Gateway

- Enables outbound internet access for private instances
- Prevents inbound traffic from the internet by default
- Best used when private resources need software updates or package downloads

## 8. Public IP vs Elastic IP

- Public IP: temporary and may change when the instance stops or restarts
- Elastic IP: static public IP you reserve and keep until you release it

Elastic IPs are useful when a resource must keep a predictable public address.

## 9. NAT Gateway vs NAT Instance

| Option       | Managed by AWS | High availability     | Maintenance                     |
| ------------ | -------------- | --------------------- | ------------------------------- |
| NAT Gateway  | Yes            | Yes, within an AZ     | AWS-managed                     |
| NAT Instance | No             | Depends on your setup | You manage patching and scaling |

In most AWS workloads, NAT Gateway is preferred because it is simpler and more reliable.

## 10. Typical Scenario Questions

### Q1. What is a VPC?

A VPC is a logically isolated virtual network in AWS where you launch resources and control IP ranges, subnets, route tables, and security rules.

### Q2. What is the default VPC? Can you delete it?

AWS creates one default VPC per region in each account. It contains default subnets and basic networking settings. It can be deleted, but custom VPCs are usually better for production workloads.

### Q3. What is CIDR? What are the smallest and largest VPC sizes?

CIDR is the standard notation for IP ranges. A VPC uses CIDR blocks to define address ranges. The common smallest size is `/28` and the largest is `/16` for a VPC.

### Q4. Is a subnet tied to an AZ or a region?

A subnet is always associated with exactly one Availability Zone. A VPC can span multiple AZs, but each subnet lives in a single AZ.

### Q5. Can you have multiple VPCs in one account?

Yes. An AWS account can have multiple VPCs in a region, and the default quota is often five per region, although it can be increased.

### Q6. What are the main components of a VPC?

Main components include VPCs, subnets, route tables, Internet Gateway, NAT Gateway, security groups, NACLs, and optionally VPN or peering connections.

### Q7. What makes a subnet public?

A subnet is public when its route table points to an Internet Gateway using `0.0.0.0/0 -> IGW` and the instance has a public IP or Elastic IP.

### Q8. Difference between IGW and NAT Gateway?

An IGW provides inbound and outbound internet access for public resources. A NAT Gateway provides outbound-only internet access for private resources.

### Q9. Where do you place the NAT Gateway, and why?

In a public subnet, because it must be able to reach the Internet Gateway.

### Q10. Difference between public IP and Elastic IP?

A public IP is temporary and may change. An Elastic IP is a static public IP that you own until you release it.

### Q11. NAT Gateway vs NAT instance?

A NAT Gateway is AWS-managed and easier to operate. A NAT instance is an EC2 instance you manage yourself, including patching and scaling.

### Q12. Difference between SG and NACL?

Security groups work at the instance level and are stateful. NACLs work at the subnet level and are stateless. Security groups are allow-only; NACLs can allow or deny based on order.

### Q13. What does stateful mean?

Stateful firewalls remember connection responses automatically. If inbound traffic is allowed, the matching return traffic is allowed without a second rule.

### Q14. Why do NACLs need ephemeral ports open?

Because the response from a server often uses a random high port such as `1024-65535`. Since NACLs are stateless, that return traffic must be explicitly allowed.

### Q15. Can you deny a specific IP with a security group?

No. Security groups only support allow rules. To block a specific IP, you typically use a NACL deny rule or a WAF or another layer of filtering.

### Q16. Private instances need to download updates. What do you do?

Create a NAT Gateway in a public subnet and add a route in the private subnet route table sending `0.0.0.0/0` to the NAT Gateway.

### Q17. Explain the traffic path from a private EC2 to the internet.

The instance sends traffic to its route table, which forwards it to the NAT Gateway in a public subnet, and the NAT sends it through the Internet Gateway to the internet.

### Q18. How do you securely SSH into instances in a private subnet?

Use a bastion host in a public subnet or use AWS Systems Manager Session Manager, which avoids exposing SSH ports publicly.

### Q19. NAT Gateway or bastion host?

A NAT Gateway is for outbound internet access from private resources. A bastion host is for secure administrative access into private resources.

### Q20. How do you reach S3 from a private subnet without internet or NAT?

Use a VPC endpoint for S3, which allows private resources to access AWS services without internet access or a NAT Gateway.

### Q21. Design a highly available multi-tier web app.

Use public subnets for web servers behind a load balancer, private subnets for app servers and databases, and a NAT Gateway in each AZ. Use security groups to restrict communication between layers.

### Q22. An instance in a public subnet is not reachable from the internet. How do you troubleshoot?

Check the following in order:

- Does the instance have a public or Elastic IP?
- Is the Internet Gateway attached to the VPC?
- Does the subnet route table have `0.0.0.0/0 -> IGW`?
- Do the NACLs allow inbound and outbound traffic?
- Do the security groups allow the port?
- Is the application listening on the expected port?

### Q23. Two instances in the same VPC can't talk. Why?

The local route within the same VPC should work automatically. In practice, the issue is usually a security group or NACL rule blocking the required port or source IP.

### Q24. VPC peering vs Transit Gateway?

VPC peering connects two VPCs directly and is non-transitive. Transit Gateway is a central hub used when there are many VPCs or hybrid network connections.

### Q25. How do you monitor traffic in a VPC?

Use VPC Flow Logs. They help you see accepted and rejected traffic and are useful for troubleshooting and security analysis.

### Q26. VPC vs VPN?

A VPC is your AWS virtual network. A VPN is the encrypted connection used to connect your on-premises network to that VPC.

## 11. Useful Resources

- AWS VPC Documentation: https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- Private subnets with NAT: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/89d8316e-7b70-4821-a6bf-67d1dcc4d2fb)

## 12. Quick Revision Summary

- VPC = private network in AWS
- Subnet = IP range in one AZ
- IGW = internet access for public resources
- NAT Gateway = outbound internet access for private resources
- Security Group = instance firewall
- NACL = subnet firewall
- VPC endpoint = private access to AWS services
- Flow Logs = traffic visibility
- VPN = hybrid connectivity to on-premises networks

This structure makes the file easier to study, review, and use during interviews.
