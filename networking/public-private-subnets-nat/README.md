# Public & Private Subnets with NAT Gateway

**Task ID:** AWS-003  
**Category:** Networking  
**Status:** ✅ Completed  
**AWS Services:** VPC, EC2, NAT Gateway, Elastic IP, Internet Gateway, Route Tables, Security Groups  
**Date Completed:** October 8, 2026

---

## Objective

Deploy EC2 instances in both public and private subnets, establish the bastion host (jumphost) pattern for accessing private instances, and provide outbound internet access to private instances through a NAT Gateway.

---

## Why Public & Private Subnets with NAT?

Many production architectures require:

- **Public Subnets:** Host bastion hosts, NAT gateways, load balancers (internet-facing)
- **Private Subnets:** Host application servers, databases (no direct internet access)
- **Bastion Host Pattern:** Controlled access to private instances through a public jump server
- **NAT Gateway:** Allows private instances to initiate outbound connections (software updates, API calls) while preventing inbound internet connections

This architecture maximizes security by minimizing the attack surface while maintaining operational functionality.

---

## Architecture Overview

```
Sandpit VPC
├── Internet Gateway (IGW)
├── Public Subnet
│   ├── EC2: Public Server (Bastion Host)
│   │   ├── Public IP: Assigned
│   │   ├── Security Group: Allow SSH from 0.0.0.0/0
│   │   └── User Data: Install packages, create website
│   ├── NAT Gateway
│   │   └── Elastic IP: Attached
│   └── Route Table (Public)
│       ├── 10.0.0.0/16 → Local
│       └── 0.0.0.0/0 → IGW
├── Private Subnet (Demo)
│   ├── EC2: Private Server
│   │   ├── Private IP Only
│   │   ├── Security Group: Allow SSH from Public SG
│   │   └── User Data: Install packages, create website
│   └── Route Table (Private/Demo)
│       ├── 10.0.0.0/16 → Local
│       └── 0.0.0.0/0 → NAT Gateway
```

---

## Configuration Details

### VPC Setup
- **VPC:** Sandpit VPC (created in AWS-001)
- **Public Subnet:** Pre-existing from VPC creation
- **Private Subnet:** Demo Subnet (new, created in this exercise)
- **Internet Gateway:** Connected to Sandpit VPC

### EC2 Instances
#### Public Server (Bastion Host)
- **Instance Name:** Public Server
- **AMI:** Ubuntu Server LTS
- **Instance Type:** t2.micro
- **VPC:** Sandpit VPC
- **Subnet:** Public Subnet
- **Public IP:** Auto-assigned
- **Security Group:** Allow SSH (22) and HTTP (80) from 0.0.0.0/0
- **User Data:** Install packages, create simple website
- **Website Content:** Displays server hostname

#### Private Server
- **Instance Name:** Private Server
- **AMI:** Ubuntu Server LTS
- **Instance Type:** t2.micro
- **VPC:** Sandpit VPC
- **Subnet:** Demo (Private) Subnet
- **Public IP:** None
- **Private IP:** Assigned from subnet CIDR
- **Security Group:** Allow SSH (22) from Public Server SG
- **User Data:** Install packages, create simple website
- **Website Content:** Displays server hostname

### NAT Gateway
- **Location:** Public Subnet
- **Elastic IP:** Allocated and associated
- **Route Table:** Demo Route Table targets NAT Gateway for 0.0.0.0/0

---

## Implementation Steps

### Step 1: Create Demo (Private) Subnet
1. Navigate to **VPC Dashboard** → **Subnets**
2. Click **Create Subnet**
3. **VPC:** Sandpit VPC
4. **Subnet Name:** Demo Subnet
5. **Availability Zone:** Same as public subnet (or different for HA)
6. **IPv4 CIDR Block:** e.g., `10.0.10.0/24` (different from public subnet)
7. Click **Create Subnet**

### Step 2: Create Private Route Table
1. Navigate to **VPC Dashboard** → **Route Tables**
2. Click **Create Route Table**
3. **Name:** Demo Route Table
4. **VPC:** Sandpit VPC
5. Click **Create**
6. Select the new route table
7. Go to **Routes** tab → **Edit Routes**
8. Add Route: `0.0.0.0/0` → **Target: NAT Gateway** (we'll create the NAT next)
9. For now, leave it incomplete (save the route table first)

### Step 3: Create Elastic IP for NAT Gateway
1. Navigate to **EC2 Dashboard** → **Elastic IPs**
2. Click **Allocate Elastic IP Address**
3. **Network Border Group:** Default
4. Click **Allocate**
5. Note the Elastic IP address (we'll use it for NAT Gateway)

### Step 4: Create NAT Gateway
1. Navigate to **VPC Dashboard** → **NAT Gateways**
2. Click **Create NAT Gateway**
3. **Subnet:** Public Subnet (NAT Gateways must be in public subnets)
4. **Elastic IP Allocation ID:** Select the EIP created in Step 3
5. Click **Create NAT Gateway**
6. Wait for NAT Gateway status to change to **Available** (1-2 minutes)

### Step 5: Update Demo Route Table with NAT Gateway
1. Navigate to **Route Tables**
2. Select **Demo Route Table**
3. **Routes** tab → **Edit Routes**
4. For the route `0.0.0.0/0`, **Target:** Select **NAT Gateway** → Choose the NAT Gateway created in Step 4
5. Click **Save Routes**

### Step 6: Associate Demo Route Table with Private Subnet
1. Select **Demo Route Table**
2. **Subnet Associations** tab → **Edit Subnet Associations**
3. Select **Demo Subnet** (the private one)
4. Click **Save Associations**

### Step 7: Create Security Groups

#### Public SG (for Public Server)
1. Navigate to **EC2 Dashboard** → **Security Groups**
2. Click **Create Security Group**
3. **Name:** Public-SG
4. **VPC:** Sandpit VPC
5. **Inbound Rules:**
   - SSH (22) from 0.0.0.0/0
   - HTTP (80) from 0.0.0.0/0
6. **Outbound Rules:** All (default)
7. Click **Create**

#### Private SG (for Private Server)
1. Click **Create Security Group**
2. **Name:** Private-SG
3. **VPC:** Sandpit VPC
4. **Inbound Rules:**
   - SSH (22) from **Public-SG** (source: the public security group)
5. **Outbound Rules:** All (default)
6. Click **Create**

### Step 8: Launch Public Server
1. Navigate to **EC2 Dashboard** → **Instances**
2. Click **Launch Instances**
3. **Name:** Public Server
4. **AMI:** Ubuntu Server 22.04 LTS
5. **Instance Type:** t2.micro
6. **VPC:** Sandpit VPC
7. **Subnet:** Public Subnet
8. **Auto-assign Public IP:** Enable
9. **Security Group:** Public-SG
10. **User Data** (Advanced Details):
```bash
#!/bin/bash
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
echo "<h1>Public Server - $(hostname)</h1>" | sudo tee /var/www/html/index.html
```
11. Click **Launch Instance**

### Step 9: Launch Private Server
1. Click **Launch Instances**
2. **Name:** Private Server
3. **AMI:** Ubuntu Server 22.04 LTS
4. **Instance Type:** t2.micro
5. **VPC:** Sandpit VPC
6. **Subnet:** Demo Subnet (private)
7. **Auto-assign Public IP:** Disable
8. **Security Group:** Private-SG
9. **User Data:**
```bash
#!/bin/bash
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
echo "<h1>Private Server - $(hostname)</h1>" | sudo tee /var/www/html/index.html
```
10. Click **Launch Instances**

### Step 10: Test Public Server Connectivity
```bash
# Get public server IP from EC2 dashboard
ssh -i your-key.pem ubuntu@<public-ip>

# Once connected, test internet access
ping 8.8.8.8
ping google.com

# Test web server
curl localhost
```

### Step 11: Test Bastion Host Pattern (Access Private Server)

```bash
# On your local machine, SSH to public server
ssh -i your-key.pem ubuntu@<public-ip>

# Once on public server, create key file
cat > ~/.ssh/private-key.pem << 'EOF'
[Paste your private key content here]
EOF
chmod 400 ~/.ssh/private-key.pem

# SSH from public server to private server (using private IP)
ssh -i ~/.ssh/private-key.pem ubuntu@<private-ip>

# Once on private server, test internet access
ping 8.8.8.8
ping google.com

# Test web server
curl localhost
```

---

## Commands & Configuration

### SSH Bastion Pattern
```bash
# One-liner: SSH through bastion (ProxyJump)
ssh -i key.pem -J ubuntu@<public-ip> ubuntu@<private-ip>

# Or traditional: Step 1 - SSH to public server
ssh -i key.pem ubuntu@<public-ip>

# Step 2 - From public server, SSH to private server
ssh -i key.pem ubuntu@<private-ip>
```

### Connectivity Testing
```bash
# Test DNS resolution
nslookup 8.8.8.8
dig google.com

# Test internet connectivity
ping 8.8.8.8
ping google.com
curl https://example.com

# Test internal connectivity (from public to private)
ping <private-ip>
ssh -i key.pem ubuntu@<private-ip>
```

### User Data for EC2 Instances
```bash
#!/bin/bash
# Update package lists
sudo apt update

# Upgrade packages
sudo apt upgrade -y

# Install nginx
sudo apt install -y nginx

# Start nginx
sudo systemctl start nginx

# Enable nginx on boot
sudo systemctl enable nginx

# Create simple status page
echo "<h1>Server: $(hostname)</h1>" | sudo tee /var/www/html/index.html
```

---

## Validation Steps

### Verify NAT Gateway
1. Navigate to **VPC Dashboard** → **NAT Gateways**
2. Confirm NAT Gateway shows **State: Available**
3. Confirm Elastic IP is associated
4. Confirm it's in the Public Subnet

### Verify Route Tables
1. **Public Route Table:**
   - 10.0.0.0/16 → local
   - 0.0.0.0/0 → IGW
2. **Private Route Table:**
   - 10.0.0.0/16 → local
   - 0.0.0.0/0 → NAT Gateway

### Verify EC2 Instances
```bash
# Public server should have public IP
aws ec2 describe-instances --filters "Name=tag:Name,Values=Public Server"

# Private server should NOT have public IP
aws ec2 describe-instances --filters "Name=tag:Name,Values=Private Server"
```

### Verify Connectivity
1. **Public Server → Internet**
   ```bash
   ssh -i key.pem ubuntu@<public-ip>
   ping 8.8.8.8  # Should succeed
   ```

2. **Public Server ↔ Private Server**
   ```bash
   # From public server
   ping <private-ip>  # Should succeed
   ```

3. **Private Server → Internet (through NAT)**
   ```bash
   # From private server (via bastion)
   ping 8.8.8.8  # Should succeed (via NAT Gateway)
   ```

---

## What I Learned

### Bastion Host Pattern is Essential
- Private instances cannot be accessed directly from the internet
- Using a public bastion host as a jump server is the standard pattern for accessing private instances
- This provides a single control point for security auditing and monitoring

### NAT Gateway Enables Outbound Internet
- IAM permissions are necessary but not sufficient; network connectivity is equally important
- A private instance with IAM permissions to S3 still cannot access S3 without:
  - A NAT Gateway (for internet access), OR
  - A VPC Gateway Endpoint (for private path to AWS services)
- NAT Gateways allow private instances to:
  - Download software updates (apt update, yum update)
  - Call external APIs
  - Upload logs and metrics to monitoring services

### Route Tables Control Traffic Flow
- Route tables are the "traffic managers" of your VPC
- The same subnet can behave differently based on its route table
- A route table with an IGW route = public subnet
- A route table with a NAT route = private subnet with outbound internet

### Security Group Rules are Directional
- Inbound rules control who can connect TO the instance
- Outbound rules control who the instance can connect to
- For the bastion pattern: Private instance's security group must allow inbound SSH from the Public instance's security group

### ElasticIP and NAT Gateways
- ElasticIPs must be allocated even if they're only used for NAT
- NAT Gateways charge for both the gateway itself ($0.045/hour) AND for data processed ($0.045/GB)
- This makes NAT Gateways expensive; use VPC Endpoints when possible for AWS service access

---

## Real-World Production Use Cases

### Three-Tier Web Application
```
Public Tier:
- Load Balancer in public subnet
- NAT Gateway in public subnet

Application Tier:
- Application servers in private subnets (auto-scaling group)
- Behind load balancer
- Access private tier through internal load balancer

Data Tier:
- RDS database in private subnet
- Multi-AZ for high availability
- Only accessible from application tier
```

### Microservices Architecture
```
Bastion Host:
- One public bastion for operator access
- Used to manage/debug private microservices

Microservices:
- All run in private subnets
- Scale horizontally without public IPs
- Communicate through service mesh or internal load balancers

External Traffic:
- Ingress through ALB/NLB in public subnet
- Routed to microservices in private subnet
```

### Hybrid Cloud with VPN
```
Private Data Center:
- Connected via VPN through NAT Gateway
- Access private AWS resources without internet

AWS Private Subnets:
- Host sensitive data and applications
- NAT Gateway provides outbound internet when needed
- Bastion host for emergency access
```

---

## Security Considerations

### Bastion Host Security
- ✅ Restrict SSH to bastion from known IPs only (not 0.0.0.0/0 in production)
- ✅ Use SSH keys, disable password authentication
- ✅ Enable VPC Flow Logs to audit traffic
- ✅ Use Session Manager (SSM) as alternative to SSH
- ⚠️ Consider IP whitelisting for bastion access

### NAT Gateway Security
- ✅ NAT Gateway hides private instance IPs from internet (IP masquerading)
- ⚠️ All private instances using the same NAT share the Elastic IP
- ⚠️ NAT Gateway is a chokepoint; monitor for DDoS attacks
- ✅ Use NACLs and Security Groups for additional protection

### Private Subnet Security
- ✅ Private subnets reduce attack surface by design
- ✅ Only accessible from bastion or internal resources
- ⚠️ Still need Security Group restrictions (defense in depth)
- ✅ Use VPC Endpoints to avoid internet for AWS service access

### Best Practices
- Defense in depth: Security Groups + NACLs + Route Tables
- Monitor NAT Gateway CloudWatch metrics for anomalies
- Use VPC Flow Logs to troubleshoot connectivity
- Regularly audit security group rules

---

## Cost Considerations

| Component | Cost | Notes |
|-----------|------|-------|
| NAT Gateway | $0.045/hour | Only charged while running |
| NAT Gateway Data Processing | $0.045/GB | For all traffic through NAT |
| Elastic IP (unused) | $0.005/hour | Only if not associated |
| Elastic IP (in-use) | Free | When associated with NAT or instance |
| EC2 t2.micro | ~$0.0116/hour | May qualify for free tier |

**Example Costs:**
- 1 NAT Gateway for 24 hours: $1.08
- 100GB data through NAT: $4.50
- 2 t2.micro instances for 24 hours: ~$0.56
- **Total for 1 day: ~$6-7**

---

## Resource Cleanup

```bash
# 1. Terminate EC2 instances
# EC2 Dashboard → Instances → Select both → Instance State → Terminate

# 2. Delete NAT Gateway (after instances are terminated)
# VPC Dashboard → NAT Gateways → Select NAT → Delete

# 3. Release Elastic IP
# EC2 Dashboard → Elastic IPs → Select EIP → Release

# 4. Delete Private Route Table
# VPC Dashboard → Route Tables → Select Demo Route Table → Delete

# 5. Delete Private Subnet
# VPC Dashboard → Subnets → Select Demo Subnet → Delete

# 6. Delete Security Groups
# EC2 Dashboard → Security Groups → Delete Public-SG and Private-SG
```

---

## Interview Questions

### Foundational
1. **What is a bastion host and why would you use one?**
   - Answer: A bastion host is a public EC2 instance used as a jump server to access private instances. It provides a single, auditable access point and keeps private resources completely isolated from direct internet access.

2. **What is the difference between a bastion host pattern and a NAT Gateway?**
   - Answer: Bastion hosts control inbound access to private instances (operator access). NAT Gateways enable outbound internet access from private instances.

3. **Can a private instance be accessed without a bastion host?**
   - Answer: Only through other methods like AWS Systems Manager Session Manager, VPN connectivity, or AWS PrivateLink. The bastion pattern is traditional SSH access.

### Intermediate
4. **How does a NAT Gateway hide private IP addresses?**
   - Answer: The NAT Gateway performs IP masquerading: it translates the private instance's source IP to the NAT Gateway's Elastic IP for outbound packets, and reverse-translates inbound responses.

5. **Why can't a private instance access S3 even with an IAM role if there's no NAT Gateway?**
   - Answer: IAM roles control permissions but don't provide network connectivity. A private instance has no route to the internet, so it cannot reach S3. Either a NAT Gateway or S3 Gateway Endpoint is needed.

6. **What happens if you delete a NAT Gateway while instances are using it for internet access?**
   - Answer: Private instances lose outbound internet connectivity immediately. Active connections are dropped; new connections fail.

### Advanced
7. **How would you design a highly available bastion host setup?**
   - Answer: Deploy bastion hosts in multiple AZs behind an NLB, or use an Auto Scaling Group that maintains at least 2 instances across AZs. Route53 can provide health-checked access.

8. **What are the trade-offs between NAT Gateways, NAT instances, and VPC Endpoints?**
   - NAT Gateway: Fully managed, high availability, but expensive ($0.045/GB)
   - NAT Instance: Self-managed, cheaper, but needs manual configuration and HA
   - VPC Endpoint: Most cost-effective for AWS services, but only works with AWS APIs (S3, DynamoDB, etc.)

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 8, 2026)
- **Documentation:** ✅ Complete
- **Validation:** ✅ Verified (bastion access working, private internet through NAT confirmed)
- **Screenshots:** 📸 Available in `./screenshots/` directory

---

## Related Tasks

- **Previous:** [VPC Subnets, Route Tables & IGW](../vpc-subnets-routes-igw/) - Prerequisites
- **Next:** [VPC Endpoints & S3 Access](../../application-services/vpc-endpoint-s3/) - Alternative to NAT for AWS services
- **Related:** [VPC Peering Multi-Region](../vpc-peering-multiregion/) - Advanced networking

---

## References

- [NAT Gateways Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Bastion Host Architecture](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/welcome.html)
- [VPC Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security.html)
