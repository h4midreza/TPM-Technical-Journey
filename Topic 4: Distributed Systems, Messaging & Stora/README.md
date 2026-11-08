# Topic 4: Distributed Systems, Messaging & Storage

## 1. Objective

The objective of this topic is to understand distributed systems, messaging platforms, distributed data processing, and storage technologies used in modern Data Platforms.

The focus is on understanding system architecture, component responsibilities, dependencies, failure handling, and technical limitations from a Technical Program Manager (TPM) perspective.

## 2. Distributed Systems Fundamentals

### Definition

A Distributed System consists of multiple independent computers or services that communicate over a network and work together to achieve a common goal.

In this architecture, each service has a specific responsibility and can be developed, deployed, or scaled independently.

### Key Benefits

- Scalability: Increasing system capacity by adding resources or services.
- Fault Tolerance: Maintaining system functionality despite failures in some components, within the limits of the system design.
- High Availability: Improving service availability and reducing downtime.
- Performance: Enabling parallel processing and workload distribution.

### Key Challenges

- Network Failure: Communication failures between services.
- Latency: Delays in data transmission or processing.
- Partial Failure: Some components fail while other parts of the system remain operational.
- Data Consistency: Maintaining consistent data across multiple services.
- Monitoring: Tracking the health and performance of distributed components.

### Important Concepts

- Timeout: The maximum time a system waits for a response.
- Retry: Attempting an operation again after a failure.
- Idempotency: Ensuring that repeated execution of an operation does not create unintended duplicate effects.
- Redundancy: Providing alternative resources or components.
- Fault Tolerance: The ability to continue operating despite certain failures.
- High Availability: Designing systems to minimize service downtime.

**Note:** A timeout does not necessarily mean that an operation failed. The operation may have completed successfully, but its response did not reach the sender. Retrying without Idempotency can therefore result in duplicate processing.

## 3. Message Brokers and Event Streaming

### What is a Message Broker?

A Message Broker is an intermediary responsible for receiving, storing, and delivering messages between services.

It allows a service that produces messages (Producer) to communicate with multiple consuming services (Consumers) without requiring direct connections to each of them.

This architecture reduces direct dependencies between services and enables asynchronous processing.

### Message Queue vs. Event Streaming

- Message Queues are commonly used to distribute tasks among consumers.
- Event Streaming platforms are designed to capture, store, and distribute streams of events, allowing multiple systems to consume them independently.

In Event Streaming systems, events can be retained for a defined period and replayed when needed.

### Common Tools

| Tool | Primary Use Case | Example |
|---|---|---|
| Apache Kafka | Event Streaming and event retention | Distributing document events to Analytics, Audit, and Notification services |
| RabbitMQ | Message Queue and task distribution | Distributing document processing tasks among workers |
| Apache Pulsar | Messaging and Event Streaming | Large-scale messaging and streaming workloads |

Tool selection depends on message volume, consumption patterns, replay requirements, ordering, retention policies, and operational complexity.

## 4. Apache Kafka

### Overview

Apache Kafka is a Distributed Event Streaming Platform designed to receive, store, and distribute event streams at scale.

### Core Components

| Concept | Description |
|---|---|
| Cluster | A group of Kafka Brokers |
| Broker | A server responsible for storing and managing messages |
| Topic | A logical category for organizing messages |
| Partition | An independent segment of a Topic that enables parallel processing |
| Producer | A service that publishes messages |
| Consumer | A service that receives and processes messages |
| Consumer Group | A group of Consumers that share the workload of a Topic |
| Offset | The position of a message within a Partition |
| Retention | The time- or size-based policy for retaining messages |

### Topics and Partitions

For example, a Topic named `document-events` can store events related to document operations.

If a Topic has four Partitions, messages are distributed across four independent logs. This enables parallel processing.

Message ordering is guaranteed only within an individual Partition, not across the entire Topic.

To preserve the ordering of events related to a specific document, `document_id` can be used as the Message Key. This directs events for the same document to the same Partition.

### Consumer Groups

If a Topic has four Partitions, a maximum of four Consumers in a single Consumer Group can actively consume those Partitions simultaneously.

If six Consumers are assigned to the same Group, at least two may remain idle, assuming all four Partitions are assigned.

Independent Consumer Groups can consume the same Topic separately.

For example:

- Analytics Group: Processes document information for analytical purposes.
- Audit Group: Records document event history.
- Notification Group: Sends notifications.

Consumption by one Group does not remove messages for other Groups.

### Offset and Commit

An Offset represents the position of a Consumer within a Partition.

A common processing sequence is:

1. The Consumer receives a message.
2. It performs the required operation in the destination system.
3. It commits the Offset after successful processing.

If the database write succeeds but the Consumer stops before committing the Offset, Kafka may deliver the same message again, potentially causing duplicate records.

A key solution is Idempotency, using a unique identifier such as `event_id` to prevent duplicate effects.

### Retention and Replay

Kafka does not immediately delete messages after they are consumed. Messages are retained according to configured Retention policies.

Replay means reading previously published messages again by adjusting Consumer Offsets, provided those messages are still available in Kafka.

Replay is useful for recovery and reprocessing, but its impact on downstream systems and the risk of duplicate processing must be considered.

### Delivery Semantics

- At-most-once: A message is processed no more than once; message loss is possible.
- At-least-once: A message is delivered at least once; duplicate processing is possible.
- Exactly-once: Can be implemented within specific Kafka capabilities and carefully designed processing boundaries. It does not automatically guarantee exactly-once processing across all external systems, such as databases.

### Dead Letter Queue (DLQ)

Messages that cannot be processed after a defined number of attempts can be moved to a Dead Letter Queue.

- Temporary errors, such as short network interruptions: Retry with an appropriate backoff policy.
- Permanent errors, such as invalid schemas: Move to DLQ for investigation.
- After correcting the issue, DLQ messages can be reprocessed with appropriate controls.

## 5. Distributed Data Processing

Distributed Data Processing means processing large volumes of data across multiple machines or nodes instead of relying on a single computer.

The workload is divided into smaller tasks that can run in parallel, improving scalability and processing capacity.

### Common Tools

| Tool | Main Purpose | Typical Use Case |
|---|---|---|
| Apache Spark | Distributed Data Processing | Large-scale batch processing and data transformation |
| Apache Flink | Distributed Stream Processing | Real-time event processing and stateful computations |
| Apache Kafka | Event Streaming | Transferring and retaining events between systems |

### Spark vs. Flink vs. Kafka

- Apache Spark is commonly used for large-scale batch processing, data transformation, and analytical workloads. It also supports stream processing through Structured Streaming.
- Apache Flink is designed for stateful stream processing and applications that require continuous event processing with low latency.
- Apache Kafka handles event transport, storage, and distribution. It is not a replacement for a general-purpose distributed processing engine.

### Practical DMS Example

A DMS may use:

- Kafka to publish document upload and status-change events.
- Apache Flink to process events continuously and identify important changes.
- Apache Spark to process historical document records and generate analytical datasets.

The choice depends on processing requirements, latency, data volume, and operational complexity.

## 6. Apache NiFi and Data Ingestion

### Overview

Apache NiFi is a tool for designing, managing, and executing data flows. It provides a visual interface for data ingestion, validation, transformation, routing, and delivery.

### Core Components

| Component | Responsibility |
|---|---|
| Processor | Performs a specific operation, such as receiving, transforming, or sending data |
| Connection / Queue | Connects Processors and temporarily stores FlowFiles |
| FlowFile | The unit of data managed by NiFi |
| Attributes | Metadata such as filename, document ID, and ingestion timestamp |
| Content | The actual file or data content |
| Provenance | Records the processing history and path of a FlowFile |

### DMS Use Case

NiFi can retrieve files from SFTP, validate them, store them in Object Storage, and publish related events to Kafka.

### Back Pressure

If the incoming data rate exceeds the downstream processing rate, internal NiFi queues grow.

Back Pressure uses configurable thresholds based on data size or FlowFile count to restrict or stop additional data from entering a queue.

Back Pressure does not fix a slow downstream component; it prevents uncontrolled queue growth and resource consumption.

### Provenance and Error Handling

Provenance provides traceability of data throughout the flow. For complete troubleshooting, it should be used alongside logs and metrics from other components, such as Kafka and MinIO.

- Temporary failures: Retry with a defined policy.
- Invalid files: Route to Quarantine.
- Delivery failures: Record the error and retry.
- Partial failures: Recover from the failed stage without unnecessarily repeating successful operations.

## 7. Data Ingestion Tools Comparison

The appropriate tool depends on the source system, data type, and destination.

| Tool | Primary Use Case | Strengths | Limitations / Considerations |
|---|---|---|---|
| Apache NiFi | File and data ingestion and routing | Visual flow design, queue management, and Back Pressure | Requires proper flow design and resource management |
| Kafka Connect | Data transfer between Kafka and external systems | Connectors and task management | Requires suitable connectors for supported sources and destinations |
| Debezium | Capturing database changes through CDC | Captures database changes without relying on simple continuous polling | Depends on database CDC capabilities and configuration |
| Logstash | Log and text-based data ingestion and processing | Filtering, transformation, and integrations with multiple destinations | Primarily focused on log and event data processing |

### Practical Tool Selection

- Receiving and validating files from SFTP: NiFi.
- Transferring PostgreSQL changes to Kafka: Debezium with Kafka Connect.
- Collecting service logs: Logstash or similar tools.
- Delivering Kafka data to supported destinations: Kafka Connect.

These tools can be used together in the same architecture. They are not necessarily direct replacements for one another.

## 8. Storage Types

### Database Storage

Used for structured data and operations such as inserting, updating, and querying records.

Common technologies:

- PostgreSQL
- MySQL
- Microsoft SQL Server
- Oracle Database

DMS example: Document information, user IDs, document ownership, processing status, and registration timestamps.

### File Storage

Traditional File Storage organizes data as files and directories, typically through shared file systems.

Common technologies:

- NAS
- NFS
- Shared File Servers

This model is suitable for systems that require file-based access and shared directory structures.

### Object Storage

Object Storage stores data as objects inside Buckets. Each Object has content and metadata and is identified by an Object Key.

Common technologies:

- MinIO
- Ceph Object Gateway
- Amazon S3

Object Storage is suitable for large files, images, PDFs, backups, and unstructured data.

**Note:** Folder-like structures in Object Storage do not necessarily represent actual directories as in a traditional File System. Object Keys generally define the logical namespace.

## 9. MinIO and Ceph

### MinIO

MinIO is an S3-compatible Object Storage platform.

Core concepts:

- Bucket: A logical container for storing Objects.
- Object: A file together with its metadata.
- Object Key: The identifier or logical path of an Object.
- S3 API: A standard interface for accessing and managing Objects.

MinIO supports distributed deployments and uses Erasure Coding for data protection.

### Ceph

Ceph is a distributed storage platform that provides three primary storage types:

- Object Storage through RADOS Gateway (RGW).
- Block Storage through RBD.
- File Storage through CephFS.

Important components:

- OSD: Stores data and handles storage operations.
- MON: Maintains cluster state and cluster maps.
- Pool: A logical storage area with associated data placement and protection policies.
- Placement Group: An internal structure used to distribute and manage data across the cluster.

### MinIO vs. Ceph

| Criteria | MinIO | Ceph |
|---|---|---|
| Primary Focus | Object Storage | Distributed Storage Platform |
| Object Storage | Yes | Yes, through RGW |
| Block Storage | Not its primary focus | Yes |
| File Storage | Not its primary focus | Yes |
| S3 Compatibility | Yes | Through RGW |
| Operational Complexity | Generally simpler for Object Storage use cases | Generally more complex and multi-purpose |

The final choice should consider project requirements, data volume, operational expertise, required capabilities, and maintenance costs.

## 10. Replication and Erasure Coding

### Replication

Replication stores multiple copies of data across different locations.

For example, three-way Replication requires approximately 300TB of physical storage to store 100TB of raw data, under a simple 3x replication model.

### Erasure Coding

Erasure Coding divides data into multiple fragments and creates additional fragments for data recovery.

Example: 4+2

- Four Data Shards.
- Two Parity Shards.
- Six Shards in total.

When all six shards are distributed across appropriate Failure Domains, the system can tolerate the loss of up to two shards from the same coding set.

For 100TB of raw data, the theoretical storage requirement is approximately 150TB, excluding metadata, reserved capacity, and other overhead.

### Comparison

| Criteria | 3x Replication | 4+2 Erasure Coding |
|---|---|---|
| Approximate Storage Requirement | 3x raw data | 1.5x raw data |
| Storage Overhead | Higher | Lower |
| Recovery | Uses other available copies | Reconstructs data using Data and Parity Shards |
| Complexity | Generally simpler | More complex |
| Key Considerations | Replica placement | Shard distribution and Failure Domains |

The number of Node failures a system can tolerate cannot be determined from the Node count alone. Replica or Shard placement and Failure Domain configuration must also be considered.

Replication is not a replacement for Backup. Data deletion or corruption may also be reflected in replicated copies. Independent backups and recovery policies are therefore required.

## 11. Real-World DMS Architecture

### Proposed Architecture

SFTP / Source Files  
↓  
Apache NiFi  
↓  
Validation and Metadata Extraction  
↓  
MinIO Object Storage  
↓  
Kafka Event Publishing  
↓  
Consumer / Processing Service  
↓  
Analytical Database / Data Warehouse  
↓  
BI and Analytics

PostgreSQL can also be used alongside this flow to store operational document metadata.

### Component Responsibility Matrix

| Component | Responsibility |
|---|---|
| SFTP | File source |
| NiFi | File ingestion, validation, routing, and delivery |
| MinIO | Storage of original files |
| PostgreSQL | Metadata and document status management |
| Kafka | Event delivery and retention |
| Consumer | Event consumption and processing |
| ClickHouse or Data Warehouse | Analytical data storage |
| BI | Reporting and analytics |

### Why Not Store Original Files in Kafka?

Kafka is designed for messages and events. Transferring large files through Kafka can increase storage requirements, network traffic, and the load on Brokers and Consumers.

A better approach is to store the original file in Object Storage and publish only the event and the Object Key required to access it.

Example Event:

```json
{
  "event_id": "evt-1001",
  "document_id": "doc-123",
  "event_type": "DOCUMENT_UPLOADED",
  "object_key": "documents/2026/11/doc-123.pdf",
  "created_at": "2026-11-07T10:30:00Z"
}
```

### Important Consideration: Consistency Between Database, Object Storage, and Kafka

Storing a file in MinIO, registering its metadata in PostgreSQL, and publishing an event to Kafka are separate operations.

One operation may succeed while another fails. For example:

- File stored in MinIO, but metadata registration fails: Orphan Object.
- Metadata registered, but event publishing fails: Consumers are not notified of the event.
- Event published, but the file is unavailable: Consumer processing fails.

To reduce these risks, systems can use unique identifiers, Retry, Reconciliation, and, where appropriate, the Transactional Outbox Pattern.

The Transactional Outbox Pattern can ensure that metadata updates and event records are committed in the same database transaction. A separate process then publishes these events to Kafka.

The publishing process still needs Retry and duplicate-handling mechanisms because an event may be published successfully even if the publisher fails before recording that success.

## 12. Throughput, Latency, and Consumer Lag

### Throughput

The amount of data or number of operations processed within a given period.

Example: 1,000 messages per minute.

### Latency

The time required to perform an operation or transfer data from one point to another.

Example: Receiving and processing a file takes 5 seconds.

### Consumer Lag

Consumer Lag represents the difference between the latest available message position in a Partition and the Consumer's committed Offset.

It indicates how far behind a Consumer is from the latest available messages.

Example:

- Message production rate: 1,000 messages per minute.
- Consumer processing rate: 600 messages per minute.
- Approximate Lag growth: 400 messages per minute.

### Potential Causes of Increasing Lag

- Consumer failure or slow processing.
- Insufficient Partition count for parallel processing.
- Increased message processing time.
- Slow destination Database.
- Slow Object Storage or Network.
- Sudden increases in incoming message volume.

Increasing the number of Consumers is not always the solution. The actual bottleneck must first be identified.

## 13. Failure Scenarios and Recovery

| Scenario | Potential Impact | Recommended Action |
|---|---|---|
| NiFi cannot retrieve a file | File does not enter the processing flow | Retry and check SFTP and queues |
| MinIO cannot store a file | File is unavailable in the destination | Retry and check cluster health and capacity |
| Metadata registration fails | File may exist without traceable metadata | Reconciliation and Orphan Object management |
| Kafka becomes unavailable | Events cannot be published | Retry, persistent NiFi queues, and Back Pressure |
| Consumer stops | Consumer Lag increases | Recover Consumer and investigate Partitions |
| Destination Database slows down | Consumer processing rate decreases | Check queries, indexes, capacity, and connections |
| Database write succeeds but Offset Commit fails | Message may be processed again | Idempotency and Event ID validation |

### Important Recovery Concepts

- Retry: Attempting a failed operation again.
- Idempotency: Preventing unintended duplicate effects.
- DLQ: Isolating messages that cannot be processed.
- Replay: Reading previously published messages again.
- Reprocessing: Running processing logic again on existing data.
- Back Pressure: Controlling queue growth when downstream processing slows down.
- Provenance: Tracking the processing history of data.
- Reconciliation: Comparing the state of different components to identify missing or inconsistent data.

## 14. TPM Technical Checklist

During technical meetings and project follow-ups, the following areas should be monitored.

### Architecture and Dependencies

- What is the responsibility of each tool?
- What are the dependencies between NiFi, Kafka, Object Storage, and Database?
- Can a failure in one component stop the entire data flow?
- Are services designed to operate and recover independently?

### Capacity and Performance

- What is the incoming file and message rate?
- What is the Throughput of each component?
- Within what range is Consumer Lag?
- How much free storage capacity remains?
- What is the daily data growth rate and future storage requirement?

### Reliability and Recovery

- What is the Retry policy?
- Is duplicate processing controlled?
- Are DLQ and Replay mechanisms available?
- What happens if a Node fails?
- What are the expected RPO and RTO?
- Is an independent Backup available?

### Monitoring

- How is Kafka Cluster health monitored?
- How is Consumer Lag tracked?
- What is the status of NiFi queues?
- How are MinIO Node health and storage capacity monitored?
- Are alerts configured for failures and critical thresholds?

### Reporting to Management

Status reports should clearly communicate:

- What is the problem or risk?
- Which components are affected?
- What is the potential impact on data, services, or delivery timelines?
- Has the root cause been identified, or is the investigation still ongoing?
- Who is responsible for follow-up?
- What is the estimated recovery time?
- Are there alternative solutions or mitigation measures?
