
# Interview Prep: Day 4

Answer format that sounds senior: **direct answer → how it works → concrete example → one trade-off.** Keep each answer to 30–60 seconds spoken.

## Packet paths (know these cold)

```
Inbound:   User → IGW → ALB (public subnets, both AZs) → target SG allows ALB SG → EC2 (private subnet)
Outbound:  EC2 (private) → private route table 0.0.0.0/0 → NAT GW (public subnet) → IGW → internet
Admin:     You → bastion (public subnet, SSH from your IP only) → EC2 (private, SSH from bastion SG only)
```

- Public subnet = route table has `0.0.0.0/0 → Internet Gateway`. Private subnet = no IGW route.
- Return traffic from the internet to the NAT gateway is only allowed for connections started from inside. Nothing can start a connection inward through NAT.

## Interview round

### Q1. What is the difference between a public and a private subnet?

A subnet is public if its route table sends `0.0.0.0/0` to an Internet Gateway; private if it doesn't. A public subnet's instances also need a public/Elastic IP to be reachable. Example: the ALB and bastion sit in public subnets, the web servers in private ones. Trade-off: private is safer but needs a NAT gateway for outbound access, which costs money.

### Q2. NAT gateway vs Internet gateway vs NAT instance?

- **IGW**: two-way door between the VPC and the internet, for resources with public IPs. Free, highly available, managed by AWS.
- **NAT gateway**: lets private instances start outbound connections; blocks inbound-initiated ones. Lives in a public subnet, needs an Elastic IP, is zonal.
- **NAT instance**: an EC2 you manage yourself as a NAT. Cheaper at tiny scale but you handle patching, failover, scaling and source/dest check. NAT gateway is the default answer.

Trade-off: NAT gateway bills hourly plus per GB processed, so it is a common surprise on the bill. Check current pricing on the AWS pricing page.

### Q3. A customer wants web servers that can't be reached from the internet, but users must access the app. What do you build?

Internet-facing ALB in public subnets across at least 2 AZs. Web servers in private subnets in an Auto Scaling Group, registered to the ALB target group. Target SG allows the app port only from the ALB's security group. Outbound updates go through NAT gateways. Admin access via bastion or, better, SSM Session Manager. Trade-off: more moving parts and NAT cost versus servers with public IPs, but the attack surface shrinks to the ALB.

### Q4. Troubleshoot (you answer, then I'll grade it)

You open the ALB DNS name and get a 502/504 or a timeout. The two instances show as running. Walk through, in order, what you check and why.

### Q5. Design / trade-off (you answer, then I'll grade it)

Your NAT gateway bill is too high and one AZ outage would cut outbound access for the whole app. How do you make outbound access highly available, and how do you reduce the cost without giving up security? Mention at least two options.

> Reply with your answers to Q4 and Q5 and I'll score them for correctness and clarity.

## Rapid-fire (answer out loud, then check)

| # | Question | Short answer |
|---|---|---|
| 1 | Security group vs NACL? | SG: instance-level, stateful, allow rules only. NACL: subnet-level, stateless, allow and deny, rules evaluated in number order. |
| 2 | Can instances in a private subnet be targets of an internet-facing ALB? | Yes. ALB nodes live in the public subnets and have private IPs in the VPC, so they reach private targets. |
| 3 | Why must an ALB span 2+ AZs? | HA, and AWS requires subnets in at least two AZs for an ALB. |
| 4 | How do you restrict SSH to private servers to the bastion only? | Private SG inbound 22 with source = bastion's SG (not a CIDR). Bastion SG inbound 22 from your IP only. |
| 5 | Launch template vs launch configuration? | Launch template is current: versioned, supports newer features. Launch configurations are deprecated; use templates. |
| 6 | What does the ASG do when an instance dies? | Marks it unhealthy, terminates it, launches a replacement in an AZ to restore desired capacity. |
| 7 | Does an ASG replace an instance that fails the ALB health check? | Only if the ASG health check type includes ELB. Default is EC2 status checks only. |
| 8 | Min / desired / max capacity? | Min: floor. Max: ceiling. Desired: the target count right now; scaling policies move it between min and max. |
| 9 | Do you need a public IP on a NAT gateway's clients? | No. The NAT gateway holds the Elastic IP; clients keep private IPs. |
| 10 | Why use a bastion over opening SSH on the servers? | One hardened, logged entry point instead of many exposed ones. |
| 11 | Better alternative to a bastion? | SSM Session Manager: no inbound ports, no SSH keys, IAM-based, session logging. Needs the SSM agent, an instance role, and a route to SSM (NAT or VPC endpoints). |
| 12 | Where does the NAT gateway go: public or private subnet? | Public subnet, because it must route to the IGW. |
| 13 | ASG scaling policy types? | Target tracking (simplest), step, simple, scheduled, predictive. |
| 14 | Why not put the key pair `.pem` file permanently on the bastion? | Anyone who compromises the bastion gets the key. Use SSH agent forwarding or SSM instead. |

## SAA-C03-style practice questions

Attempt first, answers are below.

**Q1.** A company runs web servers in private subnets across two Availability Zones. The servers must download OS patches from the internet. No inbound connections from the internet to the servers are allowed. Which solution meets this with the **least operational overhead**?

- A. Deploy a NAT instance in each public subnet and update private route tables.
- B. Deploy a NAT gateway in each AZ's public subnet and point each private route table to its local NAT gateway.
- C. Attach an internet gateway to the private subnets and assign public IPs to the instances.
- D. Create a gateway VPC endpoint for the patch repositories.

**Q2.** A web application runs on EC2 instances in an Auto Scaling Group behind an Application Load Balancer. An instance's application crashes, but the instance passes EC2 status checks, so the ASG does not replace it. The ALB has already stopped sending traffic to it. What should a solutions architect do to have unhealthy instances replaced automatically?

- A. Increase the ASG minimum capacity.
- B. Enable ELB health checks for the Auto Scaling Group.
- C. Shorten the ALB deregistration delay.
- D. Add a CloudWatch alarm that emails the operations team.

**Q3.** An application must be reachable from the internet. For security, the EC2 instances must not have public IP addresses and must be spread across two AZs for high availability. Which design meets the requirements?

- A. Instances in public subnets, ALB in private subnets.
- B. Instances in private subnets, internet-facing ALB in public subnets in both AZs.
- C. Instances in private subnets, internal ALB, and an internet gateway attached to the VPC.
- D. Instances and ALB both in private subnets, with a NAT gateway for inbound traffic.

**Q4.** Administrators need shell access to EC2 instances in private subnets. The security team requires **no inbound ports open on the instances, no SSH key management, and an audit trail of sessions**. What is the best solution?

- A. A bastion host with SSH restricted to the corporate IP range.
- B. Open port 22 on the private instances to the corporate CIDR.
- C. AWS Systems Manager Session Manager with an IAM instance profile.
- D. A VPN connection and a shared key pair for all instances.

**Q5. (Choose two.)** A solutions architect needs to improve the availability of outbound internet access for private subnets in two AZs, which currently use a single NAT gateway in AZ-1. Which actions are required?

- A. Create a NAT gateway in a public subnet in AZ-2.
- B. Create a second internet gateway in AZ-2.
- C. Update the AZ-2 private route table to point `0.0.0.0/0` to the AZ-2 NAT gateway.
- D. Place the second NAT gateway in a private subnet.
- E. Enable cross-zone load balancing on the NAT gateway.

### Answers

**Q1: B.** Keyword: *least operational overhead*. NAT gateway is managed and zonal, so one per AZ with a local route avoids a cross-AZ dependency.
- A: NAT instances need patching, scaling and failover handling.
- C: attaching an IGW route and public IPs makes the subnets public and exposes the servers.
- D: gateway endpoints only cover S3 and DynamoDB, not arbitrary internet repos.

**Q2: B.** By default an ASG uses only EC2 status checks. Turning on the ELB health check type lets the ASG terminate instances the load balancer marks unhealthy. Mind the health check grace period so new instances are not killed while booting.
- A: more capacity doesn't replace the broken one.
- C: deregistration delay only controls draining.
- D: notifies, doesn't remediate.

**Q3: B.** Keywords: *no public IPs*, *internet reachable*. An internet-facing ALB must sit in public subnets in 2+ AZs; targets can be private.
- A: reversed.
- C: an internal ALB has no internet-facing path.
- D: NAT is for outbound only; it can't receive inbound traffic.

**Q4: C.** Keywords: *no inbound ports, no key management, audit trail*. Session Manager uses outbound connections from the SSM agent and IAM for access control, and sessions can be logged to CloudWatch Logs or S3.
- A: needs port 22 on the bastion and key management.
- B: opens port 22 and uses keys.
- D: shared key pair is poor practice and has no per-user audit.

**Q5: A and C.** A NAT gateway is zonal and must be in a public subnet; the AZ-2 route table then has to send `0.0.0.0/0` to it. This also avoids cross-AZ data charges.
- B: a VPC has one IGW and it is already regionally redundant.
- D: NAT gateway in a private subnet can't reach the internet.
- E: cross-zone is a load balancer feature, not a NAT feature.

## Cheat sheet (memorize)

1. Public subnet = route to **IGW**. Private subnet = route to **NAT gateway** (or nothing).
2. NAT gateway: lives in a **public** subnet, **zonal**, needs an EIP. Use one per AZ for HA.
3. ALB: needs subnets in **2+ AZs**, internet-facing ALB goes in **public** subnets, targets can be **private**.
4. SG is **stateful** (allow only); NACL is **stateless** (allow + deny). Reference SGs by SG ID instead of CIDR for tier-to-tier rules.
5. ASG health: default is EC2 checks only. Enable **ELB health checks** to replace instances the load balancer marks unhealthy.

## Break it on purpose (lab extra)

| Change | Expected symptom | How to diagnose |
|---|---|---|
| Delete the private route `0.0.0.0/0 → NAT` | `yum update` / `apt update` hangs on the private instance | Check the private subnet's route table |
| Remove the ALB SG from the target SG's inbound rule | ALB returns 504 / targets unhealthy | Target group health status, target SG |
| Stop the app on one instance | ALB marks it unhealthy; traffic goes to the other | Target group health. ASG replaces it only if ELB health check is on |
| Change the bastion SG source to a wrong IP | SSH to bastion times out | Bastion SG inbound rules, then NACL |
| Health check path wrong (e.g. `/health` not served) | All targets unhealthy, ALB returns 503 | Target group health check path and port |

## Troubleshooting order (use this every time)

DNS → route table → NACL → security group → target service listening on the port → health check config → IAM/policy → evidence (VPC Flow Logs, ALB access logs, CloudWatch).

## Cost checks

NAT gateways, load balancers, EC2, Elastic IPs and EBS keep billing while they exist. Delete in this order: Auto Scaling Group → load balancer → target group → bastion → NAT gateways → release Elastic IPs → VPC (this removes subnets, route tables, IGW, SGs).