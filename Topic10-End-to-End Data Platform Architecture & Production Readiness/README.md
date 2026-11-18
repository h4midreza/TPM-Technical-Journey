# Topic 10: End-to-End Data Platform Architecture & Production Readiness

## 1. Objective

The objective of this final topic is to consolidate the technical and program management concepts covered throughout the learning journey by working through a realistic, end-to-end Data Platform project.

We've simulated the lifecycle of a Data Platform project, starting with business requirements and architecture review, continuing through technical design, implementation planning, deployment, monitoring, and operational readiness, and ending with a final production readiness assessment.

By completing this project, a TPM should be able to:

- Understand and evaluate an end-to-end Data Platform architecture.
- Trace data from source applications to analytical destinations.
- Evaluate database, storage, ingestion, and processing choices.
- Identify technical, infrastructure, and cross-team dependencies.
- Assess data quality, governance, security, and lineage requirements.
- Review Kubernetes/OpenShift deployment and CI/CD readiness.
- Identify performance bottlenecks, capacity constraints, and operational risks.
- Evaluate monitoring, reliability, and recovery capabilities.
- Coordinate technical decisions and assess delivery readiness.
- Communicate architecture gaps, risks, and recommendations to engineering leads and management.

---

## 2. Capstone Project: Telecom Document Management Data Platform

### 2.1 Business Context

A telecom operator uses a Document Management System (DMS) to receive and manage documents from customers and internal applications.

Examples include:

- Customer registration documents.
- Contracts and agreements.
- Identity verification documents.
- Service request attachments.
- Scanned forms and supporting documents.

The current DMS application stores documents and their operational metadata. However, business and operational teams need a centralized analytical platform to monitor document activity and understand processing performance.

The company plans to build a Data Platform that collects document metadata and lifecycle events, stores original files, and provides analytical data for dashboards and reporting.

### 2.2 Business Objectives

The proposed platform should provide:

- Document upload and processing statistics.
- Document volume by type, source, and time period.
- Visibility into successful and failed processing operations.
- Document processing time and service performance.
- Historical reporting and trend analysis.
- Near-real-time operational dashboards.
- Reliable and traceable data delivery.

The platform should support future growth and allow additional source applications and analytical consumers to be integrated.

### 2.3 Project Scope

The first release focuses on document metadata and lifecycle analytics.

**In Scope**

- Document file storage.
- Operational metadata collection.
- Document upload and status events.
- Data validation and transformation.
- Event streaming and analytical loading.
- Historical storage and reporting.
- Monitoring, alerting, and operational recovery.

**Out of Scope for the Initial Release**

- Full-text search inside original documents.
- OCR and advanced document content extraction.
- Machine learning-based document classification.
- Replacing the existing operational DMS application.

These capabilities may be considered in future releases.

---

## 3. Project Requirements and Constraints

Before reviewing the architecture, the TPM works with business stakeholders and Technical Leads to establish the expected workload and operational requirements.

The following values are illustrative assumptions for this case study. They are not yet approved requirements and must be validated during the Project Kickoff.

### 3.1 Expected Workload

| Requirement | Initial Assumption |
|---|---:|
| Documents per day | 20,000 |
| Average document size | 1 MB |
| Approximate daily file storage growth | 20 GB |
| Peak upload rate | 10 documents/second |
| Analytics freshness target | Within 5 minutes |
| Original document retention | 7 years |
| Analytical metadata retention | 3 years |

The estimated 20 GB daily storage growth corresponds to approximately 7.3 TB per year before considering replication, metadata overhead, and storage policies.

### 3.2 Non-Functional Requirements

The project must also define measurable non-functional requirements.

| Area | Initial Assumption |
|---|---|
| Availability | 99.9% service availability |
| Data Freshness | Analytical data available within 5 minutes |
| Data Integrity | No unaccounted document or event loss |
| Scalability | Support expected growth and peak traffic |
| Security | Protected access to documents and metadata |
| Recovery | Defined RPO and RTO |
| Observability | Monitoring and alerting across the complete pipeline |

For planning purposes, the project may initially consider an RPO of 15 minutes and an RTO of 2 hours. These values must be reviewed and approved based on business criticality.

### 3.3 Stakeholders and Technical Teams

The project involves multiple teams:

| Team | Main Responsibility |
|---|---|
| Application Team | Source Application and DMS integration |
| Data Engineering | Data ingestion, NiFi flows, data transformation |
| Platform / DevOps | Kubernetes/OpenShift and CI/CD |
| Storage Team | MinIO and persistent storage |
| Database Team | PostgreSQL and StarRocks |
| Network Team | Network connectivity, DNS, Firewall |
| Security Team | Access control, encryption, and security approval |
| BI / Consumer Team | Dashboards and analytical consumption |
| Operations Team | Monitoring, incident response, and support |
| TPM | Coordination, dependencies, risks, delivery, and readiness |

The TPM coordinates these teams and ensures that technical decisions, dependencies, and delivery commitments remain aligned.

---

## 4. Project Kickoff and Requirements Validation

After defining the initial business context, scope, and assumptions, the project moves into the Kickoff and Requirements Validation stage.

The purpose is to ensure that Business Stakeholders and Technical Leads have a shared understanding of what must be delivered, how success will be measured, and which technical constraints must be considered.

The values listed in Section 3 are initial assumptions. They should not be treated as final commitments until they are reviewed and approved.

### 4.1 Confirm Business Requirements and Scope

The TPM should confirm:

- Which business problems the Data Platform must solve.
- Which reports and dashboards are required.
- Which document types and source systems are included.
- Which document lifecycle events must be captured.
- Which capabilities are included in the first release.
- Which capabilities are explicitly excluded.
- Who will consume the analytical data.

For example, "document processing analytics" must be clarified.

Does the business need only upload statistics, or does it also need to track events such as:

`UPLOADED → VALIDATED → PROCESSING → COMPLETED / FAILED`

This decision affects the data model, event architecture, and downstream analytics.

### 4.2 Validate Non-Functional Requirements

Technical Leads should help clarify whether the proposed performance, availability, and recovery requirements are realistic.

Examples:

**Data Freshness**

Does the five-minute target measure the time from document upload until its information becomes available in StarRocks?

Does it apply to normal conditions, peak load, or both?

**Availability**

Does 99.9% apply to the complete Data Platform, the DMS application, or selected services?

Which components and functions are included in the availability calculation?

**Peak Throughput**

Can the platform handle 10 documents per second continuously, or is this only a short burst?

How long should the peak load be sustained?

**Data Integrity**

What does "no data loss" mean in this context?

Does every successfully uploaded document need to have a corresponding analytical record, or are some event types excluded?

**Retention**

Does the seven-year retention policy apply to all original documents?

Are there legal, regulatory, or business rules for deletion, archiving, and access?

**Recovery**

Are the proposed RPO and RTO applicable to the entire platform or only critical components?

Do the recovery targets differ between the operational application and analytical reporting?

### 4.3 Define Acceptance Criteria

Each important requirement should have a measurable acceptance criterion.

For example:

| Requirement | Possible Acceptance Criterion |
|---|---|
| Data Freshness | 95th percentile of upload-to-StarRocks availability is within 5 minutes under agreed workload |
| Availability | Agreed services achieve 99.9% availability over the measurement period |
| Data Integrity | Source and target reconciliation identifies no unaccounted records during the test |
| Peak Throughput | Platform sustains 10 uploads/second for an agreed duration |
| Recovery | A recovery test demonstrates compliance with approved RPO and RTO |
| Monitoring | Critical failures trigger alerts to the responsible team |

The exact measurement method, test duration, exclusions, and reporting period must be agreed with stakeholders.

### 4.4 Confirm Ownership and Responsibilities

Each important requirement and dependency should have an owner.

Examples:

- Business Stakeholders own business expectations and acceptance.
- Application Team owns source application behavior and event generation.
- Data Engineering owns ingestion and transformation design.
- Platform Team owns deployment infrastructure and CI/CD.
- Database Team owns database readiness and performance.
- Security Team owns security approval and access requirements.
- Operations Team owns monitoring and incident response.
- TPM coordinates cross-team commitments and tracks unresolved decisions.

The RACI model can be used where ownership or approval responsibilities are unclear.

### 4.5 Record Assumptions, Constraints, and Open Questions

Not every requirement can be resolved during the first meeting.

The TPM should maintain a record of:

- Assumptions that still need confirmation.
- Technical or infrastructure constraints.
- External dependencies.
- Unresolved design questions.
- Decisions requiring stakeholder approval.
- Risks caused by missing information.

For example:

| Item | Type | Owner | Status |
|---|---|---|---|
| Five-minute freshness target | Requirement to validate | Business / Data Engineering | Open |
| Ten uploads per second | Capacity assumption | Application / Platform | Open |
| Seven-year file retention | Business constraint | Business / Security | Open |
| MinIO deployment model | Architecture decision | Storage Team | Open |
| RPO and RTO | Recovery requirement | Business / Operations | Open |

### 4.6 Kickoff Outputs

At the end of this stage, the project should have:

- Agreed business objectives and initial scope.
- A validated requirements baseline.
- Approved or explicitly provisional non-functional requirements.
- Measurable acceptance criteria.
- Named owners for major requirements and dependencies.
- An assumptions and constraints register.
- An open questions and decision log.
- A shared understanding of the next technical activities.

The outcome of Kickoff becomes the baseline for architecture review and technical planning.

---

## 5. Initial Reference Architecture

The engineering team proposes the following architecture:

```text
Source Application
       |
       v
     MinIO
       |
       v
      NiFi
       |
       v
     Kafka
       |
       v
   StarRocks

Prometheus + Grafana
     Monitoring
```

The architecture provides the initial direction, but it is not yet a complete production design.

During the architecture review, the TPM and Technical Leads must identify missing components, unclear data paths, and operational requirements.

### 5.1 Operational Database

The Source Application also requires an operational database such as PostgreSQL.

It stores information such as:

- Document ID.
- Customer or business reference.
- Document type.
- Upload status.
- Processing status.
- Creation and update timestamps.
- Object storage reference.

The operational database is not explicitly shown in the initial diagram and must be added to the detailed architecture.

### 5.2 Refined Architecture

The reviewed architecture may look like this:

```text
                      SOURCE APPLICATION
                              |
                 +------------+------------+
                 |                         |
                 v                         v
              MinIO                    PostgreSQL
          (Object Storage)          (Operational DB)
                 |                         |
                 |                   Metadata / Outbox
                 |                         |
                 +------------+------------+
                              |
                              v
                             NiFi
                   (Discovery, Ingestion,
                    Validation, Routing)
                              |
                              v
                            Kafka
                       (Event Streaming)
                              |
                              v
                          StarRocks
                      (Analytical Database)
                              |
                              v
                       BI / Reporting
                       / Analytics


              Prometheus + Grafana
                 Monitoring Platform
```

This design retains the original data pipeline and adds the operational database and its data integration path.

The exact integration mechanism between PostgreSQL, MinIO, and NiFi must be selected during the project design.

---

## 6. Project Phase 1: Architecture Review and Component Responsibilities

The first engineering activity is to establish what each component does and why it is required.

### Source Application

Receives documents from users or upstream applications and manages the business workflow.

It initiates document uploads and maintains the document's operational status.

### PostgreSQL – Operational Database

Stores transactional metadata and the current document state.

It supports application operations such as document lookup, status updates, and metadata retrieval.

### MinIO – Object Storage

Stores the original document files.

The application stores references to these objects in PostgreSQL.

MinIO is responsible for file storage, not transactional document management.

### Apache NiFi – Ingestion and Data Flow Management

Connects to source systems and storage services, discovers new objects or records, validates data, transforms it, and routes events to Kafka.

### Apache Kafka – Event Streaming

Receives and retains validated events and makes them available to downstream consumers.

It decouples data ingestion from analytical loading and supports parallel consumption and replay within the configured retention period.

### StarRocks – Analytical Database

Consumes the events and stores structured analytical data.

It supports reporting, aggregations, historical analysis, and business dashboards.

### Prometheus and Grafana

Prometheus collects metrics from the infrastructure and applications. Grafana provides dashboards and visualization.

Alert notification and incident routing may require additional integrations depending on the operations model.

---

## 7. Project Phase 2: Source-to-Target Data Flow

After understanding the components, the project team defines how a document moves through the platform.

Consider a customer uploading a contract.

### Step 1 – Document Registration

The Source Application creates a document record in PostgreSQL.

Initially, the document may have a status such as:

`UPLOAD_PENDING`

### Step 2 – Object Upload

The application uploads the original PDF file to MinIO.

Example:

```text
Bucket: dms-documents
Object Key: contracts/2026/11/DOC-10245.pdf
```

### Step 3 – Operational Metadata Update

After a successful upload, the application updates the document status in PostgreSQL:

`UPLOAD_PENDING → UPLOADED`

The object reference is also stored in the metadata record.

The application should account for failures between object upload and database update. A reconciliation mechanism is necessary to identify incomplete or inconsistent operations.

### Step 4 – Object and Event Discovery

NiFi must discover the new document and its metadata.

Possible approaches include:

- Periodically listing new MinIO objects.
- Receiving object-created notifications.
- Reading records from a PostgreSQL Outbox table.
- Using CDC when continuous database change capture is required.

The project team must select a suitable approach based on freshness requirements, throughput, operational complexity, and recovery behavior.

An important design consideration is that MinIO object creation alone does not capture every later document status change.

If the analytical platform must report lifecycle events, those changes must also be captured from the application or operational database.

### Step 5 – Data Validation and Transformation

NiFi validates the metadata and prepares a standardized event.

Example checks:

- Required fields are present.
- Document ID follows the agreed format.
- Document type is valid.
- Object reference is available.
- Event timestamp is valid.
- Schema version is supported.
- Duplicate events can be identified.

Invalid records should be routed to a controlled error or quarantine path.

### Step 6 – Kafka Publishing

NiFi publishes the validated event to a Kafka topic.

Example topic:

`dms.document-events.v1`

Example event:

```json
{
  "event_id": "EVT-90001",
  "event_type": "DOCUMENT_UPLOADED",
  "event_time": "2026-11-17T10:30:00Z",
  "document_id": "DOC-10245",
  "document_type": "CONTRACT",
  "object_bucket": "dms-documents",
  "object_key": "contracts/2026/11/DOC-10245.pdf",
  "source_system": "DMS",
  "schema_version": "1.0"
}
```

The Kafka event should normally contain metadata and an object reference, not the entire PDF file.

### Step 7 – Analytical Loading

StarRocks consumes the Kafka events and loads them into analytical tables.

A possible implementation is StarRocks Routine Load. Other integrations, such as Kafka Connect Sink, may be evaluated based on transformation requirements and operational preferences.

### Step 8 – Analytical Consumption

BI tools and reporting applications query StarRocks.

Business users can analyze:

- Number of uploaded documents.
- Document volume by type.
- Processing success and failure rates.
- Average processing time.
- Historical trends.

The original file remains in MinIO, operational metadata remains in PostgreSQL, and analytical records are stored in StarRocks.

---

## 8. Project Phase 3: Database and Storage Design

The next activity is to validate the storage architecture and ensure that each technology is used for the right workload.

### 8.1 OLTP and OLAP Workload Separation

**PostgreSQL – OLTP**

PostgreSQL supports the operational application.

Typical operations:

- Creating document records.
- Updating document status.
- Retrieving document metadata.
- Searching for individual documents.
- Managing transactional workflows.

Its workload includes frequent inserts, updates, and point queries.

**StarRocks – OLAP**

StarRocks supports analytical workloads.

Typical operations:

- Counting documents by day.
- Calculating processing failure rates.
- Aggregating document volumes by source or type.
- Analyzing historical trends.
- Serving reporting dashboards.

Its workload includes large scans, aggregations, and analytical queries.

**Architecture Decision**

Heavy analytical queries should not compete with operational transactions.

Therefore, the project separates OLTP and OLAP workloads and transfers analytical data through the ingestion pipeline.

### 8.2 Storage Responsibilities

| Technology | Storage Responsibility |
|---|---|
| PostgreSQL | Operational records and metadata |
| MinIO | Original document files |
| Kafka | Retained event streams |
| StarRocks | Analytical tables |
| Kubernetes Persistent Volumes | Persistent storage for stateful services |

Persistent volumes may be required by NiFi, Kafka, PostgreSQL, StarRocks, and monitoring services, depending on the deployment model.

MinIO is object storage and should not be confused with Kubernetes block storage.

### 8.3 Backup and Recovery

The project must define backup and recovery for each persistent component.

For example:

- PostgreSQL backups and transaction log recovery.
- MinIO replication, versioning, or backup strategy.
- Kafka retention and replication.
- StarRocks backup and restore.
- NiFi flow configuration and repository recovery.

Kafka retention is useful for replay but does not replace a complete backup strategy.

---

## 9. Project Phase 4: Data Quality, Data Contract, and Governance

The engineering team must establish a consistent data contract between the source and downstream systems.

### 9.1 Data Contract

The contract defines:

- Field names and data types.
- Required and optional fields.
- Field meaning and valid values.
- Event types and their semantics.
- Timestamp formats.
- Schema versioning rules.
- Compatibility requirements.
- Ownership and change approval.

For example, changing `document_id` from Integer to String should be treated as a contract change.

The team should define how producers and consumers are migrated without breaking existing integrations.

### 9.2 Data Quality

Validation controls should cover:

- Missing required fields.
- Invalid data types.
- Invalid document types.
- Incorrect timestamps.
- Invalid object references.
- Unsupported file formats.
- Duplicate events.
- Missing or inconsistent records.

Invalid records must be traceable and have a defined handling process.

### 9.3 Idempotency and Reconciliation

The platform must account for duplicate or repeated events.

A stable `event_id` can help identify duplicate events. Downstream processing must be designed so that repeated delivery does not create incorrect analytical results.

Reconciliation compares source records and target records to detect missing or inconsistent data.

For example:

- Documents marked as UPLOADED in PostgreSQL.
- Corresponding objects in MinIO.
- Events published to Kafka.
- Records loaded into StarRocks.

### 9.4 Data Lineage

Each analytical record should be traceable to its source.

Useful lineage attributes include:

- Source system.
- Document ID.
- Object bucket and key.
- Event ID.
- Schema version.
- Kafka topic, partition, and offset.
- Processing timestamp.
- Target table.
- Load status.

### 9.5 Security and Governance

The team must define:

- Data classification.
- Role-based access control.
- Encryption in transit and at rest.
- Secret management.
- Audit logging.
- Retention and deletion policies.
- Protection of sensitive document metadata.
- Access restrictions for original documents.

---

## 10. Project Phase 5: Infrastructure and Kubernetes/OpenShift Design

After the logical architecture and data requirements are clear, the team prepares the deployment architecture.

### 10.1 Deployment Model

The components may run inside Kubernetes/OpenShift or use externally managed services.

The deployment model must be selected based on operational capability, availability, resource requirements, and existing infrastructure.

| Component | Deployment Considerations |
|---|---|
| Source Application | Application deployment and database connectivity |
| PostgreSQL | Stateful deployment or managed database |
| MinIO | Distributed object storage or managed object storage |
| NiFi | Cluster deployment, repositories, and persistent volumes |
| Kafka | Stateful cluster or managed Kafka service |
| StarRocks | FE/BE deployment topology and persistent storage |
| Prometheus | Metrics collection and time-series storage |
| Grafana | Dashboards and alert visualization |

### 10.2 Kubernetes Infrastructure Requirements

The platform team must evaluate:

- Cluster capacity and worker nodes.
- CPU and memory requests and limits.
- Storage classes and persistent volumes.
- Network policies and firewall rules.
- DNS and service discovery.
- Ingress and external connectivity.
- ConfigMaps and Secrets.
- Service accounts and RBAC.
- Liveness, readiness, and startup probes.
- Pod disruption budgets.
- Scheduling and failure-domain awareness.

### 10.3 Stateful and Stateless Components

The deployment plan must distinguish stateful from stateless workloads.

For example:

- Stateless application pods can generally be replaced without retaining local data.
- Kafka brokers require persistent storage for retained events.
- PostgreSQL requires persistent database storage.
- MinIO requires durable object storage.
- NiFi requires appropriate repository persistence and cluster coordination.
- StarRocks requires persistent data storage for its analytical nodes.

A Pod being Running does not necessarily mean the service is ready to process traffic. Application-level health and dependency checks are also necessary.

---

## 11. Project Phase 6: CI/CD and Release Planning

The project team defines how application code, data flows, schemas, and infrastructure configurations are deployed.

### 11.1 CI/CD Pipeline

A typical release pipeline includes:

1. Source control for application code and configuration.
2. Build and container image creation.
3. Static validation of configuration and deployment manifests.
4. Unit, integration, and contract testing.
5. Security and image scanning.
6. Deployment to test environments.
7. End-to-end integration testing.
8. Performance and load testing.
9. Production approval.
10. Controlled deployment and post-release monitoring.

### 11.2 Release Dependencies

A production release depends on several components being ready.

For example:

- PostgreSQL schema and metadata fields must be available.
- MinIO buckets and credentials must be configured.
- NiFi flows must be deployed and validated.
- Kafka topics and permissions must be ready.
- StarRocks tables and loading jobs must be created.
- Network connectivity must be confirmed.
- Monitoring dashboards and alerts must be active.

### 11.3 Release and Rollback Strategy

The team should define:

- Deployment sequence.
- Version compatibility.
- Database migration approach.
- Schema evolution strategy.
- Canary or gradual rollout where appropriate.
- Rollback conditions.
- Data replay and reconciliation procedures.

Rollback of an application version does not automatically reverse data already written to Kafka or StarRocks. Data compatibility and recovery must be considered separately.

---

## 12. Project Phase 7: Performance and Capacity Planning

The architecture must support expected workloads and future growth.

### 12.1 Capacity Dimensions

**Storage Capacity**

- Documents per day.
- Average document size.
- Daily and annual growth.
- Retention period.
- Replication overhead.
- Backup requirements.

**Processing Throughput**

- Peak document uploads per second.
- NiFi processing capacity.
- Kafka producer throughput.
- Kafka consumer throughput.
- StarRocks loading capacity.

**Latency**

- Document upload time.
- Object discovery delay.
- NiFi processing time.
- Kafka delivery delay.
- Analytical loading delay.
- End-to-end data freshness.

**Infrastructure Resources**

- CPU and memory.
- Disk capacity and IOPS.
- Network bandwidth.
- Kafka partition count.
- NiFi repository capacity.
- StarRocks compute resources.

### 12.2 Example: Kafka Consumer Lag

Assume Kafka receives:

`2,000 events/second`

StarRocks consumes:

`1,200 events/second`

The consumer cannot keep up with the producer, so Kafka lag increases.

The team must investigate:

- Consumer throughput.
- Kafka partition count.
- Load job status.
- StarRocks resource utilization.
- Network latency.
- Data format and transformation overhead.

Adding more consumers does not necessarily solve the problem if the number of partitions or downstream database capacity is the limiting factor.

The TPM coordinates the investigation and evaluates whether the problem affects the delivery commitment or production service level.

---

## 13. Project Phase 8: Monitoring and Operational Readiness

The team defines the monitoring architecture before production release.

### 13.1 Infrastructure Monitoring

Prometheus collects metrics such as:

- CPU and memory utilization.
- Disk capacity and I/O.
- Network throughput and errors.
- Pod restarts.
- Resource throttling.
- Persistent volume utilization.
- Node and cluster availability.

### 13.2 Component Monitoring

| Component | Important Metrics |
|---|---|
| MinIO | Storage usage, API errors, request latency, object upload failures |
| NiFi | FlowFiles In/Out, queue size, processing time, failed FlowFiles, Back Pressure, repository usage |
| Kafka | Producer/consumer throughput, consumer lag, offline partitions, under-replicated partitions, disk usage |
| StarRocks | Load job status, loaded/rejected rows, query latency, FE/BE health, CPU and memory |
| PostgreSQL | Connections, query latency, transaction errors, replication lag, storage usage |
| Kubernetes | Pod health, restarts, CPU, memory, node and volume status |

### 13.3 End-to-End Data Monitoring

Infrastructure health alone does not prove that data is being delivered correctly.

The team must also monitor:

- Time from document upload to analytical availability.
- Source-to-target record reconciliation.
- Number of valid and rejected records.
- Duplicate event count.
- Data freshness.
- Missing or delayed records.
- Pipeline success rate.

### 13.4 Operational Support

Before production, define:

- Service ownership.
- On-call responsibility.
- Alert routing.
- Incident severity.
- Escalation paths.
- Operational runbooks.
- Recovery and replay procedures.
- Capacity monitoring.
- Support handover and knowledge transfer.

---

## 14. Project Phase 9: Technical Risk Assessment and Recovery

During architecture review and testing, the team identifies failure scenarios and defines how the platform should respond.

| Scenario | Potential Impact | Mitigation | Recovery |
|---|---|---|---|
| MinIO notification is missed | Document is not discovered | Periodic reconciliation | Re-discover and process missing objects |
| PostgreSQL and MinIO become inconsistent | Metadata references missing objects or vice versa | Upload state management and reconciliation | Repair inconsistent records |
| NiFi queue grows continuously | Delayed ingestion and disk pressure | Capacity planning and Back Pressure | Reduce intake and resolve bottleneck |
| Kafka consumer lag increases | Analytical data becomes stale | Throughput monitoring and capacity planning | Resume or replay retained events |
| Schema incompatibility | Failed or incorrect records | Contract validation and compatibility testing | Deploy compatible producer/consumer |
| StarRocks load job stops | Analytics becomes stale | Load job monitoring | Restore job and reload missing events |
| Duplicate events are processed | Incorrect analytical counts | Idempotency and duplicate detection | Deduplicate or rebuild affected data |
| Persistent volume becomes full | Service degradation or failure | Capacity alerts and retention policies | Expand storage or clean up safely |
| Credentials expire | Integration failure | Secret rotation and expiry monitoring | Renew credentials and restore connectivity |
| Kubernetes node fails | Service interruption | Redundancy and failure-domain awareness | Reschedule and recover stateful workloads |

### 14.1 Recovery Planning

For each critical scenario, the team must define:

- How the failure is detected.
- Which components are affected.
- Whether data is retained.
- Whether processing can continue.
- How missing data is identified.
- How events are replayed.
- How duplicate processing is prevented.
- Who owns the recovery activity.
- How service recovery is verified.

For example, if StarRocks becomes unavailable, Kafka may continue retaining events. Once StarRocks recovers, the loading process can resume or replay events within the retention period.

Kafka retention must be sufficient for the expected recovery window. Otherwise, historical data may need to be rebuilt from source records or MinIO objects.

---

## 15. Project Phase 10: Architecture Decisions and Trade-offs

During the project, the TPM coordinates important technical decisions with the relevant Technical Leads.

These decisions should be recorded using Architecture Decision Records (ADR).

### Example 1 – MinIO Object Discovery

**Option A: Periodic Listing**

NiFi periodically checks MinIO for new objects.

Advantages:
- Simple recovery and reconciliation.
- Less dependency on notification delivery.

Trade-offs:
- Discovery delay.
- Listing overhead.

**Option B: Event Notification**

MinIO sends object-created notifications to an integration endpoint.

Advantages:
- Lower discovery latency.
- Event-driven processing.

Trade-offs:
- Notification delivery and retry behavior must be understood.
- Reconciliation is still required.
- Additional integration dependencies are introduced.

### Example 2 – PostgreSQL Change Capture

**Option A: Periodic Database Polling**

NiFi periodically retrieves new or updated records.

Advantages:
- Relatively simple to operate.
- Suitable for moderate workloads.

Trade-offs:
- Polling delay.
- Incremental query and checkpoint management.

**Option B: Change Data Capture (CDC)**

A CDC mechanism captures database changes and makes them available to the pipeline.

Advantages:
- Continuous change capture.
- Reduced polling overhead.

Trade-offs:
- Additional operational complexity.
- Requires monitoring of replication and change capture.
- Schema and transaction behavior must be understood.

### Example 3 – Kafka to StarRocks

**Option A: StarRocks Routine Load**

Suitable for continuous loading of supported Kafka message formats.

**Option B: Kafka Connect Sink**

Useful when Kafka Connect integration or additional transformation capabilities are required.

The decision should consider delivery semantics, format support, operational ownership, monitoring, and recovery.

### Example 4 – OLTP and OLAP Separation

**Option A: Use PostgreSQL for Reporting**

Advantages:
- Fewer components.
- Simpler initial architecture.

Trade-offs:
- Analytical queries compete with operational transactions.
- Performance may degrade as data volume grows.

**Option B: Use StarRocks for Analytics**

Advantages:
- Workload isolation.
- Independent analytical scaling.
- Better alignment with large-scale analytical queries.

Trade-offs:
- More infrastructure and operational complexity.
- Requires reliable data ingestion and freshness monitoring.

### ADR Structure

Each significant decision should record:

- Decision title and date.
- Context and problem.
- Options considered.
- Selected option.
- Reasons for selection.
- Consequences and trade-offs.
- Owner and approval.
- Conditions for revisiting the decision.

---

## 16. Project Phase 11: Production Readiness and Go-Live Decision

After implementation, integration testing, and operational preparation, the TPM coordinates the final readiness review.

The goal is to establish whether the platform can safely enter production.

### 16.1 Architecture Readiness

- Components and responsibilities are documented.
- Source-to-target flow is validated.
- Database and storage choices are justified.
- Critical dependencies have owners.
- High-impact architecture gaps are resolved or formally accepted.

### 16.2 Data Readiness

- Data Contracts are documented and approved.
- Schema compatibility is validated.
- Data quality rules are implemented.
- Invalid records have a defined handling path.
- Lineage and reconciliation are available.
- Historical data requirements are understood.

### 16.3 Infrastructure Readiness

- Required environments and resources are available.
- Network connectivity is tested.
- Persistent storage is configured.
- Security and access controls are approved.
- Backup and recovery procedures are documented and tested.

### 16.4 Release Readiness

- CI/CD pipeline is validated.
- Automated tests pass.
- Integration and performance tests are completed.
- Deployment and rollback plans are reviewed.
- Monitoring and alerts are active.
- Release owners and support teams are assigned.

### 16.5 Operational Readiness

- Runbooks are available.
- Incident severity and escalation paths are defined.
- Recovery and replay procedures are tested.
- Capacity and growth assumptions are documented.
- Service and data quality targets are agreed.
- Operational ownership is confirmed.

### 16.6 Example Acceptance Criteria

The following values are illustrative and must be agreed with business and engineering stakeholders:

- At least 99.9% of successfully uploaded documents are represented in the ingestion process.
- No unaccounted data loss is observed during the agreed test period.
- Analytical data freshness remains within five minutes under the agreed workload.
- Invalid records are identifiable and traceable.
- Kafka lag returns to an acceptable level after a controlled load test.
- Critical alerts are triggered and delivered to the responsible team.
- Recovery and replay tests complete successfully.
- Dashboard results reconcile with an agreed source dataset.

### 16.7 Final Go-Live Recommendation

The TPM consolidates the readiness findings and provides one of the following recommendations:

**Go**

Critical technical and operational requirements are met, and the platform can proceed to production.

**Conditional Go**

Remaining gaps are understood, their risks are accepted by the appropriate owners, and corrective actions have agreed owners and deadlines.

**No-Go**

Critical technical, security, reliability, or operational requirements remain unresolved and production delivery should be delayed.

The recommendation must be supported by test evidence, architecture decisions, dependency status, and risk assessment.

---

## 17. Final Project Deliverables

The following deliverables are produced progressively as the project moves through its lifecycle. They are consolidated during the final readiness review.

1. **End-to-End Data Platform Architecture Diagram** – Complete component and integration view.
2. **Data Flow Diagram** – Source-to-target flow, including persistence, processing, and transport.
3. **Component and Responsibility Matrix** – Component responsibilities and team ownership.
4. **Data Quality and Governance Checklist** – Data Contracts, validation, lineage, security, and retention.
5. **Deployment and Infrastructure Dependency Map** – Infrastructure, connectivity, storage, and team dependencies.
6. **Monitoring and Reliability Checklist** – Metrics, dashboards, alerts, and recovery requirements.
7. **Technical Risk Register** – Prioritized risks, mitigation, recovery, and owners.
8. **Architecture Decision and Trade-off Summary** – Important decisions, alternatives, and consequences.
9. **Final Technical Readiness Report** – Project status, readiness gaps, corrective actions, and Go/No-Go recommendation.
10. **Executive Presentation** – A concise presentation summarizing the architecture, major risks, readiness status, and recommendations.
