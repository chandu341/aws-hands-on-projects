# README.md

AWS Hands-On Learning Repository

---

## Overview

This repository documents my hands-on AWS learning journey, covering practical implementations of core AWS networking, security, storage, and application services. Each task includes complete step-by-step implementations, architecture diagrams, validation procedures, production best practices, and real-world use cases.

**Purpose:** Build a comprehensive portfolio of AWS solutions while developing deep understanding of AWS service interactions, networking patterns, and architectural best practices.

---

## Repository Structure

```
aws-hands-on-projects/
├── README.md                          # This file
├── TASK-TRACKER.md                    # Central inventory of all tasks
├── SOURCE-INVENTORY.md                # Mapping of source files to tasks
├── DAILY-LOG.md                       # Learning progress and notes
├── networking/                        # VPC, Routing, Connectivity
│   ├── vpc-creation/
│   ├── vpc-subnets-routes-igw/
│   ├── public-private-subnets-nat/
│   └── vpc-peering-multiregion/
├── security/                          # Security Groups, NACLs, Firewalls
│   ├── sg-nacl/
│   └── network-firewall/
├── compute-and-storage/               # EC2, EBS, Disk Management
│   └── disk-management/
├── application-services/              # S3, VPC Endpoints, IAM
│   └── vpc-endpoint-s3/
└── daily-logs/                        # Detailed learning logs
```

---

## AWS Services Covered

### Networking (4 tasks)
- **VPC Creation & Management** - Multi-AZ VPC design with public/private subnets
- **Internet Gateway & Route Tables** - Enabling internet connectivity for public instances
- **NAT Gateway & Private Subnet Internet Access** - Outbound internet for private instances
- **VPC Peering (Multi-Region)** - Cross-region and multi-VPC connectivity

### Security (2 tasks in progress)
- **Security Groups & Network ACLs** - Inbound/outbound traffic filtering
- **AWS Network Firewall** - Network-level intrusion detection and prevention

### Compute & Storage (1 task in progress)
- **EC2 Disk Management** - EBS volumes, partitioning, and storage optimization

### Application Services (1 task)
- **VPC Endpoints & S3 Access** - Private connectivity to AWS services, cost optimization

---

## Quick Navigation

### Foundational Networking
Start here to build networking fundamentals:

1. **[VPC Creation with Multi-AZ Setup](./networking/vpc-creation/README.md)** — AWS-001
   - Create isolated VPC with 3 AZs, 6 subnets (3 public + 3 private)
   - Understand subnet design and public IP auto-assignment
   - **Time:** ~30 minutes | **Cost:** Free

2. **[VPC Subnets, Route Tables & IGW](./networking/vpc-subnets-routes-igw/README.md)** — AWS-002
   - Configure route tables and Internet Gateway
   - Launch EC2 instance with nginx web server
   - Test internet connectivity
   - **Time:** ~1 hour | **Cost:** ~$0.50-2.00

3. **[Public & Private Subnets with NAT](./networking/public-private-subnets-nat/README.md)** — AWS-003
   - Implement bastion host (jumphost) pattern
   - Deploy NAT Gateway for private instance internet access
   - Test connectivity through bastion
   - **Time:** ~1.5 hours | **Cost:** ~$6-7

### Advanced Networking & Service Integration
Build on foundational knowledge:

4. **[VPC Peering Multi-Region](./networking/vpc-peering-multiregion/README.md)** — AWS-006
   - Configure peering between 3 VPCs across 2 regions
   - Understand CIDR planning and route table configuration
   - Test full mesh connectivity
   - **Time:** ~2 hours | **Cost:** ~$27

5. **[VPC Endpoints & S3 Access](./application-services/vpc-endpoint-s3/README.md)** — AWS-005
   - Differentiate between IAM permissions and network connectivity
   - Use VPC Gateway Endpoint for private S3 access
   - Cost optimization: VPC Endpoint vs. NAT Gateway
   - **Time:** ~1.5 hours | **Cost:** Free (endpoint) vs. $77/month (NAT)

---

## Learning Progression

### Tier 1: AWS Networking Fundamentals (Tasks AWS-001 to AWS-003)
**Skills:** VPC design, subnets, routing, internet connectivity, bastion hosts

**Learning Outcomes:**
- Understand VPC as AWS's core networking foundation
- Master public and private subnet patterns
- Implement secure access patterns (bastion hosts)
- Troubleshoot connectivity issues
- Design for high availability (multi-AZ)

**Prerequisite Knowledge:** Basic AWS console navigation, EC2 instance basics

**Time Commitment:** ~2.5 hours | **Total Cost:** ~$7-10

---

### Tier 2: Multi-VPC Connectivity & Service Integration (Tasks AWS-005 to AWS-006)
**Skills:** VPC peering, VPC endpoints, IAM roles, network architecture

**Learning Outcomes:**
- Design multi-VPC architectures
- Understand VPC peering limitations and alternatives
- Implement VPC endpoints for cost-efficient AWS service access
- Master the relationship between permissions and connectivity
- Plan CIDR blocks for multi-VPC environments

**Prerequisite Knowledge:** Tier 1 completion

**Time Commitment:** ~3.5 hours | **Total Cost:** ~$34

---

### Tier 3: Security & Advanced Networking (Tasks AWS-004, AWS-007, AWS-008)
**Skills:** Network ACLs, security groups, firewalls, storage management

**Learning Outcomes:**
- Implement defense-in-depth (security groups + NACLs)
- Configure network firewalls for intrusion detection
- Master EBS and storage optimization
- Design production-grade network security

**Prerequisite Knowledge:** Tier 1 and Tier 2 completion

**Status:** In progress

---

## Key Concepts

### Recurring Lessons

#### 1. **Permissions ≠ Connectivity**
A critical lesson from AWS-005 (VPC Endpoints):
- IAM roles control "what" you can do
- Network configuration (IGW, NAT, endpoints, routing) controls "how" to reach resources
- Both are required for successful access

#### 2. **Route Tables Control Traffic Flow**
Across all networking tasks:
- Route tables are the "traffic managers" of your VPC
- Same subnet behaves differently based on route table
- Route tables make subnets "public" (IGW route) or "private" (no IGW route)

#### 3. **Security is Layered**
From AWS-003 (Bastion Pattern):
- Security groups filter at instance level
- NACLs filter at subnet level
- Route tables control traffic direction
- All three work together (defense in depth)

#### 4. **CIDR Planning is Critical**
From AWS-006 (VPC Peering):
- Non-overlapping CIDR blocks are prerequisites for peering
- Transitive routing doesn't work with VPC peering
- Proper planning prevents future architectural constraints

---

## Production Best Practices Applied

### High Availability
- ✅ Multi-AZ deployment (AWS-001)
- ✅ Bastion hosts in multiple AZs (AWS-003)
- ✅ Cross-region failover (AWS-006)

### Security
- ✅ Public/private subnet separation (AWS-001, AWS-003)
- ✅ Principle of least privilege (security groups)
- ✅ Network layering (security groups + NACLs)
- ✅ No internet exposure for sensitive resources (VPC endpoints)

### Cost Optimization
- ✅ VPC endpoints vs. NAT Gateways (AWS-005): $77/month savings
- ✅ Free tier instance usage where possible
- ✅ Proper resource cleanup to avoid idle charges

### Operational Excellence
- ✅ Detailed documentation for reproducibility
- ✅ Validation steps for each configuration
- ✅ Troubleshooting guides included
- ✅ Real-world architectural patterns explained

---

## How to Use This Repository

### For AWS Learning
1. **Start with AWS-001** to understand VPC fundamentals
2. **Follow the suggested progression** (Tier 1 → Tier 2 → Tier 3)
3. **Read each task README completely** before implementation
4. **Follow step-by-step implementation** without skipping
5. **Validate each step** using provided validation procedures
6. **Study interview questions** at the end of each task

### For Architecture Reference
- **Multi-VPC Setup?** → See AWS-006 (VPC Peering)
- **Private Subnet Internet?** → See AWS-003 (NAT Gateway pattern)
- **Bastion Host?** → See AWS-003 (jumphost implementation)
- **Private AWS Service Access?** → See AWS-005 (VPC Endpoints)

### For Interviews
Each task includes interview questions at three levels:
- **Foundational:** Core concepts
- **Intermediate:** Practical understanding
- **Advanced:** Architectural decisions

---

## Cost Summary

| Task | Service | Est. Cost |
|------|---------|-----------|
| AWS-001 | VPC, Subnets | Free |
| AWS-002 | EC2, IGW, Route Tables | $0.50-2.00 |
| AWS-003 | EC2, NAT Gateway | $6-7 (24 hrs) |
| AWS-004 | Security Groups, NACLs | Free |
| AWS-005 | EC2, VPC Endpoint, S3 | Free (vs. $77/month with NAT) |
| AWS-006 | EC2 (3x), VPC Peering | ~$27 (24 hrs, 3 instances) |
| AWS-007 | EBS Volumes | TBD |
| AWS-008 | Network Firewall | TBD |

**Total Cost for Complete Repository:** ~$40-50 (when run sequentially)

---

## Documentation Files

### TASK-TRACKER.md
Central inventory of all AWS tasks:
- Task ID, name, AWS services
- Implementation and documentation status
- Evidence availability
- Links to individual task READMEs

### SOURCE-INVENTORY.md
Maps original learning materials to repository tasks:
- Original file names and locations
- Corresponding repository task
- Screenshots and artifacts included

### DAILY-LOG.md
Learning progress and insights:
- Date-based entries
- Learnings and challenges
- Production insights

---

## Related AWS Services (Not Yet Covered)

- **Auto Scaling:** EC2 Auto Scaling Groups, Application Load Balancer
- **Databases:** RDS, DynamoDB, RDS Multi-AZ
- **Storage:** EFS, Backup & Restore
- **Monitoring:** CloudWatch, VPC Flow Logs, CloudTrail
- **Advanced Networking:** Transit Gateway, AWS PrivateLink
- **Hybrid Connectivity:** Site-to-Site VPN, Direct Connect

---

## Resources

### Official AWS Documentation
- [VPC User Guide](https://docs.aws.amazon.com/vpc/)
- [EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)

### AWS Certification Preparation
- **AWS Solutions Architect Associate:** Focus on Tier 1 + Tier 2
- **AWS SysOps Administrator:** Focus on Tier 1 + Tier 2 + security
- **AWS Security Specialty:** Focus on Tier 3

### Learning Approach
Each task follows this structure:
1. **Objective:** What you're building and why
2. **Architecture Overview:** Diagram of the setup
3. **Configuration Details:** All resources and settings
4. **Implementation Steps:** Step-by-step procedures
5. **Validation Steps:** Verify everything works
6. **What I Learned:** Key takeaways
7. **Production Use Cases:** Real-world applications
8. **Security Considerations:** Best practices
9. **Cost Analysis:** Pricing breakdown
10. **Interview Questions:** Three difficulty levels
11. **Resource Cleanup:** Safe deletion procedures

---

## Getting Started

### Option 1: Complete Linear Path (Recommended for Learning)
1. Read AWS-001 completely
2. Implement AWS-001
3. Move to AWS-002, repeat
4. Follow this pattern for all tasks in Tier 1 → Tier 2 → Tier 3

### Option 2: Deep Dive on Specific Topic
- Interested in **Multi-Region?** → Start with AWS-006
- Focused on **Security?** → Start with AWS-003
- Learning **Cost Optimization?** → Start with AWS-005

### Option 3: Interview Preparation
- Read all READMEs, especially interview questions
- Practice explaining concepts without reading
- Implement tasks to build hands-on confidence

---

## Implementation Status

| Task | Documentation | Implementation | Validation |
|------|---|---|---|
| AWS-001 | ✅ Complete | ✅ Done | ✅ Verified |
| AWS-002 | ✅ Complete | ✅ Done | ✅ Verified |
| AWS-003 | ✅ Complete | ✅ Done | ✅ Verified |
| AWS-004 | ⏳ In Progress | ⏳ In Progress | ⏳ Pending |
| AWS-005 | ✅ Complete | ✅ Done | ✅ Verified |
| AWS-006 | ✅ Complete | ✅ Done | ✅ Verified |
| AWS-007 | ⏳ In Progress | ⏳ In Progress | ⏳ Pending |
| AWS-008 | ⏳ In Progress | ⏳ In Progress | ⏳ Pending |

---

## Contributing to This Repository

As this is a personal learning repository, all content reflects hands-on AWS experience. To extend:

1. Complete additional AWS service tasks
2. Document lessons learned
3. Add production best practices
4. Include real-world incident responses
5. Expand security and compliance sections

---

## Notes

- All documentation is based on actual hands-on implementation
- Screenshots and configurations reflect real AWS environments
- Cost estimates are based on AWS pricing as of October 2026
- Interview questions are typical AWS Solutions Architect exam content

---

## License

This repository documents personal AWS learning. Content is available for educational and portfolio purposes.

---

## Contact & Questions

For questions about specific tasks or AWS concepts covered:
- Review the relevant task README
- Check the interview questions section
- Consult AWS official documentation
- Explore real-world use cases section

---

**Last Updated:** October 13, 2026  
**Total Tasks:** 8 (6 complete, 2 in progress)  
**Total Implementation Time:** ~10 hours  
**Total Cost:** ~$40-50
