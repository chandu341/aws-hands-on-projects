# VPC Creation with Multi-AZ Setup

**Task ID:** AWS-001  
**Category:** Networking  
**Status:** ✅ Completed  
**AWS Services:** VPC, Availability Zones, Subnets  
**Date Completed:** October 6, 2026

---

## Objective

Create a foundational VPC (Virtual Private Cloud) with multiple availability zones and both public and private subnets to support high-availability applications in AWS.

---

## Why VPC and Multi-AZ Architecture?

A VPC is the foundational networking layer in AWS that allows you to:
- Define your own isolated network environment with custom IP addressing
- Control inbound and outbound traffic through security groups and network ACLs
- Deploy resources across multiple availability zones for redundancy and high availability
- Build secure, scalable, and resilient cloud infrastructure

Multi-AZ (Availability Zone) deployment ensures:
- **High Availability:** If one AZ goes down, resources in other AZs continue running
- **Fault Tolerance:** Application uptime is protected against infrastructure failures
- **Production-Ready Architecture:** Industry standard for enterprise applications

---

## Architecture Overview

```
VPC: Sandpit VPC
├── CIDR Block: Custom private IP range (IPv4 only, IPv6 disabled)
├── Availability Zone 1
│   ├── Public Subnet 1
│   └── Private Subnet 1
├── Availability Zone 2
│   ├── Public Subnet 2
│   └── Private Subnet 2
└── Availability Zone 3
    ├── Public Subnet 3
    └── Private Subnet 3
```

---

## Configuration Details

### VPC Settings
- **VPC Name:** Sandpit VPC
- **CIDR Block:** Custom private IP address range (user-specified)
- **IPv6 CIDR Block:** Disabled (not required for this exercise)
- **DNS Settings:** Default (AWS-provided DNS enabled)

### Subnet Configuration
- **Total Availability Zones:** 3
- **Public Subnets:** 3 (one per AZ)
- **Private Subnets:** 3 (one per AZ)
- **NAT Gateway:** Not configured at VPC creation (created in later exercises)
- **VPC Endpoints:** Not configured at VPC creation

### Public IPv4 Auto-Assignment
After VPC creation, auto-assign public IPv4 address was enabled on all public subnets. This ensures:
- EC2 instances launched in public subnets automatically receive public IP addresses
- Instances can be accessed from the internet without manual EIP allocation
- Simplifies the workflow for web servers and bastion hosts

---

## Implementation Steps

### Step 1: Create the VPC
1. Navigate to AWS VPC Dashboard
2. Click **Create VPC**
3. Choose **VPC with custom settings** (to specify custom CIDR block)
4. Enter VPC name: "Sandpit VPC"
5. Specify the custom CIDR block (private IP range, e.g., `10.0.0.0/16`)
6. Disable IPv6 CIDR block
7. Click **Create**

### Step 2: Create Public Subnets (3x)
For each of the 3 availability zones:
1. Go to Subnets → **Create Subnet**
2. Select the Sandpit VPC
3. Subnet name: `Public-Subnet-AZ{1,2,3}`
4. Choose appropriate availability zone
5. Assign subnet CIDR block (e.g., `10.0.1.0/24` for AZ1, `10.0.2.0/24` for AZ2, etc.)
6. Click **Create Subnet**

### Step 3: Create Private Subnets (3x)
For each of the 3 availability zones:
1. Go to Subnets → **Create Subnet**
2. Select the Sandpit VPC
3. Subnet name: `Private-Subnet-AZ{1,2,3}`
4. Choose appropriate availability zone
5. Assign subnet CIDR block (e.g., `10.0.11.0/24` for AZ1, `10.0.12.0/24` for AZ2, etc.)
6. Click **Create Subnet**

### Step 4: Enable Auto-Assign Public IPv4 Address
For each public subnet:
1. Select the public subnet
2. Click **Actions** → **Edit subnet settings**
3. Check **Enable auto-assign public IPv4 address**
4. Click **Save**
5. Verify the setting is applied to all 3 public subnets

---

## Configuration Values

| Component | Value | Notes |
|-----------|-------|-------|
| VPC Name | Sandpit VPC | Primary VPC for learning |
| VPC CIDR Block | Custom range | User-defined, e.g., 10.0.0.0/16 |
| Availability Zones | 3 | High availability design |
| Public Subnets | 3 | One per AZ, auto-assign public IPs enabled |
| Private Subnets | 3 | One per AZ, no public IP assignment |
| IPv6 Support | Disabled | Not needed for this exercise |

---

## Validation Steps

### Verify VPC Creation
1. Navigate to **VPCs** dashboard
2. Confirm "Sandpit VPC" appears in the list
3. Verify CIDR block matches your configuration

### Verify Subnets
1. Navigate to **Subnets**
2. Filter by VPC (Sandpit VPC)
3. Confirm all 6 subnets are created (3 public + 3 private)
4. Verify each subnet is in the correct availability zone

### Verify Auto-Assign Public IPv4
1. Select each public subnet
2. Check **Details** tab
3. Confirm **Auto-assign public IPv4 address:** is enabled

---

## What I Learned

### VPC Fundamentals
- A VPC is your isolated network environment in AWS where you have full control over IP addressing, subnets, route tables, and traffic flow
- You must explicitly create subnets; resources cannot be launched in a VPC without a subnet
- CIDR blocks define IP address ranges; planning CIDR blocks prevents IP conflicts and enables future growth

### Availability Zone Strategy
- AWS has multiple availability zones per region; deploying across all 3 AZs in a region provides maximum availability
- Each subnet must be in exactly one AZ; this forces proper distribution of resources
- Multi-AZ architectures protect against zone-level failures (power outages, hardware issues, etc.)

### Public vs. Private Subnets
- This distinction is purely based on route table configuration (covered in later exercises)
- A subnet is "public" if its route table has a route to the Internet Gateway
- A subnet is "private" if it does not have a direct route to the Internet Gateway
- Auto-assigning public IPs only makes sense for public subnets

### Planning for Growth
- CIDR block planning is critical; choosing `/16` (65,536 IPs) allows room for expansion
- Subnet sizing (e.g., `/24` for 256 IPs per subnet) provides flexibility for adding resources

---

## Real-World Production Use Cases

### Web Application Architecture
```
Multi-AZ VPC with:
- Public subnets: Load balancer, bastion hosts, NAT gateways
- Private subnets: Application servers (auto-scaling groups)
- Separate subnets for databases (RDS in private subnet)
Result: Highly available, secure application stack
```

### Microservices Environment
```
Each service runs in multiple AZs:
- API Gateway in public subnets
- Microservices in private subnets
- Databases with Multi-AZ RDS
Result: Service isolation and fault tolerance
```

### Disaster Recovery
```
Secondary VPC in another region with:
- Same architecture as primary VPC
- Periodic snapshots synced to primary region
Result: Business continuity in case of region failure
```

---

## Security Considerations

### Network Segmentation
- ✅ Private subnets provide initial protection for internal services
- ⚠️ Security Groups and NACLs (covered in later exercises) provide finer-grained control
- Recommendation: Use private subnets for databases, microservices, and internal APIs

### IP Address Management
- ✅ Private CIDR blocks (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) are reserved for private networks
- ✅ Prevents accidental overlap with public internet IPs
- ⚠️ VPC peering and hybrid connectivity require careful CIDR planning to avoid conflicts

### Public Subnet Best Practices
- ✅ Auto-assign public IPs only for resources that need internet access
- ✅ Use NAT Gateway for private instances that need outbound internet access
- ⚠️ Never use public subnets for databases or sensitive services

---

## Cost Considerations

| Component | Cost | Notes |
|-----------|------|-------|
| VPC Creation | Free | No charge for creating a VPC |
| Subnets | Free | No charge for subnets |
| Route Tables | Free | Basic route tables included |
| Elastic IPs (future) | $0.005/hour | Only charged if not associated with a running instance |
| NAT Gateway (future) | $0.045/hour + data transfer | Required for private subnet internet access |

**Estimated cost for this exercise:** $0 (no running resources)

---

## Resource Cleanup

To delete this VPC and all associated resources:

1. **Terminate any EC2 instances** in the VPC
2. **Delete NAT Gateways** if created (release associated Elastic IPs)
3. **Delete VPC Endpoints** if created
4. **Navigate to VPCs dashboard**
5. **Select the Sandpit VPC**
6. **Actions** → **Delete VPC**
7. Confirm deletion

**Note:** AWS will not allow deletion of a VPC with associated resources; ensure all dependencies are removed first.

---

## Interview Questions

### Foundational
1. **What is a VPC and why do you need it?**
   - Answer: A VPC is an isolated network environment in AWS where you control networking, security, and resource deployment. It provides security isolation, IP control, and enables hybrid cloud architectures.

2. **What is the difference between public and private subnets?**
   - Answer: The difference is in route tables. Public subnets have a route to an Internet Gateway; private subnets do not. This is a logical distinction, not physical.

3. **Why deploy across multiple availability zones?**
   - Answer: Multi-AZ deployment provides high availability and fault tolerance. If one AZ fails, resources in other AZs continue operating.

### Intermediate
4. **Can you launch an EC2 instance in a VPC without a subnet?**
   - Answer: No. Every EC2 instance must launch in a subnet, and every subnet must be in exactly one AZ.

5. **What is a CIDR block and how does it relate to subnets?**
   - Answer: CIDR notation specifies IP address ranges. A VPC CIDR block (e.g., 10.0.0.0/16) defines the overall IP pool, and subnet CIDR blocks (e.g., 10.0.1.0/24) carve out smaller ranges within the VPC.

6. **What happens if you don't enable auto-assign public IPs for a public subnet?**
   - Answer: Instances still launch in the public subnet, but they won't automatically receive public IPs. You'd need to manually allocate Elastic IPs to access them from the internet.

### Advanced
7. **How would you design a VPC for a high-availability application with private databases?**
   - Answer: Create public subnets in 3 AZs for load balancers/NAT, private subnets in 3 AZs for application servers, and separate database subnets with RDS Multi-AZ for databases.

8. **What are the considerations for CIDR block planning when building multiple VPCs for VPC peering?**
   - Answer: CIDR blocks must not overlap. Plan your IP space carefully; use separate ranges for each VPC (e.g., VPC1: 10.0.0.0/16, VPC2: 10.1.0.0/16) to enable seamless peering.

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 6, 2026)
- **Documentation:** ✅ Complete
- **Screenshots:** 📸 Available in `./screenshots/` directory
- **Validation:** ✅ Verified (all subnets created, auto-assign public IPs enabled)

---

## Related Tasks

- **Next:** [VPC Subnets, Route Tables & IGW](../vpc-subnets-routes-igw/) - Configure routing and internet access
- **Foundation:** This task is the prerequisite for all subsequent networking exercises
- **Advanced:** [VPC Peering Multi-Region](../vpc-peering-multiregion/) - Multi-region connectivity

---

## References

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC CIDR Blocks](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-subnet-basics)
- [Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [Best Practices for VPCs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Best_Practices.html)
