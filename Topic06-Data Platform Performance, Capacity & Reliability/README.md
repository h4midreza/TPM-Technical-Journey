# Topic 6: Data Platform Performance, Capacity & Reliability

## 1. Objective

Understand the main performance and capacity factors affecting Data Platforms and learn how to identify technical bottlenecks, evaluate resource constraints, and assess reliability and recovery risks.

The focus is on understanding system performance, measuring processing capacity, identifying bottlenecks, and making informed decisions about scalability and reliability.

## 2. Data Volume, Velocity, and Growth Rate

Three important factors determine the workload of a Data Platform:

- **Volume:** The total amount of data generated or processed.
- **Velocity:** The speed at which data is generated and received.
- **Growth Rate:** How quickly data volume increases over time.

Example:

A network monitoring platform generates 5 TB of data per day.

| Metric | Value |
|---|---:|
| Daily Data Volume | 5 TB |
| Average Data Generation | ~58 MB/s |
| Peak Generation (3x Average) | ~174 MB/s |
| Monthly Data Volume | ~150 TB |
| Yearly Data Volume | ~1.8 PB |

These numbers are approximate and assume decimal TB.

The important distinction is that average data volume does not describe peak workload. A system may handle average traffic successfully but struggle during peak periods.

Capacity planning should consider:
- Current and expected data volume
- Peak ingestion rates
- Data retention period
- Data growth rate
- Storage overhead and redundancy
- Future processing requirements

## 3. Throughput, Latency, and Processing Time

These metrics help evaluate how efficiently a system processes data.

| Metric | Meaning | Common Unit |
|---|---|---|
| Throughput | Amount of data processed per unit of time | MB/s, GB/s, msg/s |
| Latency | Time between an event and its required response or availability | ms, seconds |
| Processing Time | Time required to execute a particular processing operation or job | seconds, minutes, hours |
| Queue Waiting Time | Time data spends waiting before processing | seconds, minutes |

### Practical Example

A Kafka pipeline receives 1,000 messages per second, while its Consumer processes 800 messages per second.

The Consumer cannot keep up with incoming data, and the backlog increases by:

`1,000 - 800 = 200 messages/second`

If this continues for 5 minutes:

`200 × 300 = 60,000 messages`

The Consumer Lag increases by approximately 60,000 messages.

Latency and processing time should also be monitored using percentiles such as P95 and P99, rather than relying only on averages.

For example, a payment processing pipeline may have an average processing time of 100 ms, but a P99 of 3 seconds. This means a small percentage of transactions take significantly longer.

## 4. IOPS, Sequential I/O, and Random I/O

Storage performance is not measured only by capacity and MB/s. The way data is read and written is also important.

### IOPS (Input/Output Operations Per Second)

IOPS measures how many read or write operations a storage device can perform per second.

Two common I/O patterns are:

**Sequential I/O**

Data is read or written in a continuous sequence.

Example:
- Reading large CSV files
- Processing historical data
- Scanning large datasets in a Data Warehouse

Sequential workloads generally benefit from high data throughput.

**Random I/O**

Data is accessed at different locations on storage.

Example:
- Database index lookups
- Reading individual customer records
- Updating specific rows in an OLTP database

Random workloads are often more sensitive to IOPS and storage latency.

| Workload | Typical I/O Pattern | Important Metric |
|---|---|---|
| Large file processing | Sequential | Throughput |
| Database index lookup | Random | IOPS and Latency |
| Data Warehouse table scan | Mostly Sequential | Throughput |
| Transactional database updates | Often Random | IOPS and Latency |

A storage system may provide high sequential throughput but still perform poorly for workloads requiring frequent random reads and writes.

## 5. CPU, Memory, Disk, and Network Constraints

A Data Platform depends on four major infrastructure resources.

| Resource | Common Bottleneck | Possible Impact |
|---|---|---|
| CPU | High utilization or insufficient processing capacity | Slow transformations and calculations |
| Memory | Memory exhaustion and frequent disk spilling | Slow queries and processing failures |
| Disk | High latency, low IOPS, or insufficient throughput | Slow reads, writes, and database operations |
| Network | Limited bandwidth, packet loss, or high latency | Slow data transfer between systems |

### Practical Bottleneck Example

Suppose a Spark job processes a large dataset.

Monitoring shows:

- CPU utilization: 35%
- Memory utilization: 90%
- Disk I/O wait: High
- Network utilization: 30%

The problem may be memory pressure causing frequent disk spilling.

Increasing CPU resources is unlikely to solve the main problem.

Another example:

- CPU utilization: 95%
- Memory utilization: 50%
- Disk latency: Low
- Network utilization: Low

Here, CPU capacity may be the primary bottleneck.

The important point is to identify the constrained resource before scaling the infrastructure.

## 6. Database Query Performance

Poor database query performance can become a bottleneck for an entire Data Platform.

Several factors affect query execution:

- Indexing
- Full Table Scans
- Join complexity
- Sorting and aggregation
- Partition pruning
- Memory availability
- Concurrent query workload

### Practical Example: Database Indexing

Consider a transactions table containing 500 million records.

A query searches for one customer's transactions:

```sql
SELECT *
FROM transactions
WHERE customer_id = 'CUST-1001';
```

Without a suitable index, the database may need to scan a large portion of the table.

An index on `customer_id` may significantly reduce the search work:

```sql
CREATE INDEX idx_customer_id
ON transactions(customer_id);
```

However, indexes also consume storage and add overhead to INSERT and UPDATE operations. They should be designed according to actual query patterns.

### Query Execution Analysis

Tools such as PostgreSQL `EXPLAIN` and `EXPLAIN ANALYZE` help identify how a query is executed.

They can reveal:

- Sequential scans
- Index scans
- Join strategies
- Estimated versus actual rows
- Execution time
- Expensive operations

A query that appears simple may be slow because it scans too much data or performs an inefficient join.

## 7. Partitioning and Parallel Processing

Partitioning divides a large dataset or workload into smaller independent parts.

It can improve performance by reducing the amount of data each operation needs to process and enabling parallel execution.

### 7.1 Database Partitioning

Consider a transactions table containing several billion records.

Instead of keeping all records in one table partition, the data can be divided by transaction date.

```text
Transactions
    |
    +-- Partition_2026_01
    +-- Partition_2026_02
    +-- Partition_2026_03
    +-- Partition_2026_04
```

If a query requests transactions from March, the database may read only the relevant partition rather than scanning all historical data.

This is called Partition Pruning.

### 7.2 Kafka Partitioning

Kafka divides a Topic into multiple Partitions to support parallel message processing.

For example:

```text
Kafka Topic: Network_Events

Partition 0 ---> Consumer 1
Partition 1 ---> Consumer 2
Partition 2 ---> Consumer 3
Partition 3 ---> Consumer 4
```

Within a Consumer Group, each partition is assigned to one consumer at a time.

Increasing the number of partitions can increase parallelism, but adding consumers beyond the number of available partitions does not increase partition-level concurrency.

The partition key also matters. A poorly chosen key can create uneven distribution and Hot Partitions.

### Important Considerations

Partitioning is not automatically beneficial.

It introduces additional considerations:
- Partition size and distribution
- Maintenance overhead
- Data skew
- Query patterns
- Ordering requirements
- Resource availability

The partition strategy should match the access and processing patterns of the system.

## 8. Bottlenecks, Consumer Lag, and Backpressure

A Bottleneck is the component or resource that limits the overall processing capacity of a system.

Consider a Data Pipeline with these stages:

```text
Data Source
    |
    v
Ingestion (NiFi)
    |
    v
Kafka
    |
    v
Processing (Spark)
    |
    v
Data Warehouse
```

Assume the measured processing capacities are:

| Component | Capacity |
|---|---:|
| Source | 150 MB/s |
| Ingestion | 140 MB/s |
| Kafka Consumers | 110 MB/s |
| Spark Processing | 90 MB/s |
| Target Database Load | 60 MB/s |

The target database has the lowest capacity.

It is therefore the primary bottleneck if these figures represent comparable sustained processing rates.

Increasing Spark resources alone will not solve the problem because the target database still cannot accept data faster than 60 MB/s.

### Kafka Consumer Lag

Consumer Lag represents the difference between the latest available position in a Kafka partition and the Consumer's committed position.

A growing lag indicates that messages are accumulating faster than the Consumer can keep up.

Possible causes:
- Insufficient Consumer processing capacity
- Slow database writes
- External service latency
- Failed or repeatedly retried messages
- Uneven partition distribution

Lag should be monitored over time. A high but decreasing lag is different from a backlog that continues to grow.

### Backpressure

Backpressure occurs when a downstream component cannot process data as quickly as it receives it.

For example:

- NiFi receives 1,000 files per minute.
- The next processing stage handles only 600 files per minute.
- The queue grows by 400 files per minute.

Backpressure mechanisms can slow or pause upstream processing to prevent queues and storage from becoming overloaded.

Backpressure helps protect the system, but it does not eliminate the underlying capacity problem.

## 9. Storage Latency and Capacity Planning

Storage planning should consider both performance and available capacity.

### Storage Latency

Storage latency measures how long a read or write operation takes.

High storage latency can affect:
- Database query execution
- Object retrieval
- Data ingestion
- Batch processing
- Recovery and rebuild operations

For example, a database may have sufficient CPU and memory but still respond slowly because its storage layer is experiencing high read/write latency.

### Capacity Planning Example

Assume a Data Lake receives 5 TB of new data every day.

For a 30-day retention period:

`5 TB × 30 = 150 TB`

This represents the approximate raw data volume before considering compression, metadata, temporary data, redundancy, and additional capacity requirements.

If the storage system uses three-way replication:

`150 TB × 3 = 450 TB`

Therefore, storage planning must distinguish between:
- Logical data size
- Raw storage capacity
- Usable storage capacity
- Reserved capacity and operational headroom

Historical growth should also be considered when estimating future storage requirements.

## 10. Replication, High Availability, and Single Points of Failure

High Availability (HA) aims to keep a system operational when individual components fail.

Replication maintains multiple copies of data across different storage locations or nodes.

For example, Kafka can replicate partitions across multiple brokers so that a broker failure does not necessarily make the partition unavailable.

However, replication alone does not guarantee High Availability. The architecture must also consider:
- Failure domains
- Leader election and failover
- Network connectivity
- Service dependencies
- Monitoring and recovery procedures

### Single Point of Failure (SPOF)

A Single Point of Failure is a component whose failure can make an entire service unavailable.

Examples:

- One Kafka broker hosting a critical partition without sufficient replication
- A single ingestion server with no failover
- One network switch connecting all storage nodes
- A single database server without a standby or recovery mechanism

Removing SPOFs requires redundancy and independent failure paths.

Replication also does not replace Backup. Accidental deletion or logical data corruption may affect replicated copies as well.

## 11. Recovery, Replay, and Reprocessing

A Data Pipeline must be able to recover from failures without losing or duplicating data.

Consider a Kafka Consumer that stops processing for 30 minutes.

During this period, messages accumulate in Kafka.

After the Consumer recovers, it must process the backlog while continuing to handle new incoming messages.

### Practical Recovery Example

Assume:

- Current Consumer Lag: 1,200,000 messages
- Incoming Rate: 600 messages/second
- Processing Capacity: 1,000 messages/second

The net backlog reduction is:

`1,000 - 600 = 400 messages/second`

Estimated recovery time:

`1,200,000 / 400 = 3,000 seconds`

That is approximately 50 minutes.

This calculation assumes stable incoming and processing rates.

### Recovery Strategies

Depending on the failure, recovery may involve:

- Restarting failed processors or Consumers
- Retrying temporary failures
- Increasing processing capacity when resources allow
- Replaying messages from Kafka
- Reprocessing historical data from Raw Storage
- Recovering from checkpoints
- Using Dead Letter Queues for permanently failed records

Reprocessing should be safe against duplicate writes. Idempotency and reliable processing-state management are important.

Kafka retention and source-data retention must also be sufficient to cover the expected recovery period.

## 12. Performance, Cost, and Reliability Trade-offs

Improving Performance or Reliability often increases infrastructure and operational costs.

| Decision | Performance Benefit | Cost / Risk Consideration |
|---|---|---|
| Add CPU resources | Faster CPU-bound processing | Higher compute cost; no benefit if another resource is the bottleneck |
| Add Memory | Less disk spilling and more caching | Higher infrastructure cost |
| Add Kafka Partitions | More potential parallelism | More operational overhead |
| Use Replication | Better fault tolerance and availability | Higher storage consumption |
| Use Erasure Coding | Lower storage overhead than multiple full replicas in suitable configurations | More computational and recovery complexity |
| Increase Retention | More time for replay and recovery | Higher storage requirements |
| Add Processing Nodes | More processing capacity | More infrastructure and coordination complexity |

The goal is not maximum performance at any cost.

A good Data Platform should meet its expected workload, recovery requirements, and availability targets while keeping resource consumption and operational complexity under control.
