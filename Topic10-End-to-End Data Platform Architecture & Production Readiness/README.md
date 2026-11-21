# Topic 10: End-to-End Data Platform Architecture & Production Readiness

## 1. Objective

The objective of this final topic is to consolidate the technical and program management concepts covered throughout the learning journey by working through a realistic Data Platform project.

The project simulates the lifecycle of a telecom Document Management Data Platform, from business requirements and architecture design to infrastructure, deployment, monitoring, risk management, and production readiness.

The focus is not on implementing every component, but on understanding how the components work together, evaluating technical decisions, identifying dependencies and risks, and ensuring that the platform is ready for production.

By completing this project, a TPM should be able to:

- Review and evaluate an end-to-end Data Platform architecture.
- Understand source-to-target data flows and component responsibilities.
- Evaluate database, storage, ingestion, and processing choices.
- Identify technical, infrastructure, and cross-team dependencies.
- Assess data quality, governance, security, and lineage requirements.
- Review Kubernetes/OpenShift deployment and CI/CD readiness.
- Identify performance bottlenecks and capacity constraints.
- Evaluate monitoring, reliability, and recovery capabilities.
- Coordinate architecture decisions and delivery commitments.
- Prepare a technical readiness assessment and communicate recommendations.

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

The existing DMS application stores original documents and their operational metadata. However, business and operational teams need a centralized analytical platform to monitor document activity and understand processing performance.

The company plans to build a Data Platform that collects document metadata and lifecycle events, stores original files, and provides analytical data for dashboards and reporting.

### 2.2 Business Objectives

The platform should provide:

- Document volume by type, source, and time period.
- Visibility into document processing status and failure rates.
- Analysis of document processing time.
- Near-real-time operational dashboards.
- Historical reporting and trend analysis.
- Reliable and traceable data delivery.
- Scalability for additional source systems and analytical consumers.

### 2.3 Project Scope

**In Scope – Initial Release**

- Document file storage.
- Operational metadata collection.
- Document upload and lifecycle events.
- Data ingestion, validation, and transformation.
- Event streaming and analytical loading.
- Historical storage and reporting.
- Monitoring, alerting, and recovery.

**Out of Scope – Initial Release**

- Full-text search inside documents.
- OCR and advanced document content extraction.
- Machine learning-based document classification.
- Replacing the existing operational DMS application.

These capabilities may be considered in future releases.

---

## 3. Project Requirements and Constraints

The following values are initial project assumptions. They must be validated with business stakeholders and technical teams before being treated as final commitments.

### 3.1 Expected Workload

| Requirement | Initial Assumption |
|---|---:|
| Documents per day | 20,000 |
| Average document size | 1 MB |
| Daily file storage growth | 20 GB |
| Peak upload rate | 10 documents/second |
| Analytics freshness | Within 5 minutes |
| Original document retention | 7 years |
| Analytical metadata retention | 3 years |

The estimated daily storage growth is approximately 20 GB, or 7.3 TB per year, before replication, backup, and storage overhead.

### 3.2 Non-Functional Requirements

| Area | Initial Assumption |
|---|---|
| Availability | 99.9% |
| Data Freshness | P95 within 5 minutes |
| Data Integrity | No unaccounted data loss during normal operation |
| Scalability | Support expected growth and peak traffic |
| Security | Protected access to documents and metadata |
| RPO | 15 minutes during disaster recovery |
| RTO | 2 hours |

These values are provisional. The measurement scope and acceptance criteria must be agreed during project kickoff.

### 3.3 Stakeholders and Technical Teams

| Team | Main Responsibility |
|---|---|
| Application Team | DMS application and event generation |
| Data Engineering | NiFi flows, ingestion, and transformation |
| Platform / DevOps | Kubernetes/OpenShift and CI/CD |
| Storage Team | MinIO and persistent storage |
| Database Team | PostgreSQL and StarRocks |
| Network Team | Connectivity, DNS, and firewall |
| Security Team | Access control, encryption, and security approval |
| BI / Consumer Team | Dashboards and analytical consumption |
| Operations Team | Monitoring and incident response |
| TPM | Dependencies, risks, coordination, and delivery readiness |

---

## 4. Project Kickoff and Requirements Validation

Before architecture design begins, the TPM organizes a kickoff and requirements validation process.

The purpose is to ensure that Business Stakeholders and Technical Leads agree on the expected functionality, workload, performance, security, and recovery requirements.

### 4.1 Validate Business Requirements

For example, the business requests near-real-time document analytics.

This must be clarified:

- Which document types are included?
- Which lifecycle events must be captured?
- Are upload statistics sufficient, or is the complete processing history required?
- Which reports and dashboards are expected?
- Which users and applications will consume the data?

The answers determine the required data model, events, and analytical tables.

### 4.2 Define Measurable Requirements

A requirement must have a measurement method and an acceptance criterion.

For example:

**Data Freshness**

The initial target is five minutes. The measurement should start when the document is successfully registered as uploaded in the Source Application and end when its analytical record becomes queryable in StarRocks.

A possible acceptance criterion is:

`P95 End-to-End Data Latency ≤ 5 minutes`

**Peak Throughput**

The initial target is 10 documents per second. Business must clarify how long this peak is expected to continue and how frequently it occurs.

The technical teams must validate that the platform can sustain this rate without unacceptable queue growth or data freshness degradation.

**Availability**

Availability should be evaluated separately for important capabilities:

- Document ingestion.
- Object storage.
- Analytical querying.
- End-to-end analytical service.

A single component may become unavailable while the rest of the platform continues working.

### 4.3 Data Integrity and Recovery Requirements

The initial expectation is:

- No loss of successfully accepted documents or events during normal operation.
- A maximum RPO of 15 minutes under the agreed disaster recovery scenario.
- A maximum RTO of 2 hours.

The RPO and RTO must be reviewed separately for PostgreSQL, MinIO, Kafka, and StarRocks.

### 4.4 Ownership and Open Questions

The TPM records the owner, status, and required resolution date for each open requirement.

For example:

| Open Item | Owner |
|---|---|
| Peak duration and frequency | Business / Application |
| File size distribution | Application Team |
| Data freshness measurement | Business / Data Engineering |
| Availability targets | Business / Operations |
| Storage retention and capacity | Storage Team |
| RPO and RTO approval | Business / Operations |

### Kickoff Output

The outcome is a validated requirements baseline containing agreed scope, acceptance criteria,Functional & Non-Functional Requirements, assumptions, constraints, owners and open decisions.

These requirements become the basis for architecture and capacity planning.

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

This is the initial architecture, not yet the complete production design.

During review, the TPM and Technical Leads identify missing components and unclear data paths.

For example, the architecture does not explicitly show where operational metadata is stored or how document status changes are captured.

### 5.1 Refined Architecture

The reviewed architecture includes PostgreSQL and a CDC path for lifecycle events.

```text
                    SOURCE APPLICATION
                            |
                 +----------+----------+
                 |                     |
                 v                     v
              MinIO                PostgreSQL
          (Object Storage)        (Operational DB)
                 |                     |
                 v                 Outbox Table
                NiFi                   |
       Discovery / Validation          v
                 |                  Debezium
                 |                     |
                 v                     v
                 +-------> Kafka <-----+
                            |
                            v
                        StarRocks
                            |
                            v
                      BI / Reporting


                       Prometheus
                           |
                        Grafana
```

The two ingestion paths have different responsibilities:

- NiFi discovers and validates document objects and related information.
- Debezium captures business lifecycle events from the PostgreSQL Outbox and publishes them to Kafka.

Their events are correlated using a common `document_id`.

This avoids relying on MinIO notifications to identify business status changes.

---

## 6. Architecture Review and Component Responsibilities

The architecture contains components with different responsibilities.

| Component | Responsibility | Data Role |
|---|---|---|
| Source Application | Upload documents and manage business workflow | Data producer |
| PostgreSQL | Store operational metadata and current status | OLTP |
| MinIO | Store original PDF and image files | Object storage |
| NiFi | Discover, validate, and route object-related data | Ingestion |
| Debezium | Capture PostgreSQL Outbox changes | CDC |
| Kafka | Retain and distribute events | Messaging |
| StarRocks | Store and query analytical data | OLAP |
| Prometheus | Collect metrics | Monitoring |
| Grafana | Visualize metrics and dashboards | Monitoring interface |

### 6.1 Document Upload and Data Consistency

When a customer uploads a document, the Source Application generates a `document_id`.

The application first creates an `UPLOAD_PENDING` record in PostgreSQL. It then uploads the file to MinIO.

After a successful upload, the application updates the status to `UPLOADED` and inserts the corresponding Outbox event within the same PostgreSQL transaction.

This does not make the MinIO upload and database transaction atomic. Therefore, a reconciliation mechanism must detect incomplete operations.

For example, if the MinIO upload succeeds but the PostgreSQL update fails, the original `UPLOAD_PENDING` record remains available for investigation and recovery.

If the initial database operation fails, a controlled temporary upload or cleanup process may be required.

### 6.2 Capturing Document Lifecycle Events

Document status may change through several stages:

`UPLOAD_PENDING → UPLOADED → VALIDATED → PROCESSING → COMPLETED / FAILED`

MinIO notifications can identify object creation, but they do not represent every business status change.

The proposed solution is Transactional Outbox with Debezium:

1. Update the document status in PostgreSQL.
2. Insert the corresponding event into the Outbox table in the same transaction.
3. Debezium captures the Outbox change.
4. Kafka receives the lifecycle event.

This reduces the risk of a successful database update without a corresponding business event.

### 6.3 NiFi and CDC Responsibilities

NiFi and Debezium should not be treated as interchangeable tools.

- NiFi manages data discovery, routing, validation, and integration flows.
- Debezium captures database changes through CDC.
- Kafka provides the messaging layer between producers and consumers.

For this project, Debezium can publish lifecycle events directly to Kafka. NiFi remains responsible for object discovery and any additional validation or metadata processing that the design requires.

This prevents unnecessary processing steps and duplicate event generation.

---

## 7. Source-to-Target Data Flow

Consider a customer uploading a contract document.

### 7.1 Document Upload

The Source Application creates:

`document_id = DOC-10245`

The original PDF is stored in MinIO:

`dms-documents/contracts/DOC-10245.pdf`

PostgreSQL stores its metadata and current status:

```json
{
  "document_id": "DOC-10245",
  "document_type": "CONTRACT",
  "status": "UPLOADED",
  "object_bucket": "dms-documents",
  "object_key": "contracts/DOC-10245.pdf"
}
```

### 7.2 Object Discovery

NiFi discovers the object in MinIO and verifies its metadata or existence according to the agreed validation design.

Typical processors may include `ListS3` and, where object content is required, `FetchS3Object`.

Since the initial release focuses on metadata analytics, the full PDF does not need to pass through Kafka.

### 7.3 Lifecycle Event Publishing

Debezium captures the Outbox event and publishes it to a Kafka topic such as:

`dms.document-events.v1`

Example event:

```json
{
  "event_id": "EVT-90002",
  "event_type": "DOCUMENT_STATUS_CHANGED",
  "event_version": "1.0",
  "document_id": "DOC-10245",
  "aggregate_version": 4,
  "event_time": "2026-11-20T10:05:00Z",
  "previous_status": "PROCESSING",
  "new_status": "COMPLETED",
  "source_system": "DMS"
}
```

### 7.4 Analytical Loading

StarRocks consumes Kafka events through a suitable integration, such as Routine Load.

The event is stored in the analytical history table and used to update the derived current-state model.

### 7.5 Analytical Consumption

BI applications query StarRocks for document statistics, processing trends, and failure reports.

PostgreSQL remains the source of truth for operational status, while StarRocks provides the analytical view.

---

## 8. Database and Storage Design

The project uses different storage technologies because each serves a different workload.

### 8.1 OLTP and OLAP Separation

**PostgreSQL – Operational Workload**

Used for:
- Document registration.
- Current status updates.
- Individual document lookups.
- Transactional workflows.
- Outbox records.

For example, retrieving the current status of `DOC-10245` is a point lookup and is naturally suited to PostgreSQL with an appropriate index.

**StarRocks – Analytical Workload**

Used for:
- Documents by type and date.
- Processing success and failure rates.
- Historical processing duration.
- Aggregations and dashboards.

Large analytical scans should not compete with operational transactions.

If users need to see a document's authoritative live status, the application should normally read PostgreSQL. StarRocks can provide a derived analytical view, but its freshness depends on the ingestion pipeline.

### 8.2 Event History and Current State

The analytical model contains two logical datasets.

**Document Event History**

Stores every lifecycle event.

Example:

| event_id | document_id | event_type |
|---|---|---|
| EVT-001 | DOC-10245 | UPLOADED |
| EVT-002 | DOC-10245 | VALIDATED |
| EVT-003 | DOC-10245 | PROCESSING |
| EVT-004 | DOC-10245 | COMPLETED |

This supports historical analysis and processing-time calculations.

**Document Current State**

Stores the latest known analytical state of each document.

Example:

| document_id | status | last_updated |
|---|---|---|
| DOC-10245 | COMPLETED | 10:35 |
| DOC-10246 | PROCESSING | 10:37 |

The current-state model reduces the need to reconstruct the latest state from all historical events for every query.

It is a derived analytical view, not a replacement for PostgreSQL.

`event_id` supports duplicate detection, while `aggregate_version` prevents older events from overwriting newer states.

### 8.3 Storage Responsibilities

| Component | Storage Purpose |
|---|---|
| PostgreSQL | Operational records and Outbox |
| MinIO | Original documents |
| Kafka | Retained event logs |
| StarRocks | Analytical tables |
| Kubernetes PV | Persistent storage for stateful services |
| NiFi | FlowFile, Content, and Provenance repositories |

### 8.4 Capacity and Backup

The initial MinIO estimate is:

- 20 GB daily.
- 7.3 TB annually.
- 51.1 TB raw storage for seven years.

Actual physical capacity must include redundancy, backups, growth, and operational headroom.

PostgreSQL needs database backup and transaction-log recovery. MinIO requires an object recovery strategy. Kafka needs appropriate replication and retention. StarRocks needs backup or a defined rebuild procedure.

Kafka retention is not a replacement for a complete backup strategy.

---

## 9. Data Quality, Data Contract, and Governance

A successful pipeline does not guarantee correct data. The DMS platform must validate records and preserve data consistency.

### 9.1 Data Contract

Each event contract should define:

- Field names and data types.
- Required and optional fields.
- Valid values and formats.
- Event types and their meaning.
- Timestamp conventions.
- Schema version.
- Compatibility rules.
- Ownership and change approval.

For example, changing `document_id` from Integer to String can break downstream consumers and must be managed as a contract change.

A versioned schema and, where appropriate, a Schema Registry can help enforce compatibility.

### 9.2 Data Quality

Consider an event with a missing `document_id` or an unsupported `new_status`.

It should not silently enter the analytical tables.

Important checks include:

| Quality Dimension | DMS Example |
|---|---|
| Completeness | Required event fields are present |
| Validity | Status belongs to the approved lifecycle |
| Consistency | Event history agrees with source state |
| Uniqueness | Duplicate event IDs are detected |
| Timeliness | Data arrives within the freshness target |

Invalid records should be routed to a controlled Quarantine or Dead Letter path, with the error reason and original event preserved.

Transient failures should use controlled retries; invalid data should not retry indefinitely.

### 9.3 Idempotency and Reconciliation

Kafka or CDC processing may result in an event being delivered more than once.

The processing model should use `event_id` to identify duplicates and `aggregate_version` to manage logical event ordering.

Reconciliation compares records across:

- PostgreSQL.
- MinIO.
- Kafka.
- StarRocks.

For example, a document marked `UPLOADED` in PostgreSQL but missing from MinIO must be identified and investigated.

### 9.4 Data Lineage

Each analytical record should be traceable through identifiers such as:

- Source system.
- Document ID.
- Event ID.
- Object reference.
- Schema version.
- Kafka topic, partition, and offset.
- Processing timestamp.
- Target table and load status.

This makes missing, delayed, or incorrect records easier to investigate.

### 9.5 Governance and Security

DMS documents may contain sensitive customer information.

The design should include:

- Data classification.
- Role-Based Access Control (RBAC).
- Encryption in transit and at rest.
- Secure Secret Management.
- Audit logging.
- Retention and deletion policies.
- Least-privilege access.

For example, a BI Analyst may need access to document statistics and status, but not necessarily access to original identity documents or sensitive customer fields.

Retention and deletion policies must also define what happens to historical events and analytical records when an original document is deleted.

---

## 10. Kubernetes / OpenShift Deployment Design

The platform team proposes using the existing Kubernetes cluster, but not every service must necessarily run inside it.

A hybrid deployment may be preferable depending on infrastructure capacity and operational expertise.

| Component | Deployment Considerations |
|---|---|
| Source Application | Existing application environment or Kubernetes |
| PostgreSQL | Managed database or stateful deployment |
| MinIO | Object storage service with durable storage |
| NiFi | Cluster deployment and persistent repositories |
| Kafka | Stateful cluster or managed service |
| Debezium | Kafka Connect workers |
| StarRocks | FE/BE deployment and persistent storage |
| Prometheus | Metrics collection and storage |
| Grafana | Dashboards and alert visualization |

### 10.1 Stateful and Stateless Workloads

PostgreSQL, MinIO, Kafka, and StarRocks require durable data storage.

NiFi also requires appropriate repository persistence and cluster coordination.

Debezium and Kafka Connect workers are generally replaceable when their connector state and offsets are retained in Kafka.

For Kubernetes, the design should account for:

- StatefulSets and suitable Deployment resources.
- Persistent Volume Claims and StorageClasses.
- CPU and Memory Requests/Limits.
- Network Policies and DNS.
- ConfigMaps and Secrets.
- RBAC and Service Accounts.
- Readiness, Liveness, and Startup Probes.
- Pod placement and failure-domain awareness.

A Pod being Running does not prove that its service is ready to process production traffic.

### 10.2 Persistent Storage and High Availability

For example, three Kafka brokers placed on the same worker node do not provide meaningful protection against that node failing.

Pod distribution, storage durability, replication, and recovery must be considered together.

Persistent Volumes are not backups. A separate backup and restore strategy is required.

---

## 11. CI/CD and Release Planning

The DMS project produces more than application code.

It also contains database schemas, NiFi flows, Debezium connector configurations, Kafka topics, StarRocks tables, and Kubernetes deployment definitions.

These artifacts should be version-controlled and released through controlled pipelines.

### 11.1 CI/CD Process

A typical pipeline includes:

1. Source control and code review.
2. Build and artifact creation.
3. Configuration and manifest validation.
4. Unit, integration, and contract testing.
5. Security and dependency scanning.
6. Deployment to test environments.
7. End-to-end and performance testing.
8. Production approval.
9. Controlled release.
10. Post-release validation and monitoring.

### 11.2 Environment Promotion

The same tested artifact should be promoted across Development, Testing, Staging, and Production.

Environment-specific values, such as Kafka addresses and database credentials, should be configured separately.

This avoids introducing untested application or flow versions during Production deployment.

### 11.3 Release Dependencies and Rollback

For example, changing the PostgreSQL Outbox schema may affect Debezium, Kafka, and downstream consumers.

The release sequence must consider compatibility:

- Prepare and validate the database change.
- Confirm CDC compatibility.
- Prepare compatible consumers.
- Deploy the producer change.
- Validate events and analytical loading.
- Remove old fields only after migration is complete.

Rollback must consider both software and data. Reverting an application version does not automatically reverse messages already published or records already loaded into StarRocks.

Production releases should include a clear owner, approval, deployment sequence, rollback procedure, and post-release monitoring.

---

## 12. Performance and Capacity Planning

Capacity planning must cover the complete pipeline rather than only Kubernetes resources.

### 12.1 Initial Workload Calculations

For 20,000 documents per day:

- Average rate: approximately 0.23 documents/second.
- Peak rate: 10 documents/second.
- Daily raw document storage: approximately 20 GB.
- Seven-year raw document storage: approximately 51.1 TB.

Assuming an average file size of 1 MB, the peak raw file transfer rate is approximately 10 MB/second, or 80 Mbps, before network overhead.

The peak duration and frequency are still required to complete the design.

### 12.2 Capacity by Component

| Component | Main Capacity Factors |
|---|---|
| MinIO | File size, daily volume, retention, redundancy |
| PostgreSQL | Transactions, metadata size, indexes, Outbox and WAL |
| NiFi | Concurrent FlowFiles, processing time, repository I/O |
| Kafka | Event size, messages/second, partitions, retention |
| StarRocks | Event volume, table growth, analytical queries |
| Network | File transfer rate, event traffic, latency |
| Kubernetes | CPU, memory, disk I/O, available nodes |

For Kafka, capacity must be based on event size and event rate, not the original PDF size.

### 12.3 Bottleneck Example

Assume Kafka receives 2,000 events/second while StarRocks can load only 1,200 events/second.

The difference accumulates as Consumer Lag.

The team must investigate partition parallelism, load-job performance, database resources, network latency, and processing overhead.

Adding consumers alone will not solve a bottleneck if the downstream database is already saturated.

### 12.4 Performance Validation

The project should include:

- Baseline Testing.
- Load Testing.
- Stress Testing.
- Spike Testing.
- Soak Testing.
- Recovery and Replay Testing.

The main acceptance condition is that the platform supports the agreed workload while meeting its Data Freshness, availability, and data integrity requirements.

Capacity estimates must be validated through actual test results rather than treated as final resource allocations.

---

## 13. Monitoring and Operational Readiness

Monitoring must prove both that the infrastructure is healthy and that the data is moving correctly.

### 13.1 Component Monitoring

| Component | Important Metrics |
|---|---|
| PostgreSQL | Connections, query latency, WAL growth, replication status |
| MinIO | Storage capacity, request errors, upload latency |
| NiFi | FlowFiles In/Out, queue size, processing time, failed FlowFiles |
| Debezium | Connector status, CDC lag, errors |
| Kafka | Throughput, Consumer Lag, offline partitions, disk usage |
| StarRocks | Query latency, load errors, rejected rows, FE/BE health |
| Kubernetes | CPU, memory, pod restarts, volume usage, node health |

### 13.2 End-to-End Monitoring

For each document, the platform should be able to measure:

`Source Commit Timestamp → StarRocks Queryable Timestamp`

This provides the Data Freshness metric.

Additional monitoring should include:

- Source-to-target reconciliation.
- Valid and rejected event counts.
- Duplicate event counts.
- Pipeline success rate.
- Analytical load delay.
- Backlog and recovery progress.

### 13.3 Alerting and Incident Response

An alert should identify the affected component, the observed condition, and the responsible team.

For example, a continuously growing Kafka Consumer Lag should trigger investigation of consumer capacity and StarRocks loading.

A temporarily increased queue may not be an incident, but sustained growth can threaten storage capacity and freshness.

The Operations model must define alert routing, severity, escalation, on-call ownership, and operational runbooks.

---

## 14. Technical Risk Assessment and Recovery

The project must evaluate realistic failure scenarios and define prevention, detection, and recovery actions.

| Failure Scenario | Impact | Recovery Approach |
|---|---|---|
| MinIO unavailable | New documents cannot be stored | Restore storage and reconcile pending uploads |
| Debezium stopped | Lifecycle events are delayed | Restore connector and process retained CDC changes |
| Kafka unavailable | Events cannot be published | Buffer at source/ingestion layer within defined limits and resume publishing |
| StarRocks unavailable | Analytical data becomes stale | Restore service and resume or replay retained events |
| NiFi repository full | Ingestion may stop | Resolve bottleneck and restore processing |
| Schema incompatibility | Events fail or are misinterpreted | Deploy compatible versions and replay affected events |
| Duplicate events | Incorrect analytical results | Idempotent processing and reconciliation |
| Storage volume failure | Data or service unavailability | Restore from protected storage and backups |
| Credential expiration | Integration failure | Renew credentials and restore connectivity |
| Unauthorized access | Data exposure | Revoke access, investigate, and apply security remediation |

### 14.1 Recovery Requirements

For critical scenarios, the project must define:

- Detection and alerting.
- Component and business impact.
- Data retention during the outage.
- Recovery owner.
- Event replay and reconciliation procedure.
- RPO and RTO.
- Recovery validation.

For example, StarRocks may become unavailable while Kafka continues retaining events. After StarRocks is restored, the loading process resumes and the team validates that no records are missing or incorrectly duplicated.

Kafka retention must be sufficient for the expected outage and backlog recovery period.

### 14.2 Risk Prioritization

A risk register should contain:

- Probability.
- Business and technical impact.
- Priority.
- Mitigation.
- Recovery action.
- Owner.
- Status.

A critical data recovery test that has never been performed is a readiness gap, not merely a documentation issue.

---

## 15. Architecture Decisions and Trade-offs

Important decisions must be documented along with the alternatives and consequences.

The project has several major decisions:

| Decision | Selected Approach | Main Trade-off |
|---|---|---|
| OLTP / OLAP | PostgreSQL + StarRocks | Additional ingestion complexity |
| Lifecycle Events | Transactional Outbox + CDC | Additional CDC operations |
| Event Buffer | Kafka | Storage and operational overhead |
| Original Files | MinIO | Independent backup and reconciliation |
| Analytical Model | Event History + Current State | More complex consistency management |
| Deployment | Hybrid model under evaluation | More deployment and connectivity complexity |

### Example: Database Polling vs. CDC

Polling PostgreSQL may be simpler to start with, but can introduce query overhead and detection delay.

CDC provides continuous change capture but requires management of replication slots, WAL, connector offsets, and recovery.

The decision should depend on throughput, latency, operational capability, and the complexity the organization is willing to manage.

### ADR Requirements

Each important Architecture Decision Record should include:

- Context and problem.
- Options considered.
- Selected decision.
- Reason for selection.
- Consequences and risks.
- Owner and approval.
- Status.

The TPM ensures decisions and their effects on scope, cost, dependencies, and delivery commitments are recorded.

---

## 16. Production Readiness and Go-Live Decision

The final review determines whether the DMS Data Platform can safely enter production.

### 16.1 Readiness Review

| Area | Required Evidence |
|---|---|
| Architecture | Approved Architecture and Data Flow Diagrams |
| Data | Contract Tests, Quality Results, Reconciliation |
| Infrastructure | Capacity, Connectivity, Security Approval |
| Release | CI/CD, Integration and Rollback Test Results |
| Operations | Dashboards, Alerts, Runbooks, Recovery Tests |

A completed deployment alone is not evidence of Production Readiness.

For example, a MinIO cluster may be running, but if its backup cannot be restored successfully, data recovery is not proven.

### 16.2 Readiness Gaps and Corrective Actions

Consider the following final review results:

| Gap | Required Action | Owner | Priority |
|---|---|---|---|
| MinIO Restore not tested | Execute and verify restore test | Storage Team | P0 |
| Database Rollback not tested | Validate Migration and Rollback | Database Team | P1 |
| BI access exceeds requirements | Apply least-privilege RBAC and retest | Security / BI | P0 |

P0 issues must be resolved before production. P1 issues require an approved corrective plan and an explicit decision based on their residual risk.

### 16.3 Go-Live Recommendation

The TPM consolidates the evidence and recommends:

- **Go:** Critical requirements are met and the platform can enter production.
- **Conditional Go:** Remaining gaps have acceptable, explicitly approved residual risks, owners, and deadlines.
- **No-Go:** Critical technical, security, recovery, or operational requirements remain unresolved.

For the example above, the initial recommendation is No-Go because document recovery has not been proven and unauthorized access remains possible.

---

## 17. Final Project Deliverables

The project concludes with the following artifacts, produced and updated throughout the architecture review and readiness process.

1. **End-to-End Data Platform Architecture Diagram** – Components, integrations, and communication paths.
2. **Data Flow Diagram** – Source-to-target data movement, persistence, and processing.
3. **Component and Responsibility Matrix** – Component roles and team ownership.
4. **Data Quality and Governance Checklist** – Data Contracts, validation, lineage, security, and retention.
5. **Deployment and Infrastructure Dependency Map** – Storage, networking, resources, and team dependencies.
6. **Monitoring and Reliability Checklist** – Metrics, alerts, dashboards, and recovery requirements.
7. **Technical Risk Register** – Prioritized risks, mitigation, recovery, and owners.
8. **Architecture Decision and Trade-off Summary** – Major decisions and alternatives.
9. **Final Technical Readiness Report** – Project status, readiness gaps, corrective actions, and Go/No-Go recommendation.
10. **Executive Presentation** – Architecture, major risks, readiness status, and recommendations.

