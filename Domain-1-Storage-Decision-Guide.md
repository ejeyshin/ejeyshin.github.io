---
layout: default
title: "AWS ML Storage Decision Guide"
description: "Study guide for AWS Machine Learning Specialty certification"
---

---

## IOPS vs Throughput

### Understanding the Difference

**IOPS (Operations per second):**
- Counts the NUMBER of operations
- 1 file open = 1 operation
- 1 file close = 1 operation
- Limited to 7,000 per second in General Purpose mode

**Throughput (MB/s or GB/s):**
- Measures the AMOUNT of data transferred
- Reading 100 MB of data in one operation
- Scales with storage size

> **Why this matters:** You can hit the IOPS limit (7,000 operations/sec) while still having plenty of throughput capacity available!

---

## Storage Options Comparison

### Quick Comparison Chart

Understanding when to use each storage type for ML workloads:

| Feature            | EFS                        | EBS                        | S3                         | FSx for Lustre             |
|--------------------|----------------------------|----------------------------|----------------------------|----------------------------|
| **Access**         | Multiple EC2 instances     | Single EC2 instance        | Any service, unlimited     | Multiple instances (HPC)   |
| **Use case**       | Shared training data       | Boot volumes, databases    | Data lakes, archives       | High-performance ML        |
| **Performance**    | Scales with size           | Fixed provisioned IOPS     | High throughput            | Sub-millisecond latency    |
| **Cost**           | $$ (per GB stored)         | $ (per GB provisioned)     | $ Cheapest                 | $$$ (high performance)     |
| **For ML**         | Shared datasets            | Single-instance training   | Data storage/versioning    | Distributed training       |
| **Max throughput** | ~10-20 GB/s                | ~4 GB/s per volume         | Hundreds GB/s              | 1+ TB/s                    |

---

## Provisioned Throughput

### EFS Specific Feature

**What is it?**
- Optional feature to provision throughput independent of storage size
- Useful when you have small amounts of data but need high performance
- **Costs extra money**

**Key difference from EBS:**
- EBS has "Provisioned IOPS" for block storage
- EFS has "Provisioned Throughput" for file storage
- They are NOT the same thing!

**When to use Provisioned Throughput:**
- Small dataset (< 1 TB) that needs more than 50 MB/s
- Predictable high-performance requirements
- Can't or don't want to add dummy data to increase storage size

---

## Vocabulary Reference

### Performance Terms

| Term                       | Definition                                            | Example                        |
|----------------------------|-------------------------------------------------------|--------------------------------|
| **IOPS**                   | Input/Output Operations Per Second - counts ops       | 7,000 file opens per second    |
| **Throughput**             | Amount of data transferred per second                 | 100 MB/s read speed            |
| **Latency**                | Time delay for a single operation                     | 3ms per file open              |
| **Baseline performance**   | Guaranteed continuous performance level               | 50 MB/s for 1 TB EFS           |
| **Burst performance**      | Temporary higher performance                          | 100 MB/s burst for 1 TB EFS    |
| **Burst credits**          | Allow temporary performance above baseline            | Accumulate when under baseline |

### File System Terms

| Term                       | Definition                                                     |
|----------------------------|----------------------------------------------------------------|
| **NFS**                    | Network File System protocol                                   |
| **Metadata operations**    | Operations on file information (list, stat, open, close)       |
| **File operations**        | Any action performed on a file                                 |
| **Shared file system**     | Accessible by multiple clients simultaneously                  |

### AWS EFS Specific

| Term                       | Definition                                                     |
|----------------------------|----------------------------------------------------------------|
| **General Purpose mode**   | Low latency, 7,000 ops/sec limit                               |
| **Max I/O mode**           | Higher aggregate throughput, higher latency                    |
| **Provisioned throughput** | Pay extra for guaranteed MB/s independent of size              |
| **Elastic**                | Automatically grows/shrinks                                    |

### Storage Comparison

| Term                       | Description                                                    |
|----------------------------|----------------------------------------------------------------|
| **EFS**                    | Shared network file system (NFS)                               |
| **EBS**                    | Block storage attached to single EC2 instance                  |
| **S3**                     | Object storage, unlimited scale                                |
| **FSx for Lustre**         | High-performance file system for ML/HPC                        |
| **Instance Store**         | Temporary local NVMe storage                                   |

---

## Key Takeaways

### 🎯 The Most Important Concepts

1. **IOPS = operations count**, not data volume
2. **Many small files = many operations** = high IOPS usage
3. **EFS General Purpose mode has a 7,000 operations/second limit**
4. **Consolidating files reduces operations** while keeping same data
5. **Performance mode cannot be changed** after file system creation
6. **EFS performance scales with storage size** (50 MB/s per TB baseline)
