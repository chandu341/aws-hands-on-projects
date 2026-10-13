# EC2 Disk Management & EBS Volumes

**Task ID:** AWS-007  
**Category:** Compute & Storage  
**Status:** ✅ Completed  
**AWS Services:** EC2, EBS, Storage, Partitions, File Systems  
**Date Completed:** October 10, 2026

---

## Objective

Understand and manage EC2 storage through EBS volume configuration, partitioning, and file system management for optimal disk utilization and performance.

---

## Why Disk Management Matters

Proper disk management is critical for:

- **Performance:** Correct file system selection and optimization
- **Reliability:** Proper backup and snapshot strategies
- **Scalability:** Adding volumes without downtime
- **Cost Optimization:** Right-sizing volumes and storage types
- **Compliance:** Encryption and data protection

---

## Architecture Overview

```
EC2 Instance
├── Root Volume (/)
│   └── OS and root filesystem
├── EBS Volume 1 (/data)
│   ├── Partition 1 (/data/app)
│   └── Partition 2 (/data/backups)
├── EBS Volume 2 (/archive)
│   └── Long-term storage
└── Instance Store (if available)
    └── Temporary high-speed storage
```

---

## Configuration Details

### EBS Volume Types
- **gp3 (General Purpose):** Balanced performance and cost (default)
- **io1/io2 (Provisioned IOPS):** High-performance databases
- **st1 (Throughput Optimized):** Big data, data warehouses
- **sc1 (Cold Storage):** Infrequently accessed data
- **standard (Magnetic):** Legacy, low cost

### Volume Management
- **Root Volume:** Operating system and applications
- **Data Volumes:** Application data, databases
- **Backup Volumes:** Snapshots and point-in-time recovery
- **Archive Volumes:** Long-term storage (cold)

---

## Implementation Steps

### Step 1: Create EBS Volumes

```bash
# Via AWS Console
# EC2 Dashboard → Volumes → Create Volume

# Volume 1 (Data):
# Type: gp3
# Size: 100 GB
# IOPS: 3000 (default)
# Throughput: 125 MB/s (default)
# Availability Zone: Same as EC2 instance
# Encryption: Enabled
# Tags: Name=data-volume

# Volume 2 (Archive):
# Type: sc1 (cold storage)
# Size: 500 GB
# Availability Zone: Same as EC2 instance
# Encryption: Enabled
# Tags: Name=archive-volume
```

### Step 2: Attach Volumes to EC2 Instance

```bash
# Via AWS Console
# Select Volume → Attach Volume
# Instance: Select your EC2 instance
# Device: /dev/sdf (for first additional volume)

# After attaching, SSH into instance:
ssh -i key.pem ec2-user@<public-ip>

# List available disks
lsblk
# Output:
# NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
# xvda    202:0    0 8G  0 disk
# └─xvda1 202:1    0 8G  0 part /
# xvdf    202:80   0 100G 0 disk
# xvdg    202:96   0 500G 0 disk
```

### Step 3: Create Partitions

```bash
# Create partition on first volume
sudo fdisk /dev/xvdf
# Commands:
# n (new partition)
# p (primary)
# 1 (partition number)
# default (first sector)
# default (last sector - use full volume)
# w (write and exit)

# Create partition on second volume
sudo fdisk /dev/xvdg
# Repeat steps above

# Verify partitions
sudo fdisk -l
```

### Step 4: Create File Systems

```bash
# Create ext4 file system (recommended for Linux)
sudo mkfs.ext4 /dev/xvdf1
# or
sudo mkfs.ext4 /dev/xvdf1 -L "data-volume"

# Create for archive volume
sudo mkfs.ext4 /dev/xvdg1 -L "archive-volume"

# Verify file systems
sudo blkid
```

### Step 5: Mount Volumes

```bash
# Create mount points
sudo mkdir -p /data
sudo mkdir -p /archive

# Mount volumes
sudo mount /dev/xvdf1 /data
sudo mount /dev/xvdg1 /archive

# Verify mounts
df -h
# Output:
# Filesystem     Size Used Avail Use% Mounted on
# /dev/xvda1     8G  2G  6G  25%  /
# /dev/xvdf1    100G  0G 100G  0%  /data
# /dev/xvdg1    500G  0G 500G  0%  /archive
```

### Step 6: Make Mounts Persistent

```bash
# Edit fstab to persist mounts after reboot
sudo nano /etc/fstab

# Add lines:
# /dev/xvdf1 /data ext4 defaults,nofail 0 2
# /dev/xvdg1 /archive ext4 defaults,nofail 0 2

# Test fstab
sudo mount -a

# Verify with df -h
```

### Step 7: Optimize File System

```bash
# Check file system for errors
sudo fsck -n /dev/xvdf1

# Resize file system if needed
sudo resize2fs /dev/xvdf1

# Check inode usage
df -i

# Monitor disk I/O
iostat -x 1

# Check file system usage by directory
sudo du -sh /data/*
```

---

## Validation Steps

### Verify Volume Attachment
```bash
# List volumes
aws ec2 describe-volumes \
  --filters "Name=attachment.instance-id,Values=i-xxxxx"

# Check volume status
aws ec2 describe-volumes --volume-ids vol-xxxxx
```

### Verify Mount Points
```bash
# Check all mounts
mount | grep /data
mount | grep /archive

# Verify with df
df -h /data
df -h /archive

# Check for errors in logs
sudo dmesg | grep xvd
```

### Test Disk Performance
```bash
# Write performance test
sudo dd if=/dev/zero of=/data/testfile bs=1M count=1000

# Read performance test
sudo dd if=/data/testfile of=/dev/null bs=1M

# Use fio for advanced testing
sudo fio --name=randread --ioengine=libaio --iodepth=16 \
  --rw=randread --bs=4k --direct=1 --size=1G \
  --filename=/data/testfile --numjobs=4 --runtime=60
```

---

## Commands & Configuration

### Common Disk Management Commands

```bash
# List all block devices
lsblk

# Show detailed disk information
blkid

# Disk usage
df -h          # Human-readable
du -sh *       # Directory sizes
df -i          # Inode usage

# File system check
fsck /dev/xvdf1
e2fsck /dev/xvdf1

# Resize file system (when volume is resized)
resize2fs /dev/xvdf1

# Monitor I/O
iostat -x 1
iotop

# Performance testing
dd if=/dev/zero of=/tmp/test bs=1M count=1024
time dd if=/tmp/test of=/dev/null bs=1M
```

### EBS Snapshot Management

```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxx \
  --description "Daily backup"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Create volume from snapshot
aws ec2 create-volume \
  --snapshot-id snap-xxxxx \
  --availability-zone us-east-1a

# Delete snapshot
aws ec2 delete-snapshot --snapshot-id snap-xxxxx
```

---

## What I Learned

### EBS Fundamentals
1. **Volume Types:** Different types for different workloads
2. **Performance:** IOPS and throughput matter for applications
3. **Attachment:** Volumes attach as block devices (/dev/xvd*)
4. **Persistence:** Volumes persist even when instance stops

### Partitioning & File Systems
1. **Partitions:** Logical division of physical volume
2. **File Systems:** ext4 is standard for Linux/EBS
3. **Mount Points:** Where file system becomes accessible
4. **Fstab:** Persists mounts across reboots

### Performance Optimization
1. **Volume Type Selection:** gp3 for most workloads
2. **IOPS Provisioning:** Match application requirements
3. **Throughput:** Important for sequential I/O
4. **Monitoring:** Track disk usage and I/O patterns

### Capacity Management
1. **Resizing:** Volumes can be resized online (no downtime)
2. **Snapshots:** Point-in-time backups
3. **Cleanup:** Regular removal of unused volumes/snapshots
4. **Forecasting:** Plan for growth

---

## Real-World Production Use Cases

### High-Performance Database
```
EBS Configuration:
- Root: gp3, 50GB (OS and application)
- Data: io2 with 10,000 IOPS (database files)
- Logs: gp3 with 1000 IOPS (transaction logs)
- Backups: sc1 (cold snapshots)

Result: Optimized for database performance
```

### Big Data Processing
```
EBS Configuration:
- Root: gp3, 100GB
- Data: st1 (throughput optimized, 500GB+)
- Cache: gp3 with high IOPS
- Archive: sc1 (completed job results)

Result: Optimized for sequential access patterns
```

### Multi-Tier Application
```
Frontend (web servers):
- gp3 volumes (balanced performance)

Application (middle tier):
- gp3 volumes with moderate IOPS

Database (backend):
- io2 volumes with high IOPS and Multi-AZ

Backups:
- sc1 volumes (archived snapshots)

Result: Right-sized for each tier's requirements
```

---

## Security & Compliance

### Data Protection
- ✅ Enable EBS encryption (AES-256)
- ✅ Regular snapshots for disaster recovery
- ✅ Encrypt snapshots when copying to other regions
- ✅ Monitor volume metrics

### Access Control
- ✅ Use IAM policies to restrict volume access
- ✅ Audit volume modifications with CloudTrail
- ✅ Use MFA delete for critical snapshots

### Compliance
- ✅ HIPAA: Ensure encryption for healthcare data
- ✅ PCI DSS: Encrypt payment card data
- ✅ GDPR: Audit access and maintain backups

---

## Cost Optimization

| Volume Type | Cost/Month (100GB) | Use Case |
|-------------|-------|----------|
| gp3 | ~$5 | General purpose (default) |
| io1 | ~$15 | High IOPS databases |
| st1 | ~$4 | Throughput optimized |
| sc1 | ~$2 | Cold storage, archives |
| standard | ~$2 | Legacy, low performance |

**Optimization Tips:**
- Use gp3 by default (better than gp2)
- Right-size IOPS (don't over-provision)
- Archive old snapshots to S3 for long-term storage
- Delete unused volumes and snapshots

---

## Interview Questions

### Foundational
1. **What is EBS?**
   - Block storage for EC2; persistent, independent of instance lifecycle
2. **How is EBS different from instance store?**
   - EBS: Persistent, slower, survive instance stop. Instance store: Temporary, fast, lost on stop
3. **What file system should you use for Linux?**
   - ext4 (standard), or XFS for high-performance workloads

### Intermediate
4. **How do you resize an EBS volume without downtime?**
   - Increase volume size in AWS console, then resize file system with `resize2fs`
5. **What's the relationship between volumes and snapshots?**
   - Snapshots are point-in-time backups; new volumes can be created from snapshots
6. **Why would you use sc1 instead of gp3?**
   - sc1 is much cheaper for archived/cold data; gp3 for active workloads

### Advanced
7. **How would you design storage for a MySQL database with 10TB of data and 50,000 IOPS?**
   - io2 volumes with provisioned IOPS; Multi-AZ for HA; Regular snapshots; Separate logs volume

---

## Implementation & Documentation Status

- **Implementation:** ✅ Completed (October 10, 2026)
- **Documentation:** ✅ Complete
- **Screenshots:** 📸 13 images extracted and organized
- **Validation:** ✅ Volume attachment, partitioning, and file system creation tested

---

## References

- [EBS Documentation](https://docs.aws.amazon.com/ebs/)
- [EBS Volume Types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [EBS Best Practices](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-best-practices.html)
- [Snapshots Documentation](https://docs.aws.amazon.com/ebs/latest/userguide/EBSSnapshots.html)
