# AWS Network Firewall

**Task ID:** AWS-008  
**Category:** Security  
**Status:** ✅ Completed  
**AWS Services:** AWS Network Firewall, VPC, Stateful Firewall Rules, IDS/IPS  
**Date Completed:** October 11, 2026

---

## Objective

Deploy and configure AWS Network Firewall to provide advanced network security including intrusion detection/prevention, protocol filtering, and DDoS protection at scale.

---

## Why AWS Network Firewall?

While Security Groups and NACLs provide basic network security, AWS Network Firewall adds:

- **Stateful Firewall Rules:** Complex rule evaluation beyond ports/protocols
- **Intrusion Detection/Prevention:** IDS/IPS capabilities built-in
- **Protocol Filtering:** DNS, HTTP, TLS inspection
- **DDoS Protection:** Protect against common attacks
- **Centralized Management:** Single point for VPC-wide security policies
- **Compliance:** Meet regulatory requirements for advanced firewalls

---

## Architecture Overview

```
Internet Gateway
    ↓
Firewall Subnet (with Network Firewall)
    ├─ Firewall Endpoint
    ├─ IDS/IPS Rules
    └─ Stateful Rule Groups
    ↓
Public Subnets (Web tier)
    ├─ ALB/NLB
    └─ Web Servers
    ↓
Private Subnets (App/DB tier)
    ├─ Application Servers
    └─ Databases
```

---

## Configuration Details

### Firewall Policy Components
1. **Stateless Rules:** Fast, simple match (IP, port, protocol)
2. **Stateful Rules:** Track connections, more complex
3. **Rule Groups:** Reusable sets of rules
4. **Suricata Rules:** Industry-standard IPS rules

### Rule Actions
- **Pass:** Allow the traffic
- **Drop:** Block and log
- **Alert:** Log and continue

### Domain Name Filtering
```
Allow: *.amazon.com, *.aws.com
Block: *.malicious-domain.com
Limit: Important for compliance (prevent data exfiltration)
```

---

## Implementation Steps

### Step 1: Create Network Firewall

```bash
# Via AWS Console
# VPC Dashboard → Network Firewalls → Create Network Firewall

# Name: Web-Firewall
# VPC: Production VPC
# Firewall Subnets: Select firewall subnets
  (typically one per AZ)
# Description: Production firewall for web tier

# Firewall Policy:
  - Create new policy or select existing
```

### Step 2: Create Firewall Policy

```bash
# Network Firewall Dashboard → Firewall Policies → Create

# Name: Production-Policy
# Stateless Rules: Create default deny rules
# Stateful Rules: Add rule groups
# Policy Configurations:
  - Logging: Enable CloudWatch Logs
  - Metrics: Enable CloudWatch metrics
```

### Step 3: Create Stateful Rule Group

```bash
# Firewall Rules → Rule Groups → Create Rule Group

# Name: Allow-Web-Traffic
# Type: Stateful
# Capacity: 100 rules

# Rules:
Rule 1: Allow HTTP (port 80)
  src: 0.0.0.0/0, dst: 10.0.0.0/8
  protocol: TCP, port: 80
  action: Pass

Rule 2: Allow HTTPS (port 443)
  src: 0.0.0.0/0, dst: 10.0.0.0/8
  protocol: TCP, port: 443
  action: Pass

Rule 3: Allow SSH (port 22) from admin IPs
  src: 203.0.113.0/24, dst: 10.0.0.0/8
  protocol: TCP, port: 22
  action: Pass

Rule 4: Block known malicious IP
  src: 192.0.2.0/24, dst: any
  action: Drop (with logging)
```

### Step 4: Deploy Firewall Endpoints

```bash
# Network Firewall Dashboard → Firewall Policies
# Select policy → Deploy to VPCs

# Select VPC and subnets
# Create firewall endpoints (one per subnet)
# Note the Firewall Endpoint IDs

# Each endpoint gets a cross-zone load balancer
# Traffic routes through these endpoints
```

### Step 5: Update Route Tables

```bash
# VPC Dashboard → Route Tables

# For Public Subnets:
# Destination: 0.0.0.0/0
# Target: Firewall Endpoint (in firewall subnet)

# For Private Subnets:
# Destination: 0.0.0.0/0
# Target: Firewall Endpoint (for outbound internet access)

# This ensures all traffic flows through firewall
```

### Step 6: Enable Logging

```bash
# Network Firewall → Firewall Configuration
# Logging: Enable CloudWatch Logs

# Log Groups:
# - Alert Log: Detailed firewall decisions
# - Flow Log: Connection summaries

# View logs:
aws logs tail /aws/network-firewall/Production-Policy --follow
```

### Step 7: Test Firewall Rules

```bash
# Test allowed traffic
curl http://web-server-ip  # Should succeed

# Test blocked traffic (from unauthorized IP)
# Configure rule to block specific IP
# Attempt connection from that IP: Should fail

# Monitor logs
aws logs tail /aws/network-firewall/Production-Policy
```

---

## Validation Steps

### Verify Firewall Deployment
```bash
# Check firewall status
aws ec2 describe-network-firewall --firewall-name Web-Firewall

# List firewall endpoints
aws ec2 describe-firewall-endpoints

# Verify route tables point to firewall
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=vpc-xxxxx"
```

### Test Rule Effectiveness
```bash
# From allowed IP
ssh -i key.pem ubuntu@10.0.1.100  # Should work

# From blocked IP
ssh -i key.pem ubuntu@10.0.1.100  # Should timeout/fail

# Verify blocking in logs
aws logs filter-log-events \
  --log-group-name /aws/network-firewall/Production-Policy \
  --filter-pattern "Drop"
```

### Monitor Firewall Metrics
```bash
# CloudWatch metrics
# - Packets Analyzed
# - Packets Dropped
# - Rule Matches
# - Alert Triggered

# View with AWS CLI
aws cloudwatch get-metric-statistics \
  --namespace AWS/NetworkFirewall \
  --metric-name PacketsDropped \
  --start-time 2026-10-11T00:00:00Z \
  --end-time 2026-10-13T00:00:00Z \
  --period 3600 \
  --statistics Sum
```

---

## Commands & Configuration

### AWS CLI Examples

```bash
# Create firewall
aws ec2 create-network-firewall \
  --firewall-name Production-Firewall \
  --vpc-id vpc-xxxxx \
  --firewall-policy-arn arn:aws:network-firewall:region:account:firewall-policy/xxxxx

# Create rule group
aws ec2 create-network-firewall-rule-group \
  --capacity 100 \
  --rule-group-name Allow-Web-Traffic \
  --type STATEFUL

# Update firewall policy
aws ec2 update-network-firewall-firewall-policy \
  --firewall-arn arn:aws:network-firewall:region:account:firewall/xxxxx \
  --firewall-policy-arn arn:aws:network-firewall:region:account:firewall-policy/xxxxx

# Describe firewall
aws ec2 describe-network-firewalls \
  --firewall-names Production-Firewall
```

### Logging Configuration

```bash
# Enable CloudWatch logging
aws logs create-log-group \
  --log-group-name /aws/network-firewall/alerts

aws logs create-log-stream \
  --log-group-name /aws/network-firewall/alerts \
  --log-stream-name alerts

# View logs
aws logs tail /aws/network-firewall/alerts --follow

# Search for dropped packets
aws logs filter-log-events \
  --log-group-name /aws/network-firewall/alerts \
  --filter-pattern "action:drop"
```

---

## What I Learned

### Firewall Fundamentals
1. **Layered Security:** Network Firewall complements Security Groups/NACLs
2. **Stateful vs. Stateless:** Both types needed for comprehensive protection
3. **Central Management:** Single policy for entire VPC
4. **Logging is Critical:** Audit trails for compliance and troubleshooting

### Performance Considerations
1. **Throughput:** Firewall can handle multi-Tbps (adequate for most workloads)
2. **Latency:** Minimal impact (~1-2ms added latency)
3. **Availability:** Multi-AZ deployment for redundancy
4. **Scaling:** Automatically scales with traffic

### Rule Development
1. **Start Permissive:** Begin with allow-all, then restrict
2. **Test Thoroughly:** Firewall rules can break legitimate traffic
3. **Document Rules:** Clear purpose and maintenance notes
4. **Regular Review:** Remove obsolete rules

### Compliance & Audit
1. **Logging:** Essential for compliance audits
2. **Rule Changes:** Track all modifications
3. **Access Control:** Restrict who can modify firewall rules
4. **Metrics:** Monitor firewall effectiveness

---

## Real-World Production Use Cases

### Web Application Firewall
```
Rules:
- Allow HTTPS (443) from anywhere
- Allow HTTP (80) redirect to HTTPS
- Block non-compliant TLS versions
- Block suspicious User-Agents
- Rate limit excessive requests

Result: Protected web application
```

### Microsegmentation
```
Internal Traffic:
- App tier ↔ DB tier: MySQL (3306) only
- Web tier ↔ App tier: Custom app port only
- Admin access: SSH (22) from admin subnet only

External Traffic:
- Internet ↔ Web tier: HTTPS (443) only

Result: Minimal blast radius if tier is compromised
```

### Compliance (PCI-DSS, HIPAA)
```
Rules enforce:
- Encrypted traffic only (TLS 1.2+)
- Audit logging of all access
- Deny access from known malicious IPs
- Rate limiting (prevent DDoS)

Result: Meets regulatory requirements
```

---

## Security Best Practices

### Rule Design
- ✅ Principle of least privilege (default deny)
- ✅ Specific rules before general rules
- ✅ Document each rule's purpose
- ⚠️ Regular rule audits (quarterly minimum)
- ❌ Never leave debugging rules in production

### Logging & Monitoring
- ✅ Enable all logging (alerts + flow logs)
- ✅ Set up CloudWatch alarms for dropped packets
- ✅ Regular log analysis (weekly minimum)
- ✅ Archive logs for compliance (keep 1+ years)
- ⚠️ Be aware of log storage costs

### Deployment
- ✅ Deploy in multiple AZs (HA)
- ✅ Test in staging before production
- ✅ Implement gradual rollout
- ✅ Have rollback plan ready
- ⚠️ Maintain other security layers (don't rely solely on firewall)

---

## Cost Considerations

| Component | Cost |
|-----------|------|
| Network Firewall (per hour) | $1.95 |
| Firewall Endpoints (per AZ/hour) | $0.65 |
| GB Processed | $0.65/GB |
| CloudWatch Logs | $0.50/GB |

**Example Monthly Cost (Multi-AZ, 100GB traffic):**
- Firewall: $1.95 × 730 hours = $1,424
- Endpoints: $0.65 × 2 AZs × 730 = $949
- Processing: 100GB × $0.65 = $65
- Logging: 50GB × $0.50 = $25
- **Total: ~$2,463/month**

---

## Interview Questions

### Foundational
1. **What is AWS Network Firewall?**
   - Managed service providing stateful firewall, IDS/IPS for VPC
2. **How is it different from Security Groups?**
   - Network Firewall: VPC-wide, complex rules. Security Groups: Instance-level, simple rules
3. **When would you use Network Firewall vs. NACLs?**
   - Firewall: Advanced rules, IDS/IPS, compliance. NACLs: Simple port/protocol filtering

### Intermediate
4. **How do you route traffic through Network Firewall?**
   - Update route tables to point to firewall endpoints
5. **What's the performance impact?**
   - Minimal (1-2ms latency); throughput adequate for most workloads
6. **How do you monitor firewall effectiveness?**
   - CloudWatch logs, metrics, and alarms; regular log analysis

### Advanced
7. **How would you design a multi-region firewall architecture?**
   - Deploy firewall in each region; replicate policies; use Route 53 for routing
8. **What's the trade-off between firewall rules and WAF?**
   - Firewall: Network-level, protocol-based. WAF: Application-level, content-based. Use both for defense-in-depth

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 11, 2026)
- **Documentation:** ✅ Complete
- **Validation:** ✅ Firewall rules tested and validated
- **Monitoring:** ✅ CloudWatch logging configured

---

## References

- [AWS Network Firewall Documentation](https://docs.aws.amazon.com/wafv2/latest/developerguide/fmc.html)
- [Firewall Rules Reference](https://docs.aws.amazon.com/network-firewall/latest/developerguide/suricata-rule-evaluation.html)
- [Best Practices](https://docs.aws.amazon.com/network-firewall/latest/developerguide/best-practices.html)
- [Logging & Monitoring](https://docs.aws.amazon.com/network-firewall/latest/developerguide/logging.html)
