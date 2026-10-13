# Daily Learning Log

**Repository:** aws-hands-on-projects  
**Purpose:** Track learning progress, insights, and challenges across AWS hands-on exercises

---

## Learning Progress Summary

| Date | Task | AWS Services | Status | Time Spent | Key Learnings | Challenges |
|------|------|--------------|--------|-----------|---------------|-----------|
| 2026-10-06 | AWS-001 | VPC, Subnets | ✅ Complete | ~30 min | Multi-AZ design, subnet CIDR planning | None |
| 2026-10-07 | AWS-002 | IGW, Route Tables | ✅ Complete | ~1 hour | Routing fundamentals, public subnet definition | Bastion access |
| 2026-10-08 | AWS-003 | NAT, Bastion | ✅ Complete | ~1.5 hours | Bastion pattern, private internet access | NAT Gateway costs |
| 2026-10-08 | AWS-005 | VPC Endpoints, S3 | ✅ Complete | ~1.5 hours | IAM ≠ Connectivity, cost savings | Endpoint policies |
| 2026-10-08 | AWS-006 | VPC Peering | ✅ Complete | ~2 hours | Multi-region setup, CIDR planning | Cross-region latency |
| TBD | AWS-004 | SG, NACLs | ⏳ In Progress | - | - | - |
| TBD | AWS-007 | Disk Management | ⏳ In Progress | - | - | - |
| TBD | AWS-008 | Network Firewall | ⏳ In Progress | - | - | - |

---

## Detailed Daily Entries

### October 6, 2026 - VPC Creation (AWS-001)

**Objective:** Create multi-AZ VPC with public and private subnets

**Work Completed:**
- Created Sandpit VPC with custom CIDR block
- Set up 3 availability zones
- Created 3 public subnets + 3 private subnets (6 total)
- Enabled auto-assign public IPv4 on public subnets

**Key Learnings:**
1. **VPC Design Philosophy:** A VPC is your isolated network; all AWS resources live in a VPC
2. **Availability Zones:** Multi-AZ deployment from the start ensures high availability
3. **Subnet Planning:** CIDR block sizing is critical; `/16` allows room for growth
4. **Public vs Private:** Determined by route table, not subnet properties

**Challenges & Solutions:**
- None encountered; straightforward VPC creation

**Time Spent:** ~30 minutes

**Cost:** Free (no running resources)

**Confidence Level:** ⭐⭐⭐⭐⭐ (Foundational understanding achieved)

---

### October 7, 2026 - IGW & Route Tables (AWS-002)

**Objective:** Enable internet access for public instances using IGW and route tables

**Work Completed:**
- Created Internet Gateway and attached to VPC
- Created custom route table with IGW route (0.0.0.0/0 → IGW)
- Associated route table with all 3 public subnets
- Launched EC2 instance with nginx
- Verified internet connectivity and web server access

**Key Learnings:**
1. **Route Tables Control Traffic:** Without IGW route, subnet is "private" regardless of name
2. **Longest Prefix Match:** AWS uses most specific route; local routes take precedence
3. **Security Groups vs Route Tables:** Different layers—instance-level vs. network-level
4. **nginx Testing:** Simple web server for verifying connectivity

**Challenges & Solutions:**
- **Challenge:** Initially unclear how to connect to instance from internet
- **Solution:** Understood auto-assign public IPs + IGW route + security group inbound rules
- **Challenge:** nginx default page didn't display
- **Solution:** Updated security group to allow HTTP (port 80) inbound

**Time Spent:** ~1 hour

**Cost:** ~$0.50-2.00 (1 t2.micro instance for ~2 hours)

**Confidence Level:** ⭐⭐⭐⭐⭐ (Clear understanding of routing)

---

### October 8, 2026 - Bastion Pattern & NAT (AWS-003)

**Objective:** Implement bastion host pattern for private instance access; enable private subnet internet via NAT

**Work Completed:**
- Created demo (private) subnet
- Launched public server (bastion) in public subnet
- Launched private server in private subnet
- Created NAT Gateway with Elastic IP in public subnet
- Configured route table for private subnet (0.0.0.0/0 → NAT)
- Tested SSH through bastion to private instance
- Verified internet access from private instance through NAT

**Key Learnings:**
1. **Bastion Host Pattern:** Industry-standard for accessing private resources
   - Single entry point for operators
   - Audit trail for all access
   - Can implement additional security (IP whitelisting, logging)

2. **NAT Gateway vs. NAT Instance:**
   - NAT Gateway: Fully managed, no configuration, costs money
   - NAT Instance: Self-managed, cheaper initially, complex setup
   - NAT Gateway is preferred for production

3. **Security Groups Enable Bastion Pattern:**
   - Public SG: Allow SSH from 0.0.0.0/0
   - Private SG: Allow SSH from Public SG only
   - This creates security boundary

4. **Elastic IPs:**
   - Must be allocated even if used only for NAT
   - Cost money if not associated
   - Required for NAT Gateway to work

**Challenges & Solutions:**
- **Challenge:** Private instance couldn't access internet initially
  - Root Cause: No NAT Gateway; private route table didn't have internet route
  - Solution: Created NAT Gateway, added route to route table
  
- **Challenge:** SSH from bastion to private instance failed initially
  - Root Cause: Security group didn't allow SSH from bastion
  - Solution: Updated private security group to allow SSH from public security group

**Time Spent:** ~1.5 hours

**Cost:** ~$6-7 for 24 hours (NAT Gateway ~$1.08 + data transfer ~$4.50 + EC2 ~$0.50)

**Confidence Level:** ⭐⭐⭐⭐⭐ (Mastered bastion pattern)

**Production Notes:**
- NAT Gateway cost for high-volume workloads can exceed $500/month
- Consider VPC Endpoints for AWS service access (no data transfer costs)

---

### October 8, 2026 - VPC Endpoints & S3 (AWS-005)

**Objective:** Demonstrate private EC2 access to S3 using VPC Endpoint; highlight IAM vs. network connectivity

**Work Completed:**
- Created S3 bucket with Block Public Access enabled
- Created IAM role with S3 read-only permissions
- Launched EC2 in public subnet with IAM role
- Launched EC2 in private subnet with IAM role
- Tested S3 access from public instance (success—via internet gateway)
- Tested S3 access from private instance without endpoint (failure—no network path)
- Created S3 VPC Gateway Endpoint
- Associated endpoint with private route table
- Tested S3 access from private instance through endpoint (success)

**Key Learnings:**
1. **Critical Lesson: IAM ≠ Network Connectivity**
   - IAM role provides "permission" to access S3
   - Network path provides "connectivity" to S3
   - Both required; one alone is insufficient
   - Very common production troubleshooting issue

2. **VPC Endpoints Cost Optimization:**
   - **NAT Gateway approach:** $0.045/hour + $0.045/GB = ~$77/month for 1TB
   - **VPC Endpoint approach:** Free (gateway type) + no data transfer = $0
   - **Savings:** $77/month for typical workload

3. **Gateway Endpoints (S3, DynamoDB):**
   - Free (no hourly charge)
   - Integrated into route tables
   - Simple configuration
   - Highly efficient

4. **Interface Endpoints (other services):**
   - Hourly charge ($0.007-0.014/hour)
   - ENI-based
   - More complex configuration
   - Necessary for services like RDS, SNS, etc.

**Challenges & Solutions:**
- **Challenge:** Private instance couldn't access S3 even with IAM role
  - Root Cause: IAM permissions alone aren't sufficient; no network path
  - Solution: Created VPC Gateway Endpoint (provided network path)
  - Lesson: Checked security groups, NACLs, route tables, IAM role—all required

- **Challenge:** Understanding when to use NAT vs. Endpoint
  - NAT: General internet access (updates, external APIs)
  - Endpoint: AWS service access (S3, DynamoDB, SNS, etc.)
  - Decision: Use endpoint for AWS services; NAT for general internet

**Time Spent:** ~1.5 hours

**Cost Analysis:**
- If using NAT: ~$0.15 (NAT gateway operation) + ~$0.45 (10GB data) = ~$0.60/day
- If using VPC Endpoint: $0 (endpoint is free)
- **Recommendation:** VPC Endpoints for AWS service access

**Confidence Level:** ⭐⭐⭐⭐⭐ (Critical AWS principle mastered)

**Interview Insight:**
- This is a frequently asked question: "You have an IAM role with S3 permissions but private instance can't access S3. Why?"
- Answer: IAM provides permissions; network path requires VPC Endpoint or NAT Gateway

---

### October 8, 2026 - VPC Peering Multi-Region (AWS-006)

**Objective:** Configure cross-region and same-region VPC peering; establish mesh connectivity between 3 VPCs

**Work Completed:**
- Created 3 VPCs: DB (N.Virginia 12.1.0.0/16), App (Hyderabad 12.2.0.0/16), Web (Hyderabad 12.3.0.0/16)
- Launched EC2 in each VPC
- Created 3 VPC peering connections:
  - DB ↔ App (cross-region)
  - App ↔ Web (same region)
  - DB ↔ Web (cross-region)
- Updated route tables on all 3 VPCs
- Tested full mesh connectivity (all servers reachable from all servers)

**Key Learnings:**
1. **VPC Peering Fundamentals:**
   - Peer-to-peer connections (not transitive)
   - Each peering is independent; no transitive routing
   - Example: A→B and B→C does NOT enable A→C; need A→C peering

2. **CIDR Block Planning is Critical:**
   - Non-overlapping blocks are prerequisite
   - 12.0.0.0/8 range used (reserved for private)
   - Proper separation: 12.1.0.0, 12.2.0.0, 12.3.0.0

3. **Cross-Region vs. Same-Region:**
   - **Same-region:** Free, low latency
   - **Cross-region:** $0.01/GB data transfer, higher latency
   - Architecture decision based on use case

4. **Route Table Configuration:**
   - Each VPC needs routes to other VPCs through peering connections
   - Route: 12.2.0.0/16 → pcx-xxxxx (DB route table)
   - Route: 12.1.0.0/16 → pcx-xxxxx (App route table)
   - Etc. for each peering connection

5. **Scalability Considerations:**
   - 3 VPCs = 3 peering connections
   - 10 VPCs = 45 peering connections (N * (N-1) / 2)
   - At scale: Use AWS Transit Gateway instead

**Challenges & Solutions:**
- **Challenge:** Understanding peering acceptance process
  - Cross-region peering requires acceptance in peer region
  - Solution: Clear documentation of accept workflow

- **Challenge:** Route table configuration complexity
  - Need routes in ALL VPCs for full connectivity
  - Solution: Systematic approach—one VPC at a time

- **Challenge:** Testing cross-region connectivity
  - Initial concern about latency
  - Reality: Acceptable for most use cases; comparable to NAT Gateway

**Time Spent:** ~2 hours (distributed across multiple regions)

**Cost:** ~$27 for 24 hours
- 3 x t2.micro: ~$0.83/day
- Cross-region data: ~$25-26/day (depends on traffic volume)

**Confidence Level:** ⭐⭐⭐⭐ (Strong understanding; ready for production design)

**Production Notes:**
- VPC peering is good for 2-5 VPCs; use Transit Gateway for 10+
- Cross-region peering data costs can add up; consider locality
- Alternative: AWS PrivateLink for SaaS integrations, Transit Gateway for hub-and-spoke

**Interview Insight:**
- Common question: "How would you connect 10 VPCs across 3 regions?"
- Answer: Transit Gateway hub-and-spoke (not individual peering connections)

---

## Key Concepts Summary

### Recurring Patterns
1. **IAM + Network = Access**
   - IAM role: permission level
   - Network configuration: connectivity level
   - Both required

2. **Route Tables Control Everything**
   - Define "public" (has IGW route) vs. "private" (no IGW route)
   - Traffic follows route rules
   - Longest prefix match wins

3. **Security is Layered**
   - Security Groups: instance level
   - NACLs: subnet level (not yet covered)
   - Route Tables: network level
   - All three provide defense in depth

4. **Cost Optimization Opportunities**
   - VPC Endpoints vs. NAT Gateways: $77/month savings
   - Same-region vs. cross-region: peering costs
   - Right-sizing instance types: t2.micro for learning

### Production Patterns Learned
1. **Bastion Host Pattern:** Single entry point for security
2. **Private Subnet with NAT:** Outbound internet for private resources
3. **VPC Endpoints:** Cost-efficient AWS service access
4. **VPC Peering:** Multi-VPC connectivity
5. **Multi-AZ Deployment:** High availability from the start

---

## Challenges & Solutions Log

### Technical Challenges

| Challenge | Root Cause | Solution | Learned |
|-----------|-----------|----------|---------|
| Can't SSH to instance | Security group didn't allow SSH | Added SSH (22) inbound rule | Security groups are critical |
| Private instance can't reach internet | No NAT, no route to IGW | Created NAT + route | Network config required |
| Private instance can't access S3 | No VPC Endpoint, no network path | Created Gateway Endpoint | IAM ≠ Network |
| Cross-region peering pending | Not accepted on peer side | Accepted in peer region | Peering requires acceptance |

### Conceptual Challenges

| Concept | Initial Confusion | Clarification | Resolution |
|---------|------------------|---------------|-----------|
| Public vs Private Subnet | Thought based on location | Based on route table (IGW route) | Study route tables |
| VPC Peering Non-Transitive | Expected A→B→C to work | Each peering independent | Use Transit Gateway at scale |
| NAT vs VPC Endpoint | Thought interchangeable | Different use cases | NAT: internet; Endpoint: AWS services |
| CIDR Block Planning | Thought not important | Prevents peering failures | Plan upfront |

---

## Skills Development

### AWS Networking Fundamentals ✅
- [x] VPC creation and design
- [x] Subnet planning (public/private)
- [x] Internet Gateway attachment
- [x] Route table configuration
- [x] NAT Gateway setup
- [x] Bastion host pattern
- [x] VPC Peering (same + cross-region)

### AWS Security (In Progress)
- [ ] Security Groups (deep understanding)
- [ ] Network ACLs
- [ ] AWS Network Firewall
- [ ] Security group rule optimization

### AWS Services Integration (Partial)
- [x] VPC Endpoints (Gateway type)
- [ ] VPC Endpoints (Interface type)
- [ ] IAM role attachment to EC2
- [ ] S3 bucket policies

---

## Next Learning Priorities

### Tier 1: Security & Advanced Networking
1. **AWS-004: Security Groups & NACLs** — Defense in depth
2. **AWS-007: EC2 Disk Management** — Storage optimization
3. **AWS-008: Network Firewall** — Advanced security

### Tier 2: Production Patterns
1. Transit Gateway for multi-VPC at scale
2. VPC Endpoints (Interface type) for other AWS services
3. AWS PrivateLink for hybrid connectivity

### Tier 3: Real-World Scenarios
1. Disaster recovery with cross-region failover
2. Multi-tenant VPC design
3. Compliance-driven network architecture

---

## Lessons for Future Learning

### What Worked Well
1. ✅ Step-by-step implementation approach
2. ✅ Validation after each step
3. ✅ Understanding "why" before "how"
4. ✅ Documenting learnings immediately

### What to Improve
1. ⏳ More time on security concepts early
2. ⏳ Include cost analysis in initial design
3. ⏳ Document troubleshooting steps during implementation
4. ⏳ Capture screenshots during implementation

---

## Estimated Cost Tracker

| Task | Service | Estimated | Actual | Notes |
|------|---------|-----------|--------|-------|
| AWS-001 | VPC | Free | Free | No running resources |
| AWS-002 | EC2, IGW | $0.50-2 | TBD | 1 t2.micro instance |
| AWS-003 | EC2, NAT | $6-7 | TBD | NAT Gateway + 2 instances |
| AWS-005 | VPC Endpoint, S3 | Free | Free | Endpoint is free; no data costs |
| AWS-006 | VPC Peering, EC2 | ~$27 | TBD | 3 instances, cross-region traffic |
| **Total** | **All** | **~$40-50** | TBD | For complete portfolio |

---

## Interview Preparation Notes

### Frequently Asked Questions
1. **"What is a VPC?"** — Isolated network environment; you control IP addressing, subnets, routing, security
2. **"Difference between public and private subnets?"** — Route table (public has IGW route, private doesn't)
3. **"How do you access a private EC2?"** — Bastion host pattern or Systems Manager Session Manager
4. **"Why can't private instance access S3 with IAM role?"** — Needs network path (VPC Endpoint or NAT)
5. **"VPC Peering vs. Transit Gateway?"** — Peering for <5 VPCs; Transit Gateway for >10 VPCs or hub-and-spoke

### Practice Scenarios
- [ ] Design 3-tier VPC architecture (web, app, database subnets)
- [ ] Plan multi-region deployment
- [ ] Troubleshoot "can't reach resource" scenarios
- [ ] Design high-availability bastion host setup
- [ ] Calculate cost comparison: NAT vs. Endpoint

---

**Last Updated:** October 13, 2026  
**Tasks Completed:** 5/8 (62.5%)  
**Total Learning Time:** ~7.5 hours  
**Confidence Level:** Intermediate (ready for AWS Solutions Architect Associate exam coverage)
