# VPC Endpoints & S3 Access Pattern

**Task ID:** AWS-005  
**Category:** Application Services  
**Status:** ✅ Completed  
**AWS Services:** VPC, VPC Endpoints (Gateway), S3, EC2, IAM, Security Groups  
**Date Completed:** October 8, 2026

---

## Objective

Demonstrate how to provide private EC2 instances with secure access to S3 using VPC Gateway Endpoints, highlighting the difference between IAM permissions and network connectivity.

---

## Why VPC Endpoints Matter

A common misconception in AWS is that IAM permissions alone grant access to AWS services. This exercise illustrates a critical principle:

**IAM Permissions + Network Connectivity = Access to AWS Services**

Without both components, access fails. VPC Endpoints provide:
- **Private connectivity** to AWS services without internet access
- **Cost efficiency** by avoiding NAT Gateway charges ($0.045/GB)
- **Security** by keeping traffic within the AWS network
- **Performance** improvements over internet-based access

---

## Architecture Overview

```
VPC
├── Public Subnet
│   ├── Internet Gateway
│   ├── EC2: Public Instance
│   │   ├── Public IP: Yes
│   │   ├── IAM Role: S3ReadAccess
│   │   ├── Route: 0.0.0.0/0 → IGW
│   │   └── Access Path: IGW → S3 (internet)
│   └── NAT Gateway (not used in this design)
├── Private Subnet
│   ├── EC2: Private Instance
│   │   ├── Public IP: No
│   │   ├── IAM Role: S3ReadAccess
│   │   ├── Route: 0.0.0.0/0 → NAT (initially)
│   │   └── Route: s3 → VPC Endpoint (after configuration)
│   └── VPC Endpoint (S3 Gateway)
│       ├── Type: Gateway Endpoint
│       ├── Service: S3
│       └── Associated Route Table: Private Route Table
└── S3 Bucket
    ├── Block Public Access: Enabled
    ├── Policy: Allow access from VPC Endpoint
    └── Objects: Test file for download
```

---

## Configuration Details

### VPC Setup
- **VPC Name:** Learning VPC
- **CIDR Block:** Custom (e.g., 10.0.0.0/16)
- **Public Subnet:** For public EC2 instance
- **Private Subnet:** For private EC2 instance
- **Internet Gateway:** Connected to VPC
- **NAT Gateway:** Optional (not required for this design)

### S3 Configuration
- **Bucket Name:** Unique name (e.g., my-learning-bucket-20261013)
- **Region:** Same as VPC
- **Block Public Access:** All enabled
- **Versioning:** Optional
- **Encryption:** Default (SSE-S3)
- **Bucket Policy:** Allows access from VPC Endpoint

### EC2 Instances
#### Public Instance
- **Name:** Public EC2
- **AMI:** Amazon Linux 2 (has AWS CLI pre-installed)
- **Instance Type:** t2.micro
- **VPC:** Learning VPC
- **Subnet:** Public Subnet
- **Public IP:** Auto-assigned
- **IAM Role:** S3ReadAccess (custom role with S3 read permissions)
- **Security Group:** Allow SSH (22) and outbound HTTPS (443)

#### Private Instance
- **Name:** Private EC2
- **AMI:** Amazon Linux 2
- **Instance Type:** t2.micro
- **VPC:** Learning VPC
- **Subnet:** Private Subnet
- **Public IP:** None
- **IAM Role:** S3ReadAccess (same role as public instance)
- **Security Group:** Allow SSH (22) from public instance

### VPC Endpoint (S3 Gateway)
- **Type:** Gateway Endpoint
- **Service:** S3 (com.amazonaws.region.s3)
- **VPC:** Learning VPC
- **Route Table Associations:** Private Route Table
- **Policy:** Allow all S3 actions (or restrict as needed)

---

## Implementation Steps

### Step 1: Create S3 Bucket
```bash
# Via AWS Console
# S3 Dashboard → Create Bucket
# Name: my-learning-bucket-20261013 (must be globally unique)
# Region: Same as VPC region
# Block Public Access: All enabled
# Create Bucket

# Upload a test file
# Click bucket → Upload → Select file
```

### Step 2: Create VPC and Subnets
1. Create VPC: Learning VPC (10.0.0.0/16)
2. Create Public Subnet: 10.0.1.0/24
3. Create Private Subnet: 10.0.2.0/24
4. Create Internet Gateway and attach to VPC
5. Create and configure route tables (as in previous exercises)

### Step 3: Create IAM Role with S3 Permissions
```bash
# Via AWS Console
# IAM Dashboard → Roles → Create Role
# Trusted Entity: EC2
# Permissions Policy: AmazonS3ReadOnlyAccess (built-in policy)
# Role Name: S3ReadAccessRole
# Create Role
```

Or via AWS CLI:
```bash
# Create trust policy
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
  --role-name S3ReadAccessRole \
  --assume-role-policy-document file://trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name S3ReadAccessRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name S3ReadAccessProfile

aws iam add-role-to-instance-profile \
  --instance-profile-name S3ReadAccessProfile \
  --role-name S3ReadAccessRole
```

### Step 4: Launch EC2 Instances with IAM Role

#### Public Instance
```bash
# Via AWS Console
# EC2 Dashboard → Launch Instances
# Name: Public EC2
# AMI: Amazon Linux 2 (ami-xxxxx)
# Instance Type: t2.micro
# VPC: Learning VPC
# Subnet: Public Subnet
# Public IP: Enable
# IAM Instance Profile: S3ReadAccessProfile
# Security Group: Allow SSH (22), HTTPS (443)
# Launch Instance
```

#### Private Instance
```bash
# Same as Public Instance, but:
# Subnet: Private Subnet
# Public IP: Disable
```

### Step 5: Create VPC Endpoint for S3
```bash
# Via AWS Console
# VPC Dashboard → Endpoints → Create Endpoint
# Service: S3 (com.amazonaws.region.s3) - Gateway type
# VPC: Learning VPC
# Route table associations: Select Private Route Table
# Policy: Full access (or custom)
# Create Endpoint

# Verify endpoint is in "Available" state
```

Or via AWS CLI:
```bash
# Create endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.region.s3 \
  --route-table-ids rtb-xxxxx
```

### Step 6: Test S3 Access from Public Instance

```bash
# SSH into public instance
ssh -i your-key.pem ec2-user@<public-ip>

# Configure AWS CLI
aws configure
# Leave all prompts blank (IAM role will handle credentials)

# List S3 buckets
aws s3 ls

# Download file from bucket
aws s3 cp s3://my-learning-bucket-20261013/test-file.txt .

# Verify download
cat test-file.txt
```

### Step 7: Test S3 Access from Private Instance (Bastion Pattern)

```bash
# From your local machine, SSH to public instance first
ssh -i your-key.pem ec2-user@<public-ip>

# From public instance, create key file and SSH to private instance
cat > ~/.ssh/private-key.pem << 'EOF'
[Paste private key content]
EOF
chmod 400 ~/.ssh/private-key.pem

# SSH to private instance
ssh -i ~/.ssh/private-key.pem ec2-user@<private-ip>

# Now on private instance - try to access S3 WITHOUT VPC Endpoint
aws configure
aws s3 ls
# This should FAIL or timeout (no internet connectivity, no NAT Gateway)

# Verify the failure
# Error: Unable to locate credentials / Connection timeout
```

### Step 8: Verify VPC Endpoint Works

```bash
# After VPC Endpoint is created and route table associated
# SSH back to private instance
ssh -i ~/.ssh/private-key.pem ec2-user@<private-ip>

# Try S3 access again
aws s3 ls

# Download file through endpoint
aws s3 cp s3://my-learning-bucket-20261013/test-file.txt .

# Verify download (should now succeed)
cat test-file.txt
```

---

## Validation Steps

### Verify IAM Role Attachment
```bash
# From EC2 instance
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Should return the role name

curl http://169.254.169.254/latest/meta-data/iam/security-credentials/S3ReadAccessRole/
# Should return temporary credentials
```

### Verify VPC Endpoint
```bash
# Via AWS CLI
aws ec2 describe-vpc-endpoints \
  --filters "Name=vpc-id,Values=vpc-xxxxx"

# Check status and route table associations
```

### Verify Route Table Configuration
```bash
# Via AWS CLI
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=vpc-xxxxx"

# Look for routes targeting the VPC Endpoint
# Example: 10.0.0.0/16 → local, s3 → vpce-xxxxx
```

### Test S3 Access with Debugging
```bash
# Enable AWS CLI debug mode
aws s3 ls --debug

# Check route to S3
# On Linux: traceroute s3.amazonaws.com
# Or use: aws ec2 describe-vpc-endpoints

# Monitor VPC Flow Logs (if enabled)
aws ec2 describe-flow-logs
```

---

## Commands & Configuration

### AWS CLI Commands
```bash
# List S3 buckets
aws s3 ls

# List objects in bucket
aws s3 ls s3://bucket-name/

# Copy file from S3 to EC2
aws s3 cp s3://bucket-name/file.txt .

# Copy file from EC2 to S3
aws s3 cp file.txt s3://bucket-name/

# Sync directory
aws s3 sync s3://bucket-name/ /local/path/

# Check current credentials
aws sts get-caller-identity

# Check role information
aws iam get-role --role-name S3ReadAccessRole
```

### Debugging Connectivity
```bash
# Test DNS resolution
nslookup s3.amazonaws.com
dig s3.amazonaws.com

# Test connectivity to S3 endpoint
curl -I https://s3.amazonaws.com
curl -I https://s3.region.amazonaws.com

# Check route tables
route -n | grep s3
# Or: netstat -rn

# Monitor active connections
netstat -tuln
ss -tulpn

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx
```

---

## What I Learned

### IAM ≠ Network Connectivity
- **Critical lesson:** IAM roles provide **permissions**, not **connectivity**
- A private EC2 instance with full S3 permissions cannot access S3 without:
  - Network path to S3 (either through NAT Gateway or VPC Endpoint)
  - Correct security group and NACL rules
- This is a frequent source of confusion in production troubleshooting

### VPC Endpoints are Cost-Efficient
- **NAT Gateway:** $0.045/hour + $0.045/GB (expensive for high-volume access)
- **VPC Endpoint:** Free (no hourly charge), more efficient than NAT
- **Best practice:** Use VPC Endpoints for AWS service access, NAT Gateways for general internet

### Two Types of VPC Endpoints
- **Gateway Endpoints:** S3 and DynamoDB (no additional cost, integrated into route tables)
- **Interface Endpoints:** All other AWS services (ENI-based, charged per hour + data)
- Most AWS services use Interface Endpoints (SNS, RDS, CloudWatch, etc.)

### Route Table Integration
- VPC Gateway Endpoints integrate directly into route tables
- Adds implicit routes for the service (S3 routes are automatically added)
- No need to manage ENIs or security groups for gateway endpoints
- Simplifies architecture compared to interface endpoints

### Security Considerations
- VPC Endpoints provide **network isolation** (traffic stays within AWS network)
- Can attach **bucket policies** that restrict access to specific VPC endpoints
- Prevents data exfiltration through internet paths
- Complies with requirements that data never touch the public internet

---

## Real-World Production Use Cases

### Data Processing Pipeline
```
S3 Data Lake → VPC Endpoint ← EC2 Processing Cluster (Private)
↓
Results → S3 via VPC Endpoint
↓
Reporting (private access only)

Benefit: Data never leaves AWS network; no NAT charges
```

### Compliance & Data Residency
```
Regulated Data → Stored in S3 (restricted)
Private EC2 → Processes via VPC Endpoint
✗ No internet exposure
✓ Data residency maintained
✓ Audit trail available
```

### Cost Optimization
```
Before: 
- Private instances: NAT Gateway ($32/month) + Data transfer ($4.50/GB)
- 1TB/month data transfer: ~$4,500

After:
- Same setup with VPC Endpoint: $0 (free endpoint + no data charges)
- Savings: ~$4,500/month for high-volume workloads
```

---

## Security Considerations

### VPC Endpoint Security
- ✅ Traffic stays within AWS network (no internet routing)
- ✅ Can restrict bucket access to specific VPC endpoints using bucket policies
- ✅ No public IP exposure for private instances
- ⚠️ Endpoint policy defaults to full access (restrict in production)

### IAM Role Security
- ✅ Use **principle of least privilege**: Only grant necessary permissions
- ✅ Use **inline policies** for resource-specific access (e.g., single bucket)
- ⚠️ AmazonS3FullAccess or AmazonS3ReadOnlyAccess are too broad for production
- ✅ Regularly audit and rotate credentials

### S3 Bucket Security
- ✅ Block Public Access: Always enabled
- ✅ Enable versioning for accidental deletion protection
- ✅ Enable MFA Delete for critical buckets
- ✅ Enable server-side encryption (SSE-S3 or SSE-KMS)
- ✅ Enable S3 access logging to audit data access

### Network Security
- ✅ Use Security Groups to restrict EC2 outbound HTTPS to S3
- ✅ Use NACLs to add subnet-level control
- ✅ Enable VPC Flow Logs to audit VPC Endpoint traffic
- ✅ Monitor CloudTrail for API calls to S3

---

## Cost Considerations

| Component | Cost | Notes |
|-----------|------|-------|
| VPC Endpoint (Gateway) | Free | No hourly or data transfer charges |
| VPC Endpoint (Interface) | $0.007/hour | For non-S3/DynamoDB services |
| NAT Gateway (alternative) | $0.045/hour + $0.045/GB | Much more expensive |
| EC2 t2.micro | ~$0.0116/hour | Free tier eligible |
| S3 Storage | $0.023/GB/month | Standard storage pricing |
| S3 Data Transfer | Depends on route | Free via endpoint; $0.09/GB via internet |

**Estimated Cost Comparison (1 month, 1TB S3 access):**
- With NAT Gateway: $32 (gateway) + $45 (1TB transfer) = $77/month
- With VPC Endpoint: $0 (gateway + transfer) = $0/month
- **Savings with VPC Endpoint: $77/month**

---

## Resource Cleanup

```bash
# 1. Terminate EC2 instances
# EC2 Dashboard → Instances → Terminate both instances

# 2. Delete VPC Endpoint
# VPC Dashboard → Endpoints → Select endpoint → Delete

# 3. Delete S3 bucket
# S3 Dashboard → Select bucket → Delete (empty first if versioning enabled)

# 4. Delete IAM role
aws iam remove-role-from-instance-profile \
  --instance-profile-name S3ReadAccessProfile \
  --role-name S3ReadAccessRole

aws iam delete-instance-profile \
  --instance-profile-name S3ReadAccessProfile

aws iam detach-role-policy \
  --role-name S3ReadAccessRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

aws iam delete-role --role-name S3ReadAccessRole

# 5. Delete VPC resources (if not reusing)
# VPC Dashboard → Delete VPC (removes all associated subnets, IGW, route tables)
```

---

## Interview Questions

### Foundational
1. **What is the difference between IAM permissions and network connectivity?**
   - Answer: IAM permissions control "what" actions are allowed, while network connectivity controls "how" to reach the resource. Both are required for successful access.

2. **Why would you use a VPC Endpoint instead of a NAT Gateway for S3 access?**
   - Answer: VPC Endpoints are free for S3/DynamoDB (gateway type), faster, and keep traffic within AWS network. NAT Gateways cost $0.045/hour plus data transfer fees.

3. **What are the two types of VPC Endpoints?**
   - Answer: Gateway Endpoints (S3, DynamoDB) and Interface Endpoints (all other AWS services). Gateway endpoints are free and integrated into route tables; interface endpoints are charged per hour plus data.

### Intermediate
4. **Can a private instance access S3 through a NAT Gateway with an IAM role but WITHOUT a VPC Endpoint?**
   - Answer: Yes, if the private subnet's route table has a route to the NAT Gateway. However, this incurs NAT data transfer costs ($0.045/GB) and is less efficient than VPC Endpoints.

5. **How does S3 know to allow traffic from a specific VPC Endpoint?**
   - Answer: Through an S3 bucket policy that restricts access to a specific VPC Endpoint ID. The policy uses the `aws:sourceVpce` condition.

6. **What happens if you delete a VPC Endpoint while EC2 instances are still using it?**
   - Answer: Instances lose connectivity to S3. Subsequent S3 API calls will fail with a connection timeout or "unable to connect" error.

### Advanced
7. **How would you restrict S3 bucket access to only a specific VPC and deny all other access?**
   - Answer: Create a bucket policy with:
     - Allow: Principal "*", Condition: aws:sourceVpce = "vpce-xxxxx"
     - Deny: All other access (implicitly, or explicitly deny from other sources)

8. **What is the architectural difference between using a NAT Gateway and VPC Endpoints for a microservices architecture?**
   - Answer: 
     - NAT Gateway: Single chokepoint, all outbound traffic flows through one gateway, potential bottleneck, data transfer costs
     - VPC Endpoints: Each service gets a direct endpoint, scalable, only AWS service access, cost-effective for high-volume workloads

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 8, 2026)
- **Documentation:** ✅ Complete
- **Validation:** ✅ Verified (S3 access confirmed from both public and private instances via VPC Endpoint)
- **Screenshots:** 📸 Available in `./screenshots/` directory

---

## Related Tasks

- **Previous:** [Public/Private Subnets & NAT](../public-private-subnets-nat/) - Bastion pattern prerequisite
- **Related:** [VPC Peering Multi-Region](../vpc-peering-multiregion/) - Advanced networking
- **Advanced:** Security Groups & NACLs - Fine-grained access control

---

## References

- [VPC Endpoints Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
- [S3 VPC Endpoint Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/privatelink-interface-endpoints.html)
- [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [VPC Endpoint Policies](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html)
