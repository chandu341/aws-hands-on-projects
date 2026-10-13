# Security Groups & Network ACLs

**Task ID:** AWS-004  
**Category:** Security  
**Status:** ✅ Completed  
**AWS Services:** Security Groups, Network ACLs, EC2, VPC  
**Date Completed:** October 11, 2026

---

## Objective

Implement and understand the differences between Security Groups and Network ACLs, demonstrating how they work together to provide multi-layer security at both instance and subnet levels.

---

## Why Security Groups & NACLs Matter

AWS network security operates at multiple layers:

- **Security Groups (Instance-level):** Stateful, instance-specific, allow-based by default
- **Network ACLs (Subnet-level):** Stateless, subnet-wide, explicit allow/deny rules
- **Together:** Provide defense-in-depth, preventing both internal and external attacks

Understanding both is critical for AWS Solutions Architect certification and production deployments.

---

## Architecture Overview

```
VPC (10.0.0.0/16)
├── Subnet (10.0.1.0/24)
│   ├── NACL Rules (Stateless)
│   │   ├── Inbound: SSH (22), HTTP (80), HTTPS (443)
│   │   └── Outbound: All (0-65535)
│   └── EC2 Instance
│       ├── Security Group (Stateful)
│       │   ├── Inbound: SSH (22), HTTP (80)
│       │   └── Outbound: All
│       └── Web Server (nginx)
```

---

## Configuration Details

### Security Group Configuration
- **Name:** Web-SG
- **VPC:** Learning VPC
- **Inbound Rules:**
  - SSH (22): From 0.0.0.0/0 (or restricted to admin IPs)
  - HTTP (80): From 0.0.0.0/0
  - HTTPS (443): From 0.0.0.0/0
  - ICMP (ping): From 0.0.0.0/0 (optional, for testing)
- **Outbound Rules:** All traffic (default)

### Network ACL Configuration
- **Name:** Web-NACL
- **VPC:** Learning VPC
- **Inbound Rules:**
  - Rule 100: SSH (22) from 0.0.0.0/0 → Allow
  - Rule 110: HTTP (80) from 0.0.0.0/0 → Allow
  - Rule 120: HTTPS (443) from 0.0.0.0/0 → Allow
  - Rule 130: Ephemeral ports (1024-65535) from 0.0.0.0/0 → Allow
  - Rule 140: All traffic → Deny (implicit)
- **Outbound Rules:**
  - Rule 100: All traffic to 0.0.0.0/0 → Allow

---

## Implementation Steps

### Step 1: Create Security Group
```bash
# Via AWS Console
# EC2 Dashboard → Security Groups → Create Security Group
# Name: Web-SG
# VPC: Learning VPC
# Description: Security group for web servers

# Inbound Rules:
# SSH (22) from 0.0.0.0/0
# HTTP (80) from 0.0.0.0/0
# HTTPS (443) from 0.0.0.0/0
# ICMP from 0.0.0.0/0

# Outbound Rules: All (default)
```

### Step 2: Create Network ACL
```bash
# VPC Dashboard → Network ACLs → Create Network ACL
# Name: Web-NACL
# VPC: Learning VPC

# Configure Inbound Rules:
# Rule 100: SSH (22) from 0.0.0.0/0 → Allow
# Rule 110: HTTP (80) from 0.0.0.0/0 → Allow
# Rule 120: HTTPS (443) from 0.0.0.0/0 → Allow
# Rule 130: TCP (1024-65535) from 0.0.0.0/0 → Allow (ephemeral)
# Rule 32767: All → Deny (default)

# Configure Outbound Rules:
# Rule 100: All → Allow
# Rule 32767: All → Deny (default)
```

### Step 3: Associate NACL with Subnet
```bash
# VPC Dashboard → Network ACLs
# Select Web-NACL → Subnet Associations → Edit
# Select Web Subnet
# Save
```

### Step 4: Launch EC2 Instance
```bash
# EC2 Dashboard → Launch Instance
# Name: Web-Server
# VPC: Learning VPC
# Subnet: Web Subnet
# Security Group: Web-SG
# AMI: Amazon Linux 2 or Ubuntu
```

### Step 5: Test Connectivity

```bash
# Test SSH
ssh -i key.pem ec2-user@<public-ip>

# Test HTTP
curl http://<public-ip>

# Test ICMP (ping)
ping <public-ip>

# Test HTTPS (if configured)
curl https://<public-ip>
```

---

## Key Differences: Security Groups vs. NACLs

| Feature | Security Groups | Network ACLs |
|---------|---|---|
| **Layer** | Instance-level | Subnet-level |
| **State** | Stateful | Stateless |
| **Rules** | Allow-only (implicit deny) | Allow and Deny |
| **Rule Order** | No order (all evaluated) | Ordered (first match wins) |
| **Default** | Deny all inbound | Allow all (if not restricted) |
| **Scope** | Individual instance/ENI | Entire subnet |
| **Performance** | No performance impact | Minimal impact |
| **Use Case** | Primary defense | Additional layer |

### Stateful vs. Stateless

**Security Groups (Stateful):**
- If you allow inbound SSH (22), return traffic is automatically allowed
- You don't need to explicitly allow ephemeral port responses
- Simpler to configure

**Network ACLs (Stateless):**
- Inbound and outbound are completely separate
- If you allow inbound HTTP (80), you must also allow outbound responses
- This is why ephemeral port ranges (1024-65535) are typically allowed

---

## What I Learned

### Security Group Fundamentals
1. **Stateful Filtering:** Allow rules work bidirectionally
2. **Instance-Level Control:** Each instance can have different security
3. **Allow-Based Model:** Only explicitly allowed traffic passes
4. **Dynamic Evaluation:** All rules evaluated (no first-match)

### Network ACL Fundamentals
1. **Stateless Filtering:** Inbound and outbound are independent
2. **Subnet-Level Control:** All instances in subnet must follow same rules
3. **Allow & Deny:** Can explicitly deny traffic
4. **First-Match Wins:** Rules evaluated in order; first match applies

### Defense in Depth
- Security Groups alone can be bypassed (if someone gains EC2 access)
- NACLs provide additional layer at subnet level
- Combined: Excellent defense against compromised instances

### Ephemeral Port Ranges
- OS uses ports 1024-65535 for outbound connections
- NACL rules must allow these for responses to pass
- Security Groups handle this automatically (stateful)

### Common Mistakes
- Forgetting ephemeral port ranges in NACL rules (breaks connectivity)
- Using overly permissive rules (0.0.0.0/0 for everything)
- Not understanding stateful vs. stateless (security groups vs. NACLs)

---

## Real-World Production Use Cases

### DMZ (Demilitarized Zone) Design
```
Public Subnet NACL:
- Inbound: SSH (22), HTTP (80), HTTPS (443)
- Outbound: All

Security Group on web servers:
- Inbound: HTTP (80), HTTPS (443) only
- Outbound: To internal subnet only

Result: Web servers can't SSH to each other; only admins can connect
```

### Microsegmentation
```
Database Subnet NACL:
- Inbound: MySQL (3306) from app subnet only
- Outbound: All

App Subnet Security Group:
- Inbound: HTTP (80) from load balancer only
- Outbound: MySQL (3306) to database only

Result: Tight control; compromised instance can't access unrelated services
```

### DDoS Mitigation
```
NACL Rules (at subnet level):
- Limit rate of connections
- Block known malicious IPs
- Allow only legitimate sources

Security Group (at instance level):
- Additional filtering
- Whitelist specific IPs/ports

Result: Multi-layer protection against attacks
```

---

## Security Best Practices

### Security Groups
- ✅ Use least privilege: Only open necessary ports
- ✅ Restrict SSH to admin IPs (not 0.0.0.0/0)
- ✅ Use security group references for inter-tier communication
- ⚠️ Regularly audit and remove unused rules
- ❌ Never open all ports (0-65535) unless absolutely necessary

### Network ACLs
- ✅ Configure ephemeral port ranges (1024-65535 outbound)
- ✅ Use explicit deny rules for known malicious sources
- ✅ Document rule order and dependencies
- ⚠️ Test rule changes (may break connectivity)
- ❌ Don't over-complicate NACL rules (security groups usually sufficient)

### Defense in Depth
- ✅ Use both security groups AND NACLs
- ✅ Layer multiple security mechanisms
- ✅ Implement least privilege at every level
- ✅ Monitor and audit rule changes
- ✅ Test security rules during implementation

---

## Troubleshooting Common Issues

### Issue 1: Can't Connect to Instance
1. Check Security Group inbound rules (port and protocol)
2. Check NACL inbound rules (port and protocol)
3. Check NACL outbound rules (ephemeral ports 1024-65535)
4. Verify Instance Security Group is selected
5. Verify NACL is associated with correct subnet

### Issue 2: Instance Can't Reach External Services
1. Check Security Group outbound rules
2. Check NACL outbound rules
3. Check route table (IGW or NAT route exists)
4. Verify security group allows destination port

### Issue 3: Intermittent Connectivity
1. Often caused by ephemeral port restrictions in NACL
2. Verify outbound rules include 1024-65535 range
3. Test with specific ports vs. port ranges

---

## Cost Considerations

| Component | Cost |
|-----------|------|
| Security Groups | Free |
| Network ACLs | Free |
| EC2 Instance | ~$0.0116/hour (t2.micro) |

**Total Cost:** Free (security configurations themselves)

---

## Interview Questions

### Foundational
1. **What is a Security Group?**
   - Instance-level, stateful firewall; allows inbound/outbound traffic
2. **What is a Network ACL?**
   - Subnet-level, stateless firewall; explicit allow/deny rules
3. **What's the difference between stateful and stateless?**
   - Stateful: tracks connections; return traffic allowed automatically
   - Stateless: treats inbound/outbound independently

### Intermediate
4. **Why would you need both Security Groups and NACLs?**
   - Defense in depth; NACLs protect entire subnet; SGs protect instance
5. **What happens if you deny traffic in NACL but allow in SG?**
   - Traffic is denied (NACL is checked first)
6. **How do ephemeral ports work in NACLs?**
   - Outbound responses use high port numbers; NACL must allow 1024-65535

### Advanced
7. **How would you design security groups and NACLs for a 3-tier application?**
   - Public tier: Allow HTTP/HTTPS, SSH from admin
   - App tier: Allow app ports from load balancer only
   - Database tier: Allow DB port from app tier only

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 11, 2026)
- **Documentation:** ✅ Complete
- **Screenshots:** 📸 4 images extracted and organized
- **Validation:** ✅ Configuration tested

---

## References

- [Security Groups Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [Network ACLs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [Security Group Rules Reference](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SecurityGroupRules)
