# VPC Subnets, Route Tables & Internet Gateway

**Task ID:** AWS-002  
**Category:** Networking  
**Status:** ✅ Completed  
**AWS Services:** VPC, EC2, Internet Gateway, Route Tables, Security Groups, Elastic IPs  
**Date Completed:** October 7, 2026

---

## Objective

Configure subnets with route tables and an Internet Gateway to enable EC2 instances in public subnets to communicate with the internet. Deploy a web server and verify internet connectivity.

---

## Why Route Tables and Internet Gateways Matter

In AWS networking, creating a VPC and subnets is just the beginning. Traffic between subnets and the internet is controlled by:

- **Route Tables:** Define where traffic destined for specific IP ranges should be directed
- **Internet Gateway (IGW):** The gateway between your VPC and the public internet
- **Subnet Associations:** Link route tables to subnets, making routing rules effective

Without proper route table configuration and IGW attachment, resources in public subnets cannot communicate with the internet.

---

## Architecture Overview

```
Testing VPC
├── Internet Gateway (IGW)
│   └── Attached to VPC
├── Route Table
│   ├── Local Route: VPC CIDR → Local
│   └── Internet Route: 0.0.0.0/0 → IGW
├── Public Subnet 1
│   ├── Route Table Association: Custom Route Table
│   └── EC2 Instance (Testing VM - nginx)
├── Public Subnet 2
│   ├── Route Table Association: Custom Route Table
│   └── (Ready for additional instances)
└── Public Subnet 3
    ├── Route Table Association: Custom Route Table
    └── (Ready for additional instances)
```

---

## Configuration Details

### Testing VPC Setup
- **VPC Name:** Testing VPC
- **CIDR Block:** Custom (user-defined)
- **Availability Zones:** 1 or more
- **Public Subnets:** 3
- **Auto-assign Public IPs:** Enabled on all public subnets

### Internet Gateway Configuration
- **IGW Name:** Testing IGW
- **Attachment:** Attached to Testing VPC
- **Status:** Active

### Route Table Configuration
- **Route Table Name:** Testing Route Table
- **Routes:**
  - **Local Route:** VPC CIDR → Local (automatic)
  - **Internet Route:** 0.0.0.0/0 → Internet Gateway
- **Subnet Associations:** All 3 public subnets

### EC2 Instance (Testing VM)
- **AMI:** Ubuntu (or similar Linux)
- **Instance Type:** t2.micro or similar
- **VPC:** Testing VPC
- **Subnet:** Public Subnet (Zone 1)
- **Security Group:** Testing SG (allows HTTP/HTTPS/SSH)
- **Public IP:** Auto-assigned
- **Web Server:** nginx

---

## Implementation Steps

### Step 1: Create Internet Gateway
1. Navigate to **VPC Dashboard** → **Internet Gateways**
2. Click **Create Internet Gateway**
3. Name it: "Testing IGW"
4. Click **Create**
5. Select the newly created IGW
6. Click **Attach to VPC**
7. Choose the Testing VPC
8. Click **Attach**
9. Verify status shows "Available"

### Step 2: Create Route Table
1. Navigate to **VPC Dashboard** → **Route Tables**
2. Click **Create Route Table**
3. Name: "Testing Route Table"
4. VPC: Select Testing VPC
5. Click **Create**

### Step 3: Add Internet Route to Route Table
1. Select the Testing Route Table
2. Click **Routes** tab
3. Click **Edit Routes**
4. Click **Add Route**
5. **Destination:** `0.0.0.0/0`
6. **Target:** Select Internet Gateway → Testing IGW
7. Click **Save Routes**

### Step 4: Associate Route Table with Public Subnets
1. Select the Testing Route Table
2. Click **Subnet Associations** tab
3. Click **Edit Subnet Associations**
4. Select all 3 public subnets
5. Click **Save Associations**

### Step 5: Create Security Group
1. Navigate to **EC2 Dashboard** → **Security Groups**
2. Click **Create Security Group**
3. Name: "Testing SG"
4. Description: "Security group for testing instances"
5. VPC: Testing VPC
6. **Inbound Rules:**
   - SSH (port 22) from 0.0.0.0/0 (or your IP)
   - HTTP (port 80) from 0.0.0.0/0
   - HTTPS (port 443) from 0.0.0.0/0
7. **Outbound Rules:** All traffic allowed (default)
8. Click **Create**

### Step 6: Launch EC2 Instance
1. Navigate to **EC2 Dashboard** → **Instances**
2. Click **Launch Instance**
3. **Name:** Testing VM
4. **AMI:** Ubuntu Server (latest LTS)
5. **Instance Type:** t2.micro
6. **VPC:** Testing VPC
7. **Subnet:** Public Subnet (Zone 1)
8. **Auto-assign Public IP:** Enable
9. **Security Group:** Testing SG
10. **Key Pair:** Create new or use existing
11. Click **Launch**

### Step 7: Connect to Instance and Install nginx
```bash
# SSH into the instance
ssh -i your-key.pem ubuntu@<public-ip>

# Update packages
sudo apt update
sudo apt upgrade -y

# Install nginx
sudo apt install nginx -y

# Start nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify nginx is running
sudo systemctl status nginx
```

### Step 8: Customize nginx Web Page
```bash
# Edit the default nginx page
sudo nano /var/www/html/index.nginx-debian.html

# Replace default content with custom HTML
# Add your own content, save (Ctrl+X → Y → Enter)

# Verify changes
curl localhost
```

---

## Validation Steps

### Verify Internet Gateway
1. Navigate to **Internet Gateways**
2. Confirm "Testing IGW" shows **State: Available**
3. Confirm **VPC ID** shows Testing VPC is attached

### Verify Route Table
1. Navigate to **Route Tables**
2. Select Testing Route Table
3. **Routes tab:** Verify two routes exist:
   - 10.0.0.0/16 → local (VPC CIDR)
   - 0.0.0.0/0 → igw-xxxxx (Internet Gateway)
4. **Subnet Associations tab:** Verify all 3 public subnets are listed

### Verify EC2 Instance
1. Navigate to **Instances**
2. Select Testing VM
3. Confirm:
   - **State:** Running
   - **Public IPv4 Address:** Assigned
   - **Subnet:** Public Subnet
   - **Security Groups:** Testing SG

### Test Internet Connectivity
```bash
# SSH into the instance
ssh -i your-key.pem ubuntu@<public-ip>

# Test internet connectivity
ping 8.8.8.8
ping google.com

# Update packages (requires internet)
sudo apt update
```

### Test Web Server
1. Open browser
2. Navigate to: `http://<public-ip>`
3. Verify nginx welcome page (or custom page) loads
4. Confirm your custom content displays

---

## Commands & Configuration

### Key Commands Used
```bash
# Check instance connectivity
ping <instance-public-ip>

# SSH into instance
ssh -i key.pem ubuntu@<public-ip>

# On the instance - update packages
sudo apt update && sudo apt upgrade -y

# Install nginx
sudo apt install nginx -y

# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check nginx status
sudo systemctl status nginx
systemctl is-active nginx

# View nginx default page location
ls -la /var/www/html/

# Test nginx locally
curl localhost
curl <public-ip>
```

### Important Configuration Files
| File/Location | Purpose |
|---------------|---------|
| `/var/www/html/` | nginx web content directory |
| `/etc/nginx/nginx.conf` | nginx main configuration |
| `/var/log/nginx/access.log` | nginx access logs |
| `/var/log/nginx/error.log` | nginx error logs |

---

## What I Learned

### Route Tables are Critical
- Without a route table associated with a subnet, the subnet is "private" (no internet access)
- Route tables control the flow of traffic; they are the "routing intelligence" of your VPC
- Each route entry specifies a destination CIDR and a target (IGW, NAT, VPN, peering connection, etc.)

### IGW is the Internet Bridge
- An Internet Gateway must be explicitly created and attached to a VPC to enable internet communication
- Creating an IGW alone doesn't make your subnets public; you must also add a route to the IGW in your route table
- Only one IGW can be attached to a VPC at a time

### Public Subnet Definition
- A subnet is "public" if:
  1. Its associated route table has a route to an Internet Gateway for 0.0.0.0/0
  2. Instances have public IPs (either auto-assigned or Elastic IPs)
- A subnet is "private" if it doesn't have an IGW route

### Instance Connectivity Requirements
- **Security Groups** act as instance-level firewalls; they must allow inbound traffic (SSH, HTTP, etc.)
- **NACLs** act as subnet-level firewalls (covered in later exercises)
- **Route Tables** determine if traffic can flow to/from the internet
- All three must be correctly configured for internet connectivity

### DNS and Name Resolution
- AWS provides DNS resolution for public IPs by default
- When you access an instance by public IP, AWS DNS resolves the IP correctly
- When instances update packages (apt update), they query DNS to reach package repositories

---

## Real-World Production Use Cases

### Web Application Deployment
```
Production Setup:
- Load Balancer in public subnet (handles internet traffic)
- Web Servers in private subnets (no direct internet, accessed via load balancer)
- NAT Gateway for private subnet outbound internet (for updates, external API calls)
Result: Secure, scalable web application
```

### Bastion Host / Jump Server Pattern
```
Use Case:
- Public Bastion Host in public subnet (SSH access from internet)
- Private Application Servers in private subnets (accessed through bastion)
- No direct internet for application servers
Result: Controlled, audited access to internal servers
```

### Multi-Tier Application
```
Architecture:
- Public Tier: Web servers (nginx, Apache)
- Private Tier: Application servers (Node.js, Python, Java)
- Private Tier: Databases (RDS)
Result: Layered security, isolated tiers
```

---

## Security Considerations

### Security Groups Best Practices
- ✅ Restrict SSH to specific IP addresses (not 0.0.0.0/0 in production)
- ✅ Only open ports that are necessary (HTTP/HTTPS for web servers)
- ⚠️ Default security group allows all outbound traffic; restrict if needed
- ❌ Never open all ports (0-65535) unless absolutely necessary

### Route Table Security
- ✅ Review route tables regularly to ensure no unintended routes
- ✅ Use route tables to enforce traffic patterns (e.g., force traffic through firewalls)
- ⚠️ Ensure no routes point to incorrect targets

### Internet Gateway Access
- ✅ Only attach IGWs to VPCs that need internet access
- ⚠️ Monitor traffic flowing through IGWs
- ❌ Never make internal-only services accessible through IGW

### Public IP Management
- ✅ Use Elastic IPs only when needed (they cost money if not associated)
- ⚠️ Be mindful of public IP exposure; restrict SSH access
- ✅ Use NACLs to add subnet-level protection

---

## Cost Considerations

| Component | Cost | Notes |
|-----------|------|-------|
| IGW | Free | No charge for creating/using IGW |
| Route Table | Free | Basic route tables included |
| EC2 Instance (t2.micro) | ~$0.0116/hour | May qualify for free tier |
| Elastic IPs (if used) | $0.005/hour (unused), Free (in-use) | Only charge if not associated |
| Data Transfer | $0.09/GB (outbound) | Downloading packages, serving web traffic |

**Estimated cost for this exercise:** $0.50-$2.00 (for a few hours of t2.micro instance)

---

## Resource Cleanup

To delete all resources created in this exercise:

```bash
# 1. Terminate EC2 instance
# Navigate to EC2 Dashboard → Instances
# Select Testing VM → Instance State → Terminate

# 2. Delete Security Group (after instance is terminated)
# EC2 Dashboard → Security Groups
# Select Testing SG → Actions → Delete Security Group

# 3. Detach Internet Gateway
# VPC Dashboard → Internet Gateways
# Select Testing IGW → Actions → Detach from VPC

# 4. Delete Internet Gateway
# VPC Dashboard → Internet Gateways
# Select Testing IGW → Actions → Delete Internet Gateway

# 5. Delete Route Table
# VPC Dashboard → Route Tables
# Select Testing Route Table → Actions → Delete Route Table

# 6. Delete VPC (or keep for future exercises)
# VPC Dashboard → Your VPCs
# Select Testing VPC → Actions → Delete VPC
```

---

## Interview Questions

### Foundational
1. **What is the difference between a route table and a security group?**
   - Answer: Route tables control network-level routing (where packets go). Security groups control access at the instance level (which traffic is allowed).

2. **How do you make a subnet public?**
   - Answer: Create a route table with a route to an Internet Gateway for 0.0.0.0/0, then associate that route table with the subnet.

3. **What happens if you don't attach an IGW to a VPC?**
   - Answer: Instances in the VPC cannot communicate with the internet, even if they have public IPs.

### Intermediate
4. **Can you have multiple Internet Gateways attached to a single VPC?**
   - Answer: No. Each VPC can have at most one IGW attached at a time.

5. **What is the difference between auto-assigned public IPs and Elastic IPs?**
   - Answer: Auto-assigned public IPs are temporary and released when the instance stops. Elastic IPs are persistent and can be reassociated to other instances.

6. **How does a route table determine which route to use if multiple routes match?**
   - Answer: AWS uses longest prefix match; the most specific route (longest CIDR prefix) takes precedence.

### Advanced
7. **How would you architect a scenario where private instances need internet access for updates but shouldn't be directly accessible from the internet?**
   - Answer: Use a NAT Gateway in a public subnet and route private subnet traffic through the NAT. This provides outbound internet access without inbound access.

8. **What is the purpose of the local route (10.0.0.0/16 → local) in a route table?**
   - Answer: The local route allows instances in the VPC to communicate with each other directly within the VPC CIDR block. It's automatically created and cannot be modified.

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 7, 2026)
- **Documentation:** ✅ Complete
- **Validation:** ✅ Verified (IGW attached, route tables configured, nginx running)
- **Screenshots:** 📸 Available in `./screenshots/` directory

---

## Related Tasks

- **Previous:** [VPC Creation with Multi-AZ Setup](../vpc-creation/) - Prerequisite for this task
- **Next:** [Public/Private Subnets & NAT](../public-private-subnets-nat/) - Add private subnet internet access
- **Security:** [Security Groups & NACLs](../../security/sg-nacl/) - Fine-grained traffic control

---

## References

- [AWS Route Tables Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Internet Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [EC2 Instance Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)
- [Security Groups for EC2](https://docs.aws.amazon.com/AWSEC2/latest/SecurityGuide/ec2-security.html)
