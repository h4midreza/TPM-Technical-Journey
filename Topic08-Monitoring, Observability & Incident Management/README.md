# Topic 8: Monitoring, Observability & Incident Management

## 1. Objective

Understand how infrastructure, applications, and data pipelines are monitored and how production issues are detected, investigated, and resolved.

The focus is on:

- Monitoring system health and performance.
- Detecting application failures and data processing delays.
- Investigating problems across dependent components.
- Coordinating technical teams during incidents.
- Evaluating service reliability and tracking corrective actions.

The goal is to understand operational behavior from a Technical Program Manager (TPM) perspective.

## 2. Practical Scenario: Monitoring a DMS Data Platform

Assume a DMS Data Platform consists of the following components:

| Component | Monitoring Focus |
|---|---|
| NiFi | FlowFile failures, queue size, processing time |
| Kafka | Broker health and Consumer Lag |
| Document Processor | Processing throughput, latency, and errors |
| PostgreSQL | Connections, query latency, and availability |
| MinIO | Storage capacity and request failures |
| ClickHouse | Insert performance and data freshness |
| Kubernetes | CPU, Memory, Pod health, and restarts |

### Practical Scenario

DMS normally processes 100,000 document events per hour.

Today, users report that recently uploaded documents are not appearing in analytical reports.

Although all services appear to be running, investigation shows that Kafka Consumer Lag is increasing and ClickHouse is receiving data with delays.

This demonstrates an important principle:

**A service can be operational while the overall data pipeline is degraded.**

Monitoring must cover Infrastructure, Applications, and Data Pipelines.

## 3. Monitoring, Observability, and Incident Management

These concepts have different but complementary responsibilities.

### Monitoring

Tracks known system conditions using metrics, dashboards, and alerts.

Example: An alert is generated when Kafka Consumer Lag remains above 10,000 messages for 10 minutes.

### Observability

Helps investigate system behavior and identify the causes of problems using Metrics, Logs, and Traces.

Example: Monitoring shows increased API latency, while tracing reveals that PostgreSQL queries are causing the delay.

### Incident Management

Coordinates the detection, investigation, recovery, and documentation of production problems.

Example: When document processing stops, the TPM coordinates the Data Platform, Database, and Infrastructure teams until service recovery.

## 4. Metrics, Logs, and Distributed Tracing

These are the three main types of telemetry used in Observability.

### Metrics

Numerical measurements collected over time.

Examples:

- CPU and Memory Usage.
- API Request Rate and Error Rate.
- Request Latency.
- Kafka Consumer Lag.
- NiFi Queue Size.

Metrics help identify abnormal behavior, analyze trends, and generate alerts.

### Logs

Timestamped records of application and infrastructure events.

Example:

```text
10:30:15 INFO  Document request received
10:30:16 INFO  Fetching metadata from PostgreSQL
10:30:21 ERROR Database connection timeout
```

Common Log Levels:

| Level | Purpose |
|---|---|
| DEBUG | Detailed execution information |
| INFO | Normal application events |
| WARN | Unexpected but potentially recoverable conditions |
| ERROR | Failed operations |
| FATAL | Critical application failure |

Logs should ideally include timestamps, service names, and identifiers such as `document_id` and `trace_id`. Sensitive information must be protected.

### Distributed Tracing

Tracks a request across multiple services.

A Trace represents the complete request journey, while a Span represents an individual operation.

Example:

```text
Client Request
      |
      v
Metadata API
      |
      v
PostgreSQL Query
      |
      v
Response
```

Suppose the request takes 2.8 seconds, of which 2.1 seconds are spent querying PostgreSQL.

Tracing helps identify the database as the main source of latency.

- **Trace ID:** Identifies the complete request.
- **Span ID:** Identifies an individual operation.

OpenTelemetry, Jaeger, and Grafana Tempo are commonly used for tracing and telemetry collection.

| Telemetry | Main Question |
|---|---|
| Metrics | What is happening? |
| Logs | What happened? |
| Traces | Where did the request slow down or fail? |

## 5. Infrastructure Monitoring and Application Monitoring

### Infrastructure Monitoring

Tracks the health and resource usage of the underlying environment.

| Metric | DMS Example |
|---|---|
| CPU Usage | CPU consumption of Document Processor Pods |
| Memory Usage | Metadata API RAM consumption |
| Disk Usage | Kafka and PostgreSQL storage capacity |
| Disk I/O | PostgreSQL read/write performance |
| Network Traffic | File transfer between NiFi and MinIO |
| Node Availability | Kubernetes Worker Node health |
| Pod Restarts | Repeated Document Processor restarts |

**Example:** If CPU usage reaches 95%, investigate incoming workload, processing time, CPU throttling, and available Worker capacity before assuming that more CPU is required.

### Application Monitoring

Tracks application behavior and service performance.

| Metric | DMS Example |
|---|---|
| Request Rate | Metadata API requests per second |
| Error Rate | Failed metadata retrieval requests |
| Response Time | Time to retrieve document metadata |
| Active Connections | PostgreSQL connections from Metadata API |
| Availability | Ability of Metadata API to serve requests |
| Processing Time | Time required to process a document |

**Example:** Metadata API is running, but P95 response time increases from 200 ms to 1,500 ms. Investigation should include Database performance, connection pools, resource usage, and recent releases.

Infrastructure Monitoring identifies resource problems, while Application Monitoring shows their effect on service behavior.

## 6. Data Observability

Data Observability focuses on the health, quality, and reliability of data throughout a pipeline.

A pipeline may be operational while delivering delayed, incomplete, or incorrect data.

### Important Indicators

| Indicator | DMS Example |
|---|---|
| Data Freshness | Time between document ingestion and availability in ClickHouse |
| Data Volume | Number of documents or events received per hour |
| Data Quality | Completeness, validity, and consistency of records |
| Schema Changes | Unexpected changes in data structure |
| Pipeline Failures | Failed ingestion or processing operations |

Data Quality includes:

- Completeness: Required fields are present.
- Validity: Values follow defined rules.
- Consistency: Related systems contain compatible information.
- Uniqueness: Duplicate records are controlled.
- Accuracy: Data correctly represents the source.

**Example:** DMS normally receives 100,000 events per hour, but today receives only 15,000. The source system, NiFi ingestion flow, and validation rules should be investigated.

### Pipeline-Specific Metrics

| Component | Important Metrics |
|---|---|
| NiFi | FlowFiles In/Out, Queue Size, Processing Time, Failed FlowFiles, Back Pressure |
| Kafka | Broker Health, Messages In/Out, Consumer Lag |
| Document Processor | Documents Processed, Processing Time, Failed Documents |
| PostgreSQL | Connections, Query Latency, Lock Waits |
| MinIO | Storage Capacity, Request Errors, Read/Write Latency |
| ClickHouse | Insert Rate, Query Latency, Data Freshness |

**Important distinction:**

- NiFi Queue Size measures FlowFiles waiting between NiFi Processors.
- Kafka Consumer Lag measures how far a Consumer is behind the latest available messages.

## 7. Prometheus and Grafana Fundamentals

### Prometheus

An open-source monitoring and alerting system that collects and stores time-series metrics.

Main responsibilities:

- Collecting metrics from applications and infrastructure.
- Storing metrics with timestamps.
- Querying metrics using PromQL.
- Evaluating alerting rules.
- Sending alerts to Alertmanager.

Prometheus commonly collects metrics through HTTP endpoints such as:

```text
http://metadata-api:8080/metrics
```

Example PromQL query for a simplified API error rate:

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

The exact metric names and labels depend on application instrumentation.

### Grafana

A visualization and dashboard platform that connects to monitoring data sources.

It is used to:

- Build monitoring dashboards.
- Visualize metrics over time.
- Compare infrastructure and application performance.
- Display logs and traces from supported sources.

### Tool Comparison

| Tool | Main Responsibility |
|---|---|
| Prometheus | Collects, stores, and queries metrics |
| Grafana | Visualizes monitoring data |
| Alertmanager | Groups and routes Prometheus alerts |

Typical flow:

```text
Applications / Infrastructure
             |
             v
         Prometheus
             |
       +-----+-----+
       |           |
       v           v
    Grafana   Alertmanager
  Dashboards       |
                   v
             Notifications
```

## 8. Centralized Logging and Log Search

Centralized Logging collects logs from different applications and infrastructure components into a common platform.

### Typical Architecture

```text
Application / Container Logs
             |
             v
        Log Collector
             |
             v
         Log Storage
             |
             v
      Search / Visualization
```

### Common Tools

| Tool | Responsibility |
|---|---|
| Fluent Bit | Collects and forwards logs |
| Logstash | Processes and transforms logs |
| Elasticsearch | Stores and searches logs |
| OpenSearch | Log search and analytics |
| Loki | Log aggregation and querying |
| Kibana | Searches Elasticsearch logs |
| Grafana | Visualizes logs from supported sources |

### Practical Example

Document Processor fails while processing `doc-123`.

The technical team searches logs using:

- Service name.
- Document ID.
- Time range.
- Error level.
- Trace ID.

The logs reveal repeated MinIO request timeouts.

This helps the team investigate MinIO availability, network latency, and retry configuration.

Log retention, access control, and protection of sensitive information must also be defined.

## 9. Health Checks and Service Readiness

A running application is not necessarily ready to receive traffic.

Kubernetes uses three main probes:

| Probe | Purpose | Failure Impact |
|---|---|---|
| Startup | Allows time for application initialization | Other probes wait until startup succeeds |
| Readiness | Checks whether the application can receive traffic | Pod is removed from ready Service endpoints |
| Liveness | Checks whether the application is functioning | Container may be restarted |

**DMS Example:**

Metadata API is running, but PostgreSQL is unavailable.

If the application cannot handle database-dependent requests, Readiness should prevent traffic from reaching that Pod.

A Readiness failure does not automatically restart the Container.

Liveness and Readiness should not necessarily use the same health-check logic. A temporary dependency failure should not automatically cause repeated application restarts.

### Service Health vs. Data Health

| Service Health | Data Health |
|---|---|
| API responds successfully | Latest data has arrived |
| Pods are running | Expected records are present |
| Database connections work | Data is complete and consistent |
| Application is available | Pipeline processes data on time |

## 10. Alerting Rules and Alert Routing

Alerting automatically notifies teams when monitoring conditions indicate a potential problem.

### Alerting Rules

Example:

```text
IF CPU Usage > 85%
FOR 5 minutes
THEN Generate Alert
```

Kafka example:

```text
IF Consumer Lag > 10,000
FOR 10 minutes
THEN Generate Alert
```

The `FOR` duration prevents alerts caused by brief fluctuations.

Thresholds should be based on normal workload patterns and operational requirements.

### Alert Routing

Alerts should be sent to the responsible team.

| Alert | Responsible Team |
|---|---|
| High CPU on Kubernetes Node | Platform |
| Metadata API Error Rate | Application |
| PostgreSQL Availability | Database |
| Kafka Consumer Lag | Data Platform |
| MinIO Storage Capacity | Storage |
| Network Packet Loss | Network |

A useful alert should include the affected component, observed condition, severity, time, impact, and investigation link.

Avoid unnecessary alerts, unclear ownership, and excessive notifications that cause Alert Fatigue.

## 11. Service Reliability and Data Pipeline Metrics

These metrics help evaluate whether services and pipelines meet their expected performance.

| Metric | Meaning | Example |
|---|---|---|
| Error Rate | Percentage of failed operations | 2,000 failures out of 100,000 requests = 2% |
| Throughput | Successful operations per unit of time | 1,200 documents per minute |
| Latency | Time required for an operation | API request takes 200 ms |
| P95 / P99 | Latency percentiles | 95% / 99% of requests complete within the measured value |
| Availability | Ability to serve eligible requests | 99.9% successful availability |
| Pipeline Success Rate | Percentage of successfully processed records | 9,800 out of 10,000 = 98% |
| Data Freshness | How recently data became available | Latest data is 5 minutes old |
| Consumer Lag | Consumer's distance from the latest Kafka messages | Lag increases over time |

**Important:**

- Average Latency can hide slow requests.
- P95 means 95% of measured requests complete within that duration or faster.
- High Consumer Lag should be investigated together with message production rate, processing rate, and event age.
- Data Freshness should use clearly defined source and destination timestamps.

## 12. SLA, SLO, and SLI

These concepts define how service reliability is measured and agreed upon.

### Service Level Indicator (SLI)

A measurable indicator of service performance.

Examples: API Availability, Error Rate, P95 Latency, and Data Freshness.

### Service Level Objective (SLO)

A target defined for an SLI over a specific measurement window.

Example:

| Indicator | Illustrative SLO |
|---|---|
| Metadata API Availability | At least 99.9% monthly |
| API P95 Latency | Below 300 ms |
| Data Freshness | Below 5 minutes |
| Pipeline Success Rate | At least 99.5% |

### Service Level Agreement (SLA)

A formal agreement between a service provider and a customer.

It may define availability commitments, support response times, and consequences for failing to meet agreed targets.

### Comparison

| Concept | Meaning |
|---|---|
| SLI | What is measured |
| SLO | What target should be achieved |
| SLA | What is formally committed |

### Error Budget

An Error Budget is the amount of unreliability allowed within an SLO measurement window.

For example, a monthly Availability SLO of 99.9% allows 0.1% unavailability.

For a 30-day month, this is approximately 43.2 minutes under an uptime-based measurement.

If the Error Budget is being consumed too quickly, teams may prioritize reliability improvements over additional releases.

## 13. Incident Severity and Escalation

An Incident is an event that causes or threatens service degradation, interruption, data loss, or failure to meet critical requirements.

### Severity Levels

Actual severity definitions should follow the organization's Incident Management policy.

| Severity | Description | DMS Example |
|---|---|---|
| SEV1 - Critical | Major outage or critical business impact | Entire DMS unavailable or large-scale data loss |
| SEV2 - High | Major functionality degraded | Document processing stopped |
| SEV3 - Medium | Limited impact or partial degradation | Increased API latency |
| SEV4 - Low | Minor issue | Non-critical dashboard warning |

Severity depends on affected users, business impact, data integrity, urgency, and available workarounds.

### Incident Lifecycle

```text
Incident Detected
       |
       v
Impact and Severity Assessment
       |
       v
Responsible Teams Engaged
       |
       v
Investigation and Mitigation
       |
       v
Recovery and Verification
       |
       v
Incident Closure
       |
       v
Root Cause Analysis
```

### TPM Responsibilities

During an incident, TPM coordinates:

- Affected components and business impact.
- Responsible teams and technical ownership.
- Dependencies and recovery blockers.
- Mitigation options and recovery progress.
- Status communication and escalation.

An ERROR log does not automatically mean SEV1. Severity is determined by actual impact.

## 14. Root Cause Analysis (RCA)

RCA identifies the underlying reason an incident occurred rather than focusing only on visible symptoms.

### Practical DMS Scenario

Kafka Consumer Lag is increasing.

Investigation reveals:

1. Kafka Brokers are available.
2. Consumer Pods are running.
3. Document Processor latency has increased.
4. PostgreSQL query latency is unusually high.
5. A recent release introduced an inefficient query.

A possible root cause is the inefficient query, which slowed down Consumer processing.

The team should confirm this using database metrics, query execution plans, application traces, and deployment history.

### RCA Techniques

- **5 Whys:** Repeatedly asks why a problem occurred to identify underlying causes.
- **Fishbone Diagram:** Organizes possible causes into categories such as People, Process, Technology, and Environment.
- **Incident Timeline:** Records alerts, changes, investigation steps, and recovery events.

RCA should be based on evidence from Metrics, Logs, Traces, Kubernetes Events, Database Monitoring, and Deployment History.

The goal is system and process improvement, not assigning personal blame.

## 15. Corrective and Preventive Actions

After identifying the root cause, teams define actions to restore service and prevent recurrence.

### Corrective Actions

Fix the existing problem.

Examples:

- Fix an inefficient database query.
- Restore failed Kafka Consumers.
- Correct database connection settings.
- Roll back a problematic release.

### Preventive Actions

Reduce the likelihood of similar incidents.

Examples:

- Add Consumer Lag alerts.
- Introduce production-scale load testing.
- Monitor database query latency.
- Improve resource monitoring.
- Add Data Freshness alerts.
- Improve deployment and rollback procedures.

### Action Tracking

| Action | Type | Owner | Expected Result |
|---|---|---|---|
| Fix inefficient query | Corrective | Database / Development | Restore processing throughput |
| Add query latency alert | Preventive | Database | Detect slow queries earlier |
| Add load testing | Preventive | Development / QA | Detect performance regressions |
| Add Consumer Lag dashboard | Preventive | Data Platform | Improve pipeline visibility |
| Review rollback criteria | Preventive | TPM / Platform / Development | Reduce recovery time |

Each action should have a clear owner, target date, expected result, and verification method.

### Incident Closure

Before closing an incident, verify:

- Service performance has returned to normal.
- Error Rate and Latency are within expected limits.
- Data processing has recovered.
- Consumer Lag and Queue Size are returning to normal.
- Data Freshness meets its target.
- No known data loss or inconsistency remains.
- Follow-up actions have assigned owners.

### TPM Operational View

During monitoring reviews and incident follow-ups, track:

| Area | Information |
|---|---|
| Incident | What happened and when? |
| Impact | Which services, users, and data flows are affected? |
| Technical Status | What is known and what is under investigation? |
| Ownership | Which teams are responsible? |
| Dependencies | What is blocking resolution? |
| Recovery | What has been fixed and what remains? |
| Business Impact | Are service availability or data delivery affected? |
| Follow-up | Which corrective and preventive actions remain open? |

**Key Principle:** Monitoring detects abnormal behavior, Observability helps explain it, and Incident Management coordinates the response and recovery. RCA and follow-up actions help prevent recurrence.
