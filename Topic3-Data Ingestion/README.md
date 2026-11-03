# Topic 3: Data Pipelines & Data Integration

## 1. Overview

Data pipelines collect data from different sources, validate and transform it, and deliver reliable datasets to analytical systems.

In this topic, a Document Management System (DMS) is used as the reference scenario. The platform receives document metadata, user information, and activity events from different sources and integrates them for analytical reporting.

### Objectives

- Understand data ingestion and integration patterns.
- Compare ETL and ELT approaches.
- Understand batch and streaming processing.
- Manage full and incremental data loading.
- Design pipeline orchestration and job dependencies.
- Handle data validation, failures, and recovery.
- Understand data contracts and schema evolution.

## 2. Data Ingestion

Data ingestion is the process of collecting data from source systems and transferring it into a data platform.

Three common ingestion patterns are:

| Pattern | Description | DMS Example |
|---|---|---|
| File-Based | Receiving files through file transfer protocols | Daily CSV files containing document metadata through SFTP |
| API-Based | Retrieving data through HTTP APIs | Synchronizing user information through a REST API |
| Message-Based | Receiving events through a message broker | Collecting document activity events through Apache Kafka |

### Important Considerations

- Verify file completeness before processing.
- Use checksums or file identifiers to detect duplicate files.
- Handle API authentication, pagination, rate limits, and timeouts.
- Manage duplicate, delayed, or invalid events.
- Validate incoming data before passing it to downstream stages.

SFTP provides secure file transfer, while REST APIs support structured data exchange. Neither automatically guarantees data completeness or correctness.

## 3. ETL vs. ELT

Both ETL and ELT extract data from source systems, transform it, and make it available for analytical use. Their main difference is when transformation happens.

| Aspect | ETL | ELT |
|---|---|---|
| Process | Extract → Transform → Load | Extract → Load → Transform |
| Transformation | Before loading | After loading |
| Raw Data Retention | Not mandatory | Commonly retained |
| Typical Use | Controlled data preparation | Large-scale analytical platforms |

**DMS Example:**

- ETL: Validate and transform document metadata before loading it into the Data Warehouse.
- ELT: Store original files and API responses in the Data Lake, then transform them for analytical use.

### Design Decision

An ELT-oriented approach is selected for the DMS platform because retaining raw data supports auditing, future transformations, and historical reprocessing.

Basic ingestion validation is still required to detect incomplete files and invalid input structures.

## 4. Processing and Data Loading

### 4.1. Batch vs. Streaming

| Aspect | Batch | Streaming |
|---|---|---|
| Processing | Periodic | Continuous |
| Latency | Higher | Lower |
| Typical Use | Scheduled reports and synchronization | Real-time event collection |

**DMS Design:**

- Document metadata files are processed through scheduled batch jobs.
- User information is synchronized periodically through REST APIs.
- Activity events are consumed continuously through Kafka.

This hybrid approach supports both analytical reporting and near-real-time activity monitoring.

### 4.2. Full Load vs. Incremental Load

**Full Load** transfers all records within the selected scope during each execution.

**Incremental Load** transfers only new or modified records since the previous successful execution.

Common incremental loading methods include:

- Timestamp-based extraction
- Change Data Capture (CDC)
- Sequence IDs
- Event offsets

For example, using `Created_At` alone is insufficient to detect changes to existing documents. `Updated_At` or another change tracking mechanism is needed.

```sql
SELECT *
FROM Documents
WHERE Updated_At > :last_successful_run;
```

A common strategy is to perform an initial Full Load and use Incremental Load for subsequent executions.

Additional considerations include:

- Watermark: Tracks the latest successfully processed position.
- Upsert: Inserts new records and updates existing records.
- Overlap Window: Retrieves a small previous time range to account for delayed data.
- Delete Handling: Ensures deleted records are reflected in the target dataset.

## 5. Data Pipeline Architecture

The proposed architecture integrates three data sources and delivers prepared datasets to the Data Warehouse.

### 5.1. High-Level Architecture

<img width="800" height="400" alt="data pipeline architecture" src="https://github.com/user-attachments/assets/509b68cb-14a2-4eae-9770-1ae0a7c5032d" />

### 5.2. Pipeline Stages and Responsibilities

| Stage | Responsibility |
|---|---|
| Ingestion | Collect data from SFTP, REST API, and Kafka |
| Raw Storage | Preserve original source data in the Data Lake |
| Validation | Check schema, required fields, data types, and duplicates |
| Quarantine | Isolate invalid records for investigation |
| Transformation | Clean, normalize, deduplicate, and enrich data |
| Curated Storage | Store validated and prepared datasets |
| Data Warehouse | Store structured analytical data |
| Reporting | Provide analytical datasets and business reports |

## 6. Orchestration and Job Dependencies

Orchestration coordinates pipeline tasks, manages their execution order, and handles dependencies and failures.

Scheduling defines when jobs should run, while orchestration manages how they work together.

Common responsibilities include:

- Scheduling pipeline execution.
- Managing task dependencies.
- Tracking execution status.
- Handling retries and failures.
- Triggering alerts and recovery operations.

Tools such as Apache Airflow can be used to manage scheduled workflows and dependencies.

For the DMS scenario, Airflow can coordinate batch ingestion, validation, transformation, and Data Warehouse loading. Kafka consumers should run continuously as independent services rather than depending on Airflow scheduling.

### Dependency Matrix

| Job | Dependency | Can Run Independently? |
|---|---|---|
| SFTP Document Ingestion | SFTP availability | Yes |
| REST API User Ingestion | User API availability | Yes |
| Kafka Event Consumption | Kafka availability | Yes |
| Data Validation | Successful ingestion | No |
| Data Integration and Enrichment | Required document and user data | No |
| Data Warehouse Loading | Successful transformation | No |
| Reporting | Required analytical tables | No |

**Scheduling Consideration:**

If the daily SFTP file is delayed, a report scheduled for 06:00 may miss its SLA.

Possible actions include setting ingestion deadlines, sending alerts, continuing independent jobs, and generating a provisional report if incomplete data is acceptable.

Activity events should still be collected even if user information is temporarily unavailable. User enrichment can happen later when the required data becomes available.

## 7. Data Reliability and Failure Handling

Reliable pipelines must handle temporary failures, invalid records, and incomplete processing without losing data or creating inconsistent results.

### 7.1. Key Reliability Concepts

| Concept | Description | DMS Example |
|---|---|---|
| Data Validation | Checking data quality and structure | Checking required document fields |
| Retry | Repeating a failed operation | Retrying a REST API request after a timeout |
| Idempotency | Preventing duplicate results from repeated execution | Processing the same CSV file without creating duplicate records |
| Replay | Processing previously received events again | Replaying Kafka events |
| Reprocessing | Running historical data through updated processing logic | Recalculating activity metrics after fixing a transformation |
| Backfill | Processing missing data for a historical period | Recovering two days of missing activity events |

### 7.2. Failure Handling Scenarios

| Scenario | Handling Strategy |
|---|---|
| Duplicate SFTP file | Detect using file identifiers or checksums and prevent duplicate processing |
| Incomplete CSV file | Verify completion before processing |
| Malformed records | Isolate invalid records in quarantine |
| REST API unavailable | Retry with exponential backoff |
| Duplicate Kafka event | Use event IDs and idempotent processing |
| Invalid Kafka event | Send to a Dead Letter Queue (DLQ) |
| Data Warehouse unavailable | Retain processed data and retry loading |
| Missing historical data | Identify the missing period and execute backfill |
| Transformation logic error | Correct the logic and reprocess affected data |

### 7.3. Recovery Considerations

A pipeline should not restart from the beginning whenever a stage fails.

For example, if data has already been stored in the Raw Data Lake but Data Warehouse loading fails, the pipeline should retry the loading stage rather than retrieving all source data again.

This requires:

- Persistent intermediate data.
- Execution status tracking.
- Idempotent loading operations.
- Stage-level error handling.

Retry should generally be used for temporary failures. Permanent errors, such as invalid schemas or missing required fields, should be isolated and investigated instead of being retried indefinitely.

## 8. Data Contracts and Schema Evolution

A Data Contract defines the structure and expectations of data exchanged between producers and consumers.

It typically includes:

- Field names and data types.
- Required and optional fields.
- Accepted values and validation rules.
- Schema version.
- Compatibility requirements.

### 8.1. Schema Evolution

Schema evolution is the process of managing data structure changes without unexpectedly breaking dependent systems.

| Schema Change | Risk | Reason |
|---|---|---|
| Adding an optional field | Usually Low | Existing consumers may continue working |
| Renaming `owner_id` to `user_id` | High | Existing consumers may fail |
| Changing `file_size` from Integer to String | High | Data type expectations change |
| Removing a required field | High | Downstream dependencies may break |

### 8.2. Compatibility

- **Backward Compatibility:** A newer schema remains compatible with older consumers.
- **Forward Compatibility:** An older consumer can process newer data, including additional fields it does not recognize.

### 8.3. Best Practices

- Use schema versioning.
- Coordinate changes between producers and consumers.
- Test schema changes before deployment.
- Avoid changing existing field names and types without a migration plan.
- Introduce replacements before removing old fields.
- Define a deprecation period for breaking changes.

## 9. ETL/ELT Design Decision Summary

The proposed DMS platform uses an ELT-oriented architecture with initial ingestion validation.

The main processing flow is:

1. Extract document files, user information, and activity events.
2. Validate incoming data and check file completeness.
3. Store original data in the Raw Data Lake.
4. Isolate invalid records in quarantine or a DLQ.
5. Transform, clean, and enrich valid data.
6. Store prepared datasets in the Curated Zone.
7. Load analytical tables into the Data Warehouse.
8. Provide data to BI and reporting systems.

This design provides flexibility for future transformations, supports historical reprocessing, and separates source ingestion from analytical processing.

Pipeline monitoring should track execution status, data freshness, processing duration, validation failures, duplicate records, and SLA compliance.

## 10. Conclusion

A reliable data pipeline must ensure more than data transfer. It should maintain data quality, consistency, availability, and recoverability.

The proposed DMS architecture combines file-based, API-based, and message-based ingestion with a Data Lake, transformation layer, Data Warehouse, and reporting tools.

Batch processing supports scheduled synchronization and reporting, while streaming ingestion enables continuous activity event collection.

Validation, idempotency, failure handling, schema management, and orchestration provide the foundation for reliable data integration.
