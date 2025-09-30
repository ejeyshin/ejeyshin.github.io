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




# Practice Questions

## Question 1: EFS IOPS Management

A data science team is using Amazon EFS in General Purpose mode to store training datasets. CloudWatch metrics show that IOPS usage is consistently at 100%, causing slow data access. The file system contains 200 GB of data across 15,000 small CSV files. What is the MOST effective solution to improve performance?

**A)** Increase the provisioned throughput of the EFS file system  
**B)** Convert the EFS file system from General Purpose to Max I/O performance mode  
**C)** Reconfigure EC2 instances to use Amazon EBS volumes instead of EFS  
**D)** Consolidate the CSV files into fewer, larger files

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: D) Consolidate the CSV files into fewer, larger files**

**Explanation:**
- EFS General Purpose mode has a limit of 7,000 file operations per second
- Each file open/close counts as an operation
- 15,000 small files generate many metadata operations, consuming IOPS
- Consolidating to fewer files (e.g., 100 larger files) reduces total operations while maintaining the same data volume

**Why not the others:**
- **A)** Provisioned throughput increases MB/s (data transfer rate), not IOPS (operations per second). This wouldn't address the 7,000 operations/second limit.
- **B)** Performance mode CANNOT be changed after file system creation. You would need to create a new file system and migrate data.
- **C)** While EBS might work for single-instance scenarios, this doesn't answer how to manage EFS performance. EFS is likely being used because multiple instances need shared access.

**Key Concept:** IOPS measures operations (open, close, list), not data volume. Many small files = many operations = IOPS bottleneck.

</details>

---

## Question 2: EFS vs EBS for ML Training

A machine learning team needs to train a deep learning model on a dataset that is 800 GB in size. They plan to use 5 EC2 instances simultaneously for distributed training. Which storage solution is MOST appropriate?

**A)** Amazon EBS volumes attached to each EC2 instance with data replicated across all volumes  
**B)** Amazon EFS file system mounted to all 5 EC2 instances  
**C)** Amazon S3 with data downloaded to local instance storage before training  
**D)** Amazon EBS Multi-Attach volumes shared across all instances

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: B) Amazon EFS file system mounted to all 5 EC2 instances**

**Explanation:**
- EFS is designed for shared access across multiple EC2 instances
- All 5 instances can read the same dataset simultaneously without data duplication
- Eliminates the need to copy/sync data across instances
- Ideal for distributed training scenarios

**Why not the others:**
- **A)** Requires manually replicating 800 GB to each instance (4 TB total storage), increases complexity and cost, and requires sync mechanisms for data consistency
- **C)** S3 doesn't support file system operations. Downloading to local storage adds latency and requires sufficient instance storage (expensive)
- **D)** EBS Multi-Attach only works with specific instance types in the same Availability Zone, and is limited to 16 instances maximum. It's also more complex and expensive than EFS for this use case.

**Key Concept:** EFS = shared file system, EBS = single-instance block storage. For distributed training with multiple instances, EFS is the natural choice.

</details>

---

## Question 3: Storage Selection for ML Pipeline

An ML engineer is designing a data pipeline with the following requirements:
- Ingest raw data from various sources (streaming and batch)
- Store raw data for long-term archival and compliance (5+ years)
- Perform ETL transformations using AWS Glue
- Provide processed data to SageMaker for training

Which storage strategy is MOST cost-effective?

**A)** Store all data in Amazon EFS with lifecycle policies  
**B)** Store raw data in S3 Standard, use S3 Intelligent-Tiering for archival, process with Glue, store training data in S3  
**C)** Store raw data in Amazon EBS volumes, archive to S3 Glacier, process with EMR  
**D)** Store everything in Amazon Redshift for easy querying

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: B) Store raw data in S3 Standard, use S3 Intelligent-Tiering for archival, process with Glue, store training data in S3**

**Explanation:**
- S3 is the most cost-effective for data lakes and long-term storage
- AWS Glue natively integrates with S3 for ETL operations
- S3 Intelligent-Tiering automatically moves data to cheaper storage tiers based on access patterns
- SageMaker can directly read training data from S3
- No need to provision storage capacity (fully elastic)

**Why not the others:**
- **A)** EFS is expensive for long-term archival storage (charged per GB-month). Not designed for archival workloads.
- **C)** EBS volumes must be provisioned and are expensive for large-scale archival. Requires manual management. EMR can read from S3 directly.
- **D)** Redshift is a data warehouse optimized for queries, not for storing raw ML training data. Much more expensive than S3 for storage.

**Key Concept:** For ML data lakes, S3 is the foundation. It's cheapest, scalable, integrates with all AWS ML services, and has built-in lifecycle policies.

</details>

---

## Question 4: EFS Performance Mode Selection

A team is setting up a new Amazon EFS file system to support a computer vision project that will have 200 EC2 instances simultaneously reading training images. The workload prioritizes maximum aggregate throughput over latency. Which configuration should they choose?

**A)** General Purpose performance mode with Bursting throughput  
**B)** General Purpose performance mode with Provisioned throughput  
**C)** Max I/O performance mode with Bursting throughput  
**D)** Max I/O performance mode with Elastic throughput

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: C) Max I/O performance mode with Bursting throughput**

**Explanation:**
- Max I/O mode is designed for highly parallelized workloads with many clients (200 instances)
- Max I/O provides higher aggregate throughput when accessed by hundreds of instances
- The question states "prioritizes maximum aggregate throughput over latency"
- Bursting throughput is included by default and scales with storage size

**Why not the others:**
- **A)** General Purpose mode has a 7,000 operations per second limit, which could be a bottleneck with 200 instances
- **B)** Provisioned throughput costs extra. Start with Bursting mode first, only provision if needed
- **D)** "Elastic throughput" is not a real EFS throughput mode (as of the exam). The modes are Bursting and Provisioned.

**Key Concept:** 
- **General Purpose**: Lower latency, up to 7,000 ops/sec, suitable for most workloads
- **Max I/O**: Higher aggregate throughput, slightly higher latency, for 100s-1000s of clients

**Important:** Performance mode CANNOT be changed after creation, so choose carefully!

</details>

---

## Question 5: Troubleshooting EFS Performance

A data scientist reports that their training job reads data slowly from an EFS file system. CloudWatch metrics show:
- PercentIOLimit: 15%
- TotalIOBytes: 45 MB/s (consistent)
- StoredBytes: 100 GB

The file system is in General Purpose mode with Bursting throughput. What is the MOST likely cause of slow performance?

**A)** The IOPS limit has been reached  
**B)** The file system is too small, providing insufficient baseline throughput  
**C)** Network bandwidth between EC2 and EFS is the bottleneck  
**D)** The file system should be using Provisioned throughput mode

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: B) The file system is too small, providing insufficient baseline throughput**

**Explanation:**
- EFS baseline throughput = 50 MB/s per TB of storage
- 100 GB = 0.1 TB → baseline = 0.1 × 50 = 5 MB/s
- Observed throughput is 45 MB/s, meaning it's using burst credits
- Burst credits will eventually deplete, causing performance to drop to 5 MB/s baseline
- This is the "small file system" performance problem

**Why not the others:**
- **A)** PercentIOLimit is only 15%, not near the 7,000 ops/sec limit. IOPS is not the issue.
- **C)** While possible, the CloudWatch metrics suggest throughput limitation, not network issues. EC2 instances typically have sufficient bandwidth for 45 MB/s.
- **D)** Provisioned throughput would help, but it costs extra money. The root cause is insufficient storage for baseline performance.

**Solution:** Either add more data to reach 1 TB (50 MB/s baseline), or enable Provisioned Throughput to guarantee performance independent of storage size.

**Key Concept:** EFS performance scales with storage size. For small datasets needing high performance, Provisioned Throughput is necessary (at extra cost).

</details>

---

## Question 6: File Operations vs Data Transfer

An ML engineer needs to process 1 million small log files (average 50 KB each) stored in EFS. Processing requires reading each file sequentially. Which statement about performance is TRUE?

**A)** Throughput (MB/s) will be the primary bottleneck  
**B)** IOPS (operations per second) will be the primary bottleneck  
**C)** Both throughput and IOPS will be equally bottlenecked  
**D)** Network latency will be the primary bottleneck

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: B) IOPS (operations per second) will be the primary bottleneck**

**Explanation:**
- 1 million files require at least 1 million file open operations
- EFS General Purpose mode limits: 7,000 file operations per second
- Time to open all files: 1,000,000 ÷ 7,000 = ~143 seconds just for open operations
- Total data: 1M files × 50 KB = 50 GB
- Even at low throughput (50 MB/s), reading 50 GB takes only 1,000 seconds
- The IOPS limit (7,000 ops/sec) will be hit long before throughput becomes an issue

**Why not the others:**
- **A)** Total data is only 50 GB. Even modest throughput can handle this. The problem is the NUMBER of operations, not data volume.
- **C)** IOPS will saturate first, not equally with throughput
- **D)** Network latency affects each operation, but the fundamental limit is EFS's 7,000 operations per second cap

**Solution:** Consolidate log files into larger files (e.g., combine 1,000 logs per file → 1,000 total files). This reduces operations from 1 million to 1,000.

**Key Concept:** 
- **Many small files = IOPS bottleneck** (operation count limit)
- **Few large files = Throughput bottleneck** (data transfer limit)

</details>

---

## Question 7: Cost Optimization for ML Storage

A team stores 50 TB of training data in Amazon S3. They actively use 5 TB for current model training, while the remaining 45 TB is accessed only occasionally for retraining legacy models (once every 3-6 months). What is the MOST cost-effective storage strategy?

**A)** Store all data in S3 Standard for consistent access performance  
**B)** Store all data in S3 Glacier Instant Retrieval for lowest cost  
**C)** Use S3 Intelligent-Tiering for all data to automatically optimize costs  
**D)** Store 5 TB in S3 Standard and 45 TB in S3 Glacier Deep Archive

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: C) Use S3 Intelligent-Tiering for all data to automatically optimize costs**

**Explanation:**
- S3 Intelligent-Tiering automatically moves objects between access tiers based on usage patterns
- Frequently accessed data stays in Frequent Access tier (same cost as S3 Standard)
- Infrequently accessed data automatically moves to Infrequent Access tier (saves ~50% cost)
- Data not accessed for 90+ days moves to Archive tiers (saves ~95% cost)
- No retrieval fees when access patterns change
- Monitoring fee is minimal: $0.0025 per 1,000 objects

**Why not the others:**
- **A)** Wastes money on 45 TB that's rarely accessed. S3 Standard is most expensive tier.
- **B)** Glacier Instant Retrieval charges per-GB retrieval fees. If usage patterns are unpredictable, costs could spike.
- **D)** Glacier Deep Archive requires 12-hour retrieval time. "Once every 3-6 months" suggests you might need faster access. Also requires manual management of which data goes where.

**Key Concept:** S3 Intelligent-Tiering is ideal when access patterns are unpredictable or change over time. It automatically optimizes without manual intervention.

**Cost Comparison (per GB-month):**
- S3 Standard: $0.023
- S3 Intelligent-Tiering (Frequent): $0.023
- S3 Intelligent-Tiering (Infrequent): $0.0125
- S3 Glacier Instant Retrieval: $0.004 (but has retrieval fees)
- S3 Glacier Deep Archive: $0.00099 (12-hour retrieval)

</details>

---

## Question 8: Distributed Training Storage Architecture

A deep learning team is training a large language model using 20 P3.8xlarge GPU instances. Training data is 5 TB and must be accessed randomly by all instances. Training runs continuously for 48 hours. Which storage architecture provides the BEST performance?

**A)** Amazon S3 with data loaded to local NVMe SSDs at job start  
**B)** Amazon EFS in Max I/O mode with Provisioned Throughput  
**C)** Amazon FSx for Lustre linked to S3 bucket  
**D)** Amazon EBS volumes with data replicated to each instance

<details>
<summary>✅ Click to reveal answer</summary>

**Answer: C) Amazon FSx for Lustre linked to S3 bucket**

**Explanation:**
- FSx for Lustre is purpose-built for high-performance computing (HPC) and ML training
- Provides sub-millisecond latencies and throughput up to hundreds of GB/s
- Can be linked to S3 for easy data import/export (lazy loading)
- Supports concurrent access from multiple instances with POSIX file system semantics
- Optimized for GPU workloads requiring high throughput

**Performance Comparison:**
- FSx for Lustre: Up to 1+ TB/s aggregate throughput
- EFS Max I/O: Up to tens of GB/s
- Local NVMe: Fastest but requires data loading time and limits flexibility

**Why not the others:**
- **A)** P3.8xlarge instances have 2×900 GB NVMe drives (1.8 TB). Cannot fit 5 TB of data. Loading 5 TB would take significant time. Good for smaller datasets.
- **B)** EFS Max I/O can work but FSx for Lustre provides 10x better performance for ML workloads at similar cost. EFS tops out around 10-20 GB/s aggregate.
- **D)** Requires 20 × 5 TB = 100 TB of provisioned storage. Expensive and complex to sync. EBS Multi-Attach has limitations.

**Key Concept:** FSx for Lustre is AWS's recommended solution for high-performance ML training with large datasets and multiple instances.

**When to use each:**
- **FSx for Lustre**: Distributed training, HPC, need maximum performance
- **EFS**: General-purpose shared file system, simpler workloads
- **Local NVMe**: Single instance, small datasets, need absolute lowest latency
- **S3**: Data lake, not for direct training (too high latency)

</details>

---

## Study Tips for Domain 1 Questions

### 1. **Know the Limits**
Every AWS service has specific limits. For EFS General Purpose:
- ✅ 7,000 file operations per second
- ✅ Throughput scales with storage size
- ✅ Cannot change performance mode after creation

### 2. **Understand "Operations" vs "Data"**
- Operations = counting actions (IOPS)
- Data = measuring bytes transferred (throughput)
- You can max out operations while having plenty of data capacity!

### 3. **Look for "Cannot be changed" patterns**
AWS often has settings that are permanent:
- EFS performance mode ❌
- RDS engine type ❌
- S3 region ❌
These make certain answer options automatically wrong!

### 4. **Think about the "Why" behind architecture**
- Why use EFS? → Shared access across multiple instances
- Why use EBS? → Dedicated performance for one instance
- Why use S3? → Scalable storage, cheapest option
- Why use FSx for Lustre? → Maximum performance for HPC/ML

### 5. **Real-world ML scenarios**
For machine learning on AWS:
- **Training data**: Often stored in S3, loaded to EFS for shared access
- **Model files**: S3 for versioning and distribution
- **Active training**: EBS for single-instance, EFS for distributed training
- **Large datasets**: Consider FSx for Lustre (high-performance file system)

---

## Additional Resources

### AWS Documentation
- [Amazon EFS User Guide](https://docs.aws.amazon.com/efs/latest/ug/)
- [EFS Performance](https://docs.aws.amazon.com/efs/latest/ug/performance.html)
- [Storage Options for ML](https://docs.aws.amazon.com/whitepapers/latest/practicing-continuous-integration-continuous-delivery/storage-options.html)

### Practice More
- [AWS Certified Machine Learning Specialty Exam Guide](https://aws.amazon.com/certification/certified-machine-learning-specialty/)
- [AWS Skill Builder](https://skillbuilder.aws/)

