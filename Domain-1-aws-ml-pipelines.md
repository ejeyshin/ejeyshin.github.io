# AWS ML Specialty: Cost-Effective Scalable ML Pipelines

**Domain:** Domain 1 - Data Engineering (20%)  
**Date:** 01/10/ 2025

---

## Streaming Ingestion Services

### Decision Tree
Multiple consumers with different processing?
├─ YES → Kafka ecosystem required?
│  ├─ YES → Amazon MSK
│  └─ NO → Kinesis Data Streams
└─ NO → Kinesis Data Firehose

### Service Comparison

| Feature | Firehose | Data Streams | MSK |
|---------|----------|--------------|-----|
| Scaling | Auto | Manual shards | Manual cluster |
| Management | Zero | Medium | High |
| Cost | $0.029/GB | $0.015/shard-hr | $0.21+/broker-hr |
| Latency | 60-900s | <1s | <1s |

### When to Use Each

**Kinesis Data Firehose**
- Single destination (S3/Redshift/Elasticsearch)
- Zero management, auto-scales
- Cost-effective for simple delivery

**Kinesis Data Streams**
- Multiple consumers need same data
- Each processes differently
- Need replay capability
- Must manage shards manually

**Amazon MSK**
- Kafka ecosystem needed (Connect, Streams)
- Exactly-once semantics required
- High operational complexity
- Expensive ($760+/month minimum)

---

## Kinesis Data Streams: Shard Management

### What is a Shard?

Unit of throughput capacity:
- Write: 1 MB/sec OR 1,000 records/sec
- Read: 2 MB/sec per consumer

### Calculate Shards Needed
Example: 8,000 events/sec × 750 bytes = 6 MB/sec
Shards needed: 6 MB/sec ÷ 1 MB/sec = 6 shards


### SageMaker Managed Spot Training
Why 70-90% Cheaper

AWS sells unused EC2 capacity at massive discount
Trade-off: Can be interrupted anytime

SageMaker automatically:

Requests Spot capacity
Saves checkpoints to S3 every few minutes
Restores from checkpoint if interrupted
Retries until max_wait reached

When to Use
### Use Spot:

Daily/weekly batch training
Flexible deadlines (can wait 2-4x normal time)
Cost-sensitive projects
Non-critical workloads

### Use On-Demand:

Hard deadline <4 hours away
Production emergencies
Very short jobs (<30 min)
Critical compliance requirements

### Key Takeaways

Firehose = Single destination, zero management
Data Streams = Multiple consumers, shard management
MSK = Only for Kafka ecosystem (expensive, complex)
Spot Training = Batch jobs with flexible deadlines (70-90% savings)
On-Demand = Urgent/critical deadlines
DynamoDB = Simple lookups, auto-scales, best for ML serving
DataBrew ≠ Training (only data prep)
Buffer ratio = Deadline ÷ Training Time (>4x = safe for Spot)


### Practice Questions

Question 1: Multi-Service Architecture
Scenario: Healthcare analytics processes patient data to predict readmission risk.

Ingest 2 GB/hour from 500 hospitals
Train weekly (6-hour job)
Serve predictions via API (<50ms)
3-day window before weekly reports
Budget-conscious startup

Which is most cost-effective?
A) MSK → Kafka Streams → S3 → SageMaker On-Demand → Aurora
B) Data Streams → Lambda → S3 → Spot Training → DynamoDB
C) Firehose → S3 → Spot Training → DynamoDB
D) Direct S3 → Glue ETL → On-Demand → ElastiCache
<details>
<summary>Answer</summary>
C) Firehose → S3 → Spot Training → DynamoDB
Why:

Firehose - Single destination (S3), auto-scales, cheapest ($60/mo vs $120/mo)
Spot Training - Weekly with 3-day window = flexible, save $480/month
DynamoDB - <50ms met (<10ms typical), auto-scales, simple lookups

Cost: $190/month vs $1,490/month (Answer A)
Key: "Weekly" + "3-day window" + "budget" = Spot Training
</details>

Question 2: Scaling Architecture
Scenario: Social media analytics has:

Firehose → S3 (working)
Daily SageMaker On-Demand training
DynamoDB serving

New requirements:

Marketing wants real-time sentiment
Data Science wants to experiment with different algorithms on same stream

Best change?
A) Add another Firehose
B) Replace with Data Streams → Multiple Lambdas
C) Replace with MSK
D) Add SageMaker Processing
<details>
<summary>Answer</summary>
B) Replace with Data Streams → Multiple Lambdas
Why:

Now have 3 consumers: sentiment, experiments, S3
Each needs different processing
Data Streams supports multiple independent consumers
Still cheaper than MSK ($200/mo vs $760/mo)

Pattern: Single → Multiple consumers = Firehose → Data Streams
</details>

Question 3: Spot Training Risk
Scenario: Fraud detection retrains every 6 hours.

Training: 45 minutes
Must complete within 2 hours
New patterns emerge quickly

Should you use Spot?
A) Yes, save 70-90% with minimal risk
B) No, 2-hour deadline too tight
C) Yes with max_wait=4 hours
D) Hybrid: Spot then On-Demand
<details>
<summary>Answer</summary>
B) No, 2-hour deadline too tight
Why:
Deadline: 120 min
Training: 45 min
Buffer: 120 ÷ 45 = 2.67x

Risk: Two interruptions could miss deadline
Business impact: Undetected fraud > cost savings
Rule: Buffer ratio <3x + critical business = On-Demand
When Spot works: If deadline was 12 hours (16x buffer), Spot is perfect
</details>


