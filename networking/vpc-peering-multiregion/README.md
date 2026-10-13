# VPC Peering Multi-Region Setup

**Task ID:** AWS-006  
**Category:** Networking  
**Status:** ✅ Completed  
**AWS Services:** VPC, VPC Peering, Multi-Region, Route Tables, EC2  
**Date Completed:** October 8, 2026

---

## Objective

Configure VPC peering between three VPCs across two AWS regions (North Virginia and Hyderabad) to enable direct communication between database, application, and web servers running in isolated VPCs.

---

## Why VPC Peering?

In complex architectures, you often need to connect multiple VPCs:

- **Isolation:** Each service (database, application, web) runs in its own VPC for security and blast radius containment
- **Service Boundaries:** Teams can manage their own VPCs independently
- **Multi-Region:** Distribute services across regions for disaster recovery and compliance
- **Direct Communication:** VPC peering provides a direct, private connection between VPCs without routing through the internet

VPC peering is the building block for large-scale AWS deployures and is heavily tested in AWS Solutions Architect interviews.

---

## Architecture Overview

```
Region: US East (N. Virginia)
└── VPC 1: DB Server VPC (12.1.0.0/16)
    └── EC2: Database Server
        ├── Private IP: 12.1.1.x
        └── Peering: ↔ App VPC, ↔ Web VPC

Region: Asia Pacific (Hyderabad)
├── VPC 2: App Server VPC (12.2.0.0/16)
│   └── EC2: Application Server
│       ├── Private IP: 12.2.1.x
│       └── Peering: ↔ DB VPC, ↔ Web VPC
└── VPC 3: Web Server VPC (12.3.0.0/16)
    └── EC2: Web Server
        ├── Private IP: 12.3.1.x
        └── Peering: ↔ DB VPC, ↔ App VPC

Peering Connections:
- DB VPC ↔ App VPC (cross-region: N.Virginia ↔ Hyderabad)
- App VPC ↔ Web VPC (same region: Hyderabad)
- DB VPC ↔ Web VPC (cross-region: N.Virginia ↔ Hyderabad)
```

---

## Configuration Details

### VPC Setup

#### VPC 1: DB Server VPC (N. Virginia)
- **Region:** US East (N. Virginia)
- **CIDR Block:** 12.1.0.0/16
- **Purpose:** Database servers
- **Subnets:** 1 private subnet (12.1.1.0/24)
- **EC2 Instance:** Database Server (t2.micro, Amazon Linux)

#### VPC 2: App Server VPC (Hyderabad)
- **Region:** Asia Pacific (Hyderabad)
- **CIDR Block:** 12.2.0.0/16
- **Purpose:** Application servers
- **Subnets:** 1 private subnet (12.2.1.0/24)
- **EC2 Instance:** Application Server (t2.micro, Amazon Linux)

#### VPC 3: Web Server VPC (Hyderabad)
- **Region:** Asia Pacific (Hyderabad)
- **CIDR Block:** 12.3.0.0/16
- **Purpose:** Web servers
- **Subnets:** 1 private subnet (12.3.1.0/24)
- **EC2 Instance:** Web Server (t2.micro, Amazon Linux)

### VPC Peering Connections
- **Peering 1:** DB VPC (12.1.0.0/16) ↔ App VPC (12.2.0.0/16) - Cross-region
- **Peering 2:** App VPC (12.2.0.0/16) ↔ Web VPC (12.3.0.0/16) - Same region
- **Peering 3:** DB VPC (12.1.0.0/16) ↔ Web VPC (12.3.0.0/16) - Cross-region

---

## Implementation Steps

### Phase 1: Create VPCs and Subnets

#### Step 1A: Create DB VPC (N. Virginia)
```bash
# Region: US East (N. Virginia)
# VPC Dashboard → Create VPC
# Name: DB-VPC
# CIDR: 12.1.0.0/16
# Create

# Create Subnet
# Subnet Name: DB-Subnet
# VPC: DB-VPC
# CIDR: 12.1.1.0/24
# Availability Zone: us-east-1a
```

#### Step 1B: Create App VPC (Hyderabad)
```bash
# Region: Asia Pacific (Hyderabad)
# VPC Dashboard → Create VPC
# Name: App-VPC
# CIDR: 12.2.0.0/16
# Create

# Create Subnet
# Subnet Name: App-Subnet
# VPC: App-VPC
# CIDR: 12.2.1.0/24
# Availability Zone: ap-south-1a
```

#### Step 1C: Create Web VPC (Hyderabad)
```bash
# Region: Asia Pacific (Hyderabad)
# VPC Dashboard → Create VPC
# Name: Web-VPC
# CIDR: 12.3.0.0/16
# Create

# Create Subnet
# Subnet Name: Web-Subnet
# VPC: Web-VPC
# CIDR: 12.3.1.0/24
# Availability Zone: ap-south-1a
```

### Phase 2: Launch EC2 Instances

#### Step 2A: Launch DB Server (N. Virginia)
```bash
# Region: US East (N. Virginia)
# EC2 Dashboard → Launch Instance
# Name: DB-Server
# AMI: Amazon Linux 2
# Instance Type: t2.micro
# VPC: DB-VPC
# Subnet: DB-Subnet
# Security Group: Allow SSH (22) from 0.0.0.0/0 (or restrict as needed)
# Launch

# Note the Private IP (e.g., 12.1.1.x)
```

#### Step 2B: Launch App Server (Hyderabad)
```bash
# Region: Asia Pacific (Hyderabad)
# EC2 Dashboard → Launch Instance
# Name: App-Server
# AMI: Amazon Linux 2
# Instance Type: t2.micro
# VPC: App-VPC
# Subnet: App-Subnet
# Security Group: Allow SSH (22), ICMP (ping) from 0.0.0.0/0
# Launch

# Note the Private IP (e.g., 12.2.1.x)
```

#### Step 2C: Launch Web Server (Hyderabad)
```bash
# Region: Asia Pacific (Hyderabad)
# EC2 Dashboard → Launch Instance
# Name: Web-Server
# AMI: Amazon Linux 2
# Instance Type: t2.micro
# VPC: Web-VPC
# Subnet: Web-Subnet
# Security Group: Allow SSH (22), ICMP (ping) from 0.0.0.0/0
# Launch

# Note the Private IP (e.g., 12.3.1.x)
```

### Phase 3: Create VPC Peering Connections

#### Step 3A: Create Peering 1 (DB VPC ↔ App VPC)
```bash
# Region: US East (N. Virginia) - Start peering request from here
# VPC Dashboard → Peering Connections
# Create Peering Connection
# Name: DB-App-Peering
# Local VPC: DB-VPC (12.1.0.0/16)
# Peer VPC: App-VPC (12.2.0.0/16)
# Peer Region: Asia Pacific (Hyderabad)
# Create Request

# Switch to Hyderabad region
# VPC Dashboard → Peering Connections
# Find pending request: DB-App-Peering
# Actions → Accept Request
# Accept
```

#### Step 3B: Create Peering 2 (App VPC ↔ Web VPC)
```bash
# Region: Asia Pacific (Hyderabad)
# VPC Dashboard → Peering Connections
# Create Peering Connection
# Name: App-Web-Peering
# Local VPC: App-VPC (12.2.0.0/16)
# Peer VPC: Web-VPC (12.3.0.0/16)
# Peer Region: Asia Pacific (Hyderabad)
# Create

# Accept Request (same region, appears immediately)
```

#### Step 3C: Create Peering 3 (DB VPC ↔ Web VPC)
```bash
# Region: US East (N. Virginia)
# VPC Dashboard → Peering Connections
# Create Peering Connection
# Name: DB-Web-Peering
# Local VPC: DB-VPC (12.1.0.0/16)
# Peer VPC: Web-VPC (12.3.0.0/16)
# Peer Region: Asia Pacific (Hyderabad)
# Create Request

# Switch to Hyderabad region
# VPC Dashboard → Peering Connections
# Find pending request: DB-Web-Peering
# Actions → Accept Request
# Accept
```

### Phase 4: Update Route Tables

#### Step 4A: Update DB Route Table
```bash
# Region: US East (N. Virginia)
# VPC Dashboard → Route Tables
# Select DB-Subnet Route Table
# Routes tab → Edit Routes
# Add Route:
#   Destination: 12.2.0.0/16 (App VPC)
#   Target: DB-App-Peering
# Add Route:
#   Destination: 12.3.0.0/16 (Web VPC)
#   Target: DB-Web-Peering
# Save Routes
```

#### Step 4B: Update App Route Table
```bash
# Region: Asia Pacific (Hyderabad)
# VPC Dashboard → Route Tables
# Select App-Subnet Route Table
# Routes tab → Edit Routes
# Add Route:
#   Destination: 12.1.0.0/16 (DB VPC)
#   Target: DB-App-Peering
# Add Route:
#   Destination: 12.3.0.0/16 (Web VPC)
#   Target: App-Web-Peering
# Save Routes
```

#### Step 4C: Update Web Route Table
```bash
# Region: Asia Pacific (Hyderabad)
# VPC Dashboard → Route Tables
# Select Web-Subnet Route Table
# Routes tab → Edit Routes
# Add Route:
#   Destination: 12.1.0.0/16 (DB VPC)
#   Target: DB-Web-Peering
# Add Route:
#   Destination: 12.2.0.0/16 (App VPC)
#   Target: App-Web-Peering
# Save Routes
```

### Phase 5: Test Connectivity

#### Test 1: DB Server → App Server
```bash
# SSH into DB Server (you need a public IP or bastion for this)
# Once connected:
ping 12.2.1.x  # App Server private IP
# Should succeed

# Verify connectivity from App side
ssh -i key.pem ec2-user@<app-private-ip>
# This works through peering
```

#### Test 2: App Server → Web Server
```bash
# SSH into App Server
ping 12.3.1.x  # Web Server private IP
# Should succeed

# SSH to Web Server
ssh -i key.pem ec2-user@<web-private-ip>
```

#### Test 3: Web Server → DB Server
```bash
# SSH into Web Server
ping 12.1.1.x  # DB Server private IP
# Should succeed
```

#### Test 4: Full Mesh Connectivity
```bash
# From DB Server:
ping 12.2.1.x && ping 12.3.1.x

# From App Server:
ping 12.1.1.x && ping 12.3.1.x

# From Web Server:
ping 12.1.1.x && ping 12.2.1.x

# All pings should succeed, confirming full mesh topology
```

---

## Commands & Validation

### AWS CLI Commands for Peering

```bash
# List all peering connections
aws ec2 describe-vpc-peering-connections --region us-east-1

# Create peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-xxxxx \
  --peer-vpc-id vpc-yyyyy \
  --peer-region ap-south-1 \
  --region us-east-1

# Accept peering connection
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxxxx \
  --region ap-south-1

# Describe route tables
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=vpc-xxxxx" \
  --region us-east-1

# Create route to peering connection
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 12.2.0.0/16 \
  --vpc-peering-connection-id pcx-xxxxx \
  --region us-east-1
```

### Connectivity Testing

```bash
# From any EC2 instance, test ICMP connectivity
ping <destination-private-ip>
# Example: ping 12.2.1.10

# Test SSH connectivity
ssh -i key.pem ec2-user@<destination-private-ip>

# Check route table entries
route -n
# or: ip route show

# Trace route to destination
traceroute 12.2.1.x
# Should show direct peering path

# Monitor traffic
tcpdump -i eth0 icmp
# Watch for ping packets through peering
```

---

## What I Learned

### VPC Peering Fundamentals
- VPC peering is a **one-to-one connection** between two VPCs; it doesn't create a transitive relationship
- For multi-VPC connectivity (3+ VPCs), you must create **multiple peering connections** (mesh topology)
- Each peering connection must be **explicitly created and accepted** on both sides

### Cross-Region Considerations
- Cross-region peering connections have **higher latency** and **data transfer costs** ($0.01/GB)
- Same-region peering is free (or much cheaper)
- Cross-region peering is useful for **disaster recovery** and **geographic distribution**

### Route Table Configuration is Critical
- Creating a peering connection alone **doesn't enable traffic flow**
- You must **update route tables** on both sides with routes pointing to the peering connection
- Each VPC needs a route for the other VPC's CIDR block

### CIDR Block Planning
- **Non-overlapping CIDR blocks are essential** for peering to work
- If two VPCs have overlapping CIDR blocks, peering will fail or traffic won't route correctly
- This is a common cause of peering failures in production

### Full Mesh Topology
- For 3 VPCs, you need 3 peering connections (not 1)
  - VPC1 ↔ VPC2
  - VPC2 ↔ VPC3
  - VPC1 ↔ VPC3
- Without all three connections, not all VPCs can communicate directly

### Limitations of VPC Peering
- Peering does **not** support transitive routing (A→B→C is not possible; you need A→C directly)
- For true hub-and-spoke architectures, use **AWS Transit Gateway** instead
- Peering is peer-to-peer; Transit Gateway is centralized

---

## Real-World Production Use Cases

### Multi-Region High Availability
```
Region 1: Primary Database Cluster
  ↓ (VPC Peering)
Region 2: Failover Database Cluster
  ↓ (VPC Peering)
Region 3: Read Replicas

Benefit: Low-latency cross-region failover, shared data
```

### Organizational Separation with Connectivity
```
Company Structure:
- VPC-Finance: Accounting systems
- VPC-Engineering: Development systems
- VPC-Operations: Shared services (logging, monitoring)

Peering:
- Finance ↔ Operations
- Engineering ↔ Operations
- Finance ↔ Engineering (limited)

Benefit: Network isolation with controlled inter-team access
```

### Microservices Across Isolated VPCs
```
VPC-API: REST API servers
  ↔ (peering)
VPC-Database: PostgreSQL clusters
  ↔ (peering)
VPC-Cache: Redis clusters

Benefit: Blast radius containment, independent scaling
```

---

## Security Considerations

### Network Segmentation
- ✅ VPC peering enables network isolation while allowing controlled communication
- ✅ Use Security Groups to restrict which traffic flows through peering
- ✅ NACLs can add subnet-level filtering
- ⚠️ Peering is private (no internet exposure), but security groups are still required

### CIDR Block Planning
- ✅ Non-overlapping CIDR blocks are a prerequisite
- ✅ Document your VPC topology to prevent future conflicts
- ⚠️ Adding peering to existing VPCs requires careful CIDR analysis

### Cross-Region Security
- ✅ Cross-region peering creates a private connection (no internet)
- ✅ AWS encrypts cross-region traffic by default
- ⚠️ Data transfer costs apply to cross-region peering
- ⚠️ Use VPC Flow Logs to audit cross-region traffic

### Access Control
- ✅ Use security groups to restrict source and destination ports
- ✅ Implement principle of least privilege (e.g., only allow required ports)
- ✅ NACLs provide subnet-level stateless filtering
- ⚠️ Don't allow all traffic (0.0.0.0/0) by default

---

## Cost Considerations

| Component | Cost | Notes |
|-----------|------|-------|
| VPC Peering (same region) | Free | No hourly or data transfer charges |
| VPC Peering (cross-region) | $0.01/GB | Only for cross-region traffic |
| EC2 t2.micro (per region) | ~$0.0116/hour | 3 instances across 2 regions |
| Data Transfer (cross-region) | $0.02/GB | Inter-region EC2 to EC2 |

**Example Monthly Costs:**
- 3 t2.micro instances: ~$25/month
- 100GB cross-region data transfer: $1-2/month
- **Total: ~$27/month** for this 3-VPC multi-region setup

---

## Resource Cleanup

```bash
# 1. Terminate EC2 instances in all VPCs/regions
# EC2 Dashboard (each region) → Instances → Terminate

# 2. Delete Peering Connections
# VPC Dashboard → Peering Connections → Select peering → Delete

# 3. Delete VPCs
# VPC Dashboard → Your VPCs → Select VPC → Delete

# AWS CLI alternative:
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx --region us-east-1
aws ec2 delete-vpc --vpc-id vpc-xxxxx --region us-east-1
```

---

## Interview Questions

### Foundational
1. **What is VPC peering and why would you use it?**
   - Answer: VPC peering is a direct, private connection between two VPCs that enables them to communicate as if they were in the same network. Use it for multi-VPC architectures, disaster recovery, or organizational separation.

2. **What are the prerequisites for VPC peering to work?**
   - Answer: Non-overlapping CIDR blocks, explicit peering connection creation, acceptance on both sides, and route table entries pointing traffic through the peering connection.

3. **How is VPC peering different from a VPN?**
   - Answer: VPC peering is private AWS-to-AWS connectivity with lower latency and no encryption overhead. VPN is encrypted but slower; typically used for on-premises to AWS connectivity.

### Intermediate
4. **Can you ping a resource in a peered VPC?**
   - Answer: Yes, if the security groups, NACLs, and route tables are all correctly configured. The connectivity test (ping) relies on all three layers.

5. **What happens if you have overlapping CIDR blocks on two VPCs you're trying to peer?**
   - Answer: The peering connection can be created but traffic won't route correctly. AWS allows the peering but doesn't automatically resolve CIDR conflicts. Route tables will be ambiguous.

6. **What is transitive routing and why doesn't VPC peering support it?**
   - Answer: Transitive routing means A can reach C through B (A→B→C). VPC peering doesn't support this; each peering connection is independent. For transitive routing, use AWS Transit Gateway.

### Advanced
7. **How would you design a multi-VPC architecture for 10 different microservices across 5 regions?**
   - Answer: Use AWS Transit Gateway instead of individual peering connections. Transit Gateway creates a hub-and-spoke topology, reducing the number of connections from 45 (full mesh) to 5 (per region). This scales better and simplifies management.

8. **What are the trade-offs between VPC peering and Transit Gateway?**
   - VPC Peering: Simple, peer-to-peer, no additional cost (same region), but doesn't scale (45 connections for 10 VPCs)
   - Transit Gateway: Centralized hub, scales to hundreds of VPCs, hourly cost, but more complex

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 8, 2026)
- **Documentation:** ✅ Complete
- **Validation:** ✅ Verified (all 3 servers reachable via peering, bidirectional connectivity confirmed)
- **Screenshots:** 📸 Available in `./screenshots/` directory

---

## Related Tasks

- **Previous:** [VPC Endpoints & S3](../../application-services/vpc-endpoint-s3/) - Private AWS service access
- **Related:** [Public/Private Subnets & NAT](../public-private-subnets-nat/) - Bastion pattern
- **Advanced:** Transit Gateway (for multi-VPC at scale)

---

## References

- [VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [VPC Peering Requester](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html)
- [Cross-Region Peering](https://docs.aws.amazon.com/vpc/latest/peering/cross-region-peering.html)
- [Transit Gateway (Alternative)](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
