# Topic 5: Data Quality, Data Governance & Data Lineage

## 1. Objective

Understand how Data Quality, Data Governance, Data Ownership, and Data Lineage contribute to reliable and maintainable Data Platforms.

The focus is on validating data, managing data-related responsibilities, tracking data dependencies, and identifying issues across data pipelines.

## 2. Data Quality Fundamentals

Data Quality describes how well data meets defined requirements and whether it is suitable for its intended use.

Six important dimensions are:

| Dimension | Description | Practical Example |
|---|---|---|
| Completeness | Required data is present | Customer records must have a customer_id |
| Accuracy | Data correctly represents real-world information | Customer address must be correct |
| Consistency | Data has the same meaning across systems | Customer status must match between CRM and Data Warehouse |
| Uniqueness | Unintended duplicate records do not exist | A transaction_id must identify only one transaction |
| Timeliness | Data is available when required | Daily network data must arrive before the reporting deadline |
| Validity | Data follows defined formats and rules | Payment status must be an approved value |

These dimensions are related but different. For example, a transaction can have valid fields but still be duplicated or contain an incorrect amount.

Data Quality can be measured using metrics such as Completeness Rate, Duplicate Rate, Invalid Record Rate, and Data Freshness.

## 3. Data Quality Validation

Data validation checks whether incoming and processed data meets technical and business requirements.

### 3.1 Pre-Ingestion Validation

Checks performed before data enters the processing pipeline.

For a CSV file received from a CRM system, typical checks include:

- Filename and file format
- File integrity and encoding (e.g. UTF-8)
- Expected columns and schema
- Record count and file size
- Mandatory fields and data types

For example, if the expected filename is `customers_YYYYMMDD.csv` but the received file has an unexpected format, it should be rejected or quarantined.

### 3.2 Record-Level Validation

After reading the file, individual records should be checked.

Example payment data:

| transaction_id | customer_id | amount | status |
|---|---|---:|---|
| TX-1001 | CUST-10 | 150 | SUCCESS |
| TX-1002 | NULL | 200 | SUCCESS |
| TX-1003 | CUST-12 | -50 | SUCCESS |
| TX-1004 | CUST-13 | 100 | UNKNOWN |

Potential issues:
- TX-1002 has a missing customer_id.
- TX-1003 has an invalid amount, depending on payment rules.
- TX-1004 has an unsupported status.

Example SQL to identify duplicate transactions:

```sql
SELECT transaction_id, COUNT(*)
FROM staging_payments
GROUP BY transaction_id
HAVING COUNT(*) > 1;
```

### 3.3 Post-Load Validation

Checks performed after data is loaded into the target system.

Common checks include:
- Source-to-target record counts
- Duplicate and missing records
- Null values in mandatory fields
- Referential integrity
- Business rule validation

Validation failures may result in rejecting an entire file or quarantining individual records. The decision depends on the severity of the issue and the business requirements.

### 3.4 Practical File Validation Checklist

Example checklist for a pipeline receiving CSV files:

| Validation | Acceptance Condition | Failure Handling |
|---|---|---|
| Filename | Matches the agreed naming convention | Reject or quarantine file |
| File Integrity | Transfer is complete; checksum matches when available | Retry transfer or quarantine |
| Encoding | Uses the expected encoding, such as UTF-8 | Reject or investigate |
| Schema | Required columns and data types are present | Stop processing or quarantine file |
| Record Count | Within the expected range | Alert and investigate |
| Mandatory Fields | Required fields are not null or empty | Reject affected records |
| Duplicates | Business key uniqueness is respected | Deduplicate or quarantine |
| Data Types | Values match expected types | Reject invalid records |
| Business Rules | Records satisfy defined conditions | Reject or flag affected records |

File-level failures, such as an incompatible schema, may require rejecting the entire file. Record-level failures may allow valid records to continue, provided partial processing is permitted by the business requirements.

## 4. Source-to-Target Reconciliation

Reconciliation verifies that source data has been correctly processed and accounted for in the destination.

### 4.1 Record Count Reconciliation

Consider a payment file containing 100,000 records:

| Processing Result | Records |
|---|---:|
| Inserted | 95,000 |
| Updated | 3,700 |
| Duplicates ignored | 500 |
| Rejected | 600 |
| Quarantined | 200 |
| **Total** | **100,000** |

The reconciliation equation is:

`Source Records = Inserted + Updated + Duplicates + Rejected + Quarantined`

Record counts alone are not sufficient. Other checks may include:

- Total transaction amount
- Number of records by status
- Distinct transaction IDs
- Minimum and maximum timestamps
- Checksums, where applicable

For example, the source and target may have the same total number of records but different numbers of successful and failed transactions.

### 4.2 Practical Reconciliation Process

A typical reconciliation process can follow these steps:

1. Assign a unique Batch ID to the incoming file or processing run.
2. Record the source record count and relevant control totals.
3. Track the final outcome of every source record: inserted, updated, duplicate, rejected, or quarantined.
4. Calculate target record counts and control totals for the same batch.
5. Compare source and target results using the expected transformation rules.
6. Record the reconciliation status and investigate any unexplained differences.

Example reconciliation result:

| Check | Source | Target | Result |
|---|---:|---:|---|
| Total Records | 100,000 | 99,200 accepted | Explained by 800 rejected/quarantined |
| Successful Payments | 80,000 | 80,000 | Pass |
| Failed Payments | 15,000 | 14,900 | Investigate |
| Total Payment Amount | 5,000,000 | 4,990,000 | Investigate |

A reconciliation difference should not automatically be considered an error if documented transformations explain it. However, unexplained differences must be investigated.

For incremental pipelines, reconciliation should also consider batch IDs, late-arriving records, updates, and reprocessing.

## 5. Schema Validation, Schema Evolution & Data Contracts

A Schema defines the expected structure of data, including field names, data types, and mandatory fields.

**Schema Validation** checks whether incoming data follows this structure.

**Schema Evolution** describes how the structure changes over time.

Example: An API originally provides:

```json
{
  "customer_id": "CUST-1001",
  "name": "Alex"
}
```

The source team later adds a new field:

```json
{
  "customer_id": "CUST-1001",
  "name": "Alex",
  "customer_segment": "Premium"
}
```

Adding an optional field is often safe, but removing or renaming a field or changing its data type may break downstream pipelines.

| Schema Change | Potential Impact |
|---|---|
| Add optional field | Usually easier to support |
| Add mandatory field | Existing producers may not provide it |
| Remove field | Downstream transformations may fail |
| Rename field | Consumers referencing the old name may break |
| Change data type | Parsing and transformation failures |

### Data Contracts

A Data Contract is an agreement between data producers and consumers that defines how data should be structured, delivered, and used.

It typically includes:

- Schema and field definitions
- Mandatory fields and data types
- Business meaning of fields
- Data Quality requirements
- Delivery frequency and freshness
- Compatibility and versioning rules
- Data ownership

For example, a payment data contract may require `transaction_id` to be unique, `amount` to use a defined currency unit, and timestamps to use UTC.

Tools such as **Confluent Schema Registry**, **Avro**, and **JSON Schema** help manage schemas and validate compatibility.

### Schema Change and Impact Analysis

Consider a CRM team planning to rename `customer_type` to `customer_segment`.

Before deployment, the change should be reviewed across the data flow:

| Component | Potential Impact |
|---|---|
| Source API / Database | Field name changes |
| Ingestion Pipeline | Schema validation may fail |
| Kafka / Schema Registry | New schema version may be required |
| Transformation Jobs | SQL queries may reference the old field |
| Data Warehouse | Target tables or models may need updates |
| BI Reports | Dashboards may depend on the old column |
| Downstream Applications | APIs or services may expect the old field |

A safe migration process is:

1. Identify all affected producers and consumers using lineage and dependency information.
2. Notify the affected teams and agree on a change date.
3. Introduce the new field while keeping the old field temporarily, where feasible.
4. Update and test downstream consumers.
5. Deploy the producer change and verify data processing.
6. Remove the old field only after confirming that no active consumers depend on it.

This approach is commonly known as Expand-Migrate-Contract.

## 6. Data Lineage and Impact Analysis

Data Lineage shows where data originates, how it moves through the platform, what transformations are applied, and which systems consume it.

Example:

```text
Payment Gateway
      |
      v
Payment API / Events
      |
      v
Raw Data Storage
      |
      v
Transformation (Spark / dbt)
      |
      v
Data Warehouse
      |
      +------------------+
      |                  |
      v                  v
BI Reports         Fraud Detection
```

Two important levels of lineage are:

- **Dataset-Level Lineage:** Relationships between files, tables, pipelines, and reports.
- **Column-Level Lineage:** How individual fields move and change between datasets.

Example:

`payment_amount_minor → payment_amount_minor / 100 → payment_amount`

If the source changes the unit or meaning of a field, lineage helps identify affected target columns and reports.

Tools:
- OpenLineage: Standard for collecting job and dataset lineage metadata.
- DataHub: Data catalog and lineage visualization.
- Apache Atlas: Metadata governance and lineage.
- dbt: Transformation metadata and lineage for supported workflows.

## 7. Metadata, Data Catalogs & Governance

**Metadata** is information that describes other data.

Common categories include:

| Metadata Type | Example |
|---|---|
| Technical | Table names, columns, data types |
| Business | Field definitions and business meaning |
| Operational | Pipeline execution time and load status |
| Ownership | Data Owner and responsible team |
| Quality | Null rate, duplicate count, validation results |
| Lineage | Source and downstream dependencies |

A **Data Catalog** is a searchable inventory of datasets and their metadata.

It helps users discover datasets, understand field definitions, identify owners, and review data lineage and quality information.

### Data Ownership and Data Stewardship

Data Governance defines how data is managed, including responsibilities, standards, ownership, and quality requirements.

| Role | Main Responsibility |
|---|---|
| Data Owner | Accountable for data definitions, business rules, and quality requirements |
| Data Steward | Maintains definitions, business metadata, and quality coordination |
| Data Producer / Source Owner | Ensures data is correctly generated at the source |
| Data Engineer | Implements ingestion, validation, and transformation |
| Data Platform Team | Provides storage, processing, and platform reliability |
| Data Consumer | Communicates data requirements and reports issues |

Example: If customer addresses are incorrect in a CRM system, the CRM source team should correct the records, while the Data Engineering team ensures the corrections are propagated through the pipeline.

The Data Owner defines the required quality standards and is accountable for the relevant business rules.

### Practical RACI Example

RACI clarifies who performs an activity and who is accountable for its outcome.

- R = Responsible: Performs the work.
- A = Accountable: Owns the final decision or outcome.
- C = Consulted: Provides input.
- I = Informed: Receives updates.

| Activity | Data Owner | Data Steward | Source Team | Data Engineering | Platform Ops |
|---|---|---|---|---|---|
| Define business rules | A | R | C | C | I |
| Generate source data | I | C | A/R | I | I |
| Implement validation | A | C | C | R | I |
| Correct source data | A | C | R | C | I |
| Monitor pipeline | I | I | I | R | A/R |
| Investigate quality incident | A | R | R | R | C |

The exact assignment depends on the organization's operating model. The Data Owner remains accountable for business requirements, while technical teams implement and operate the controls.

## 8. Business Rules, Monitoring & Incident Management

### 8.1 Business Rules

A Business Rule defines a condition that data must satisfy according to business requirements.

Examples in a payment platform:

| Business Rule | Validation |
|---|---|
| Every transaction must have a unique ID | Unique and Not Null |
| Every transaction must reference a valid customer | Referential Integrity |
| Payment status must be an approved value | Accepted Values |
| A successful payment must have a completion timestamp | Conditional Validation |

A Business Rule Catalog should document the rule, its purpose, owner, technical implementation, and exception handling.

The Data Owner approves the business rule, while the technical team implements it.

### 8.2 Data Quality Monitoring

Data Quality must be monitored continuously.

Common monitoring metrics include:

- Completeness and Duplicate Rates
- Invalid and Rejected Record Counts
- Data Freshness
- Source-to-Target Reconciliation Differences
- Unexpected changes in record volume
- Schema Validation Failures

Monitoring can use fixed thresholds or anomaly detection.

For example, if a network monitoring platform normally receives 8–10 million events per day but suddenly receives only 2 million, the deviation should trigger an investigation.

Common tools:

| Tool | Purpose |
|---|---|
| Great Expectations | Data Quality validation |
| dbt Tests | Testing transformed data |
| Soda | Data Quality monitoring |
| Prometheus | Metrics collection |
| Grafana | Dashboards and alerting |
| Apache NiFi | Ingestion validation and error routing |

### 8.3 Data Quality Incident Management

A Data Quality Incident occurs when data does not meet agreed requirements.

Examples include missing source files, invalid schemas, duplicate records, incorrect values, and failed reconciliation.

A typical incident process includes:

1. Detection through validation or monitoring.
2. Classification based on severity and impact.
3. Investigation to identify the root cause.
4. Resolution in the source system or pipeline.
5. Reprocessing of affected data when necessary.
6. Revalidation and reconciliation before closing the incident.

For example, if a payment pipeline receives 300,000 records instead of the expected 500,000, the team should investigate whether the issue originated in the source, file transfer, ingestion, or transformation stages.

Restarting the pipeline without identifying the cause may not recover the missing records.

### 8.4 Data Quality Risk Register

A Risk Register records potential problems, their impact, and the actions required to reduce them.

Example:

| Risk | Probability | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Missing source file | Medium | High | File arrival monitoring and alerts | Source Team |
| Unexpected schema change | Medium | High | Data Contracts and compatibility checks | Data Engineering |
| Duplicate records | High | Medium | Unique keys and idempotent processing | Data Engineering |
| Incorrect business rules | Low | High | Data Owner approval and rule documentation | Data Owner |
| Missing data lineage | Medium | Medium | Automated lineage collection | Data Platform Team |
| Failed reconciliation | Medium | High | Batch-level control totals and investigation | Data Engineering |

Probability and Impact should be assessed according to the actual system and business context. The register should be reviewed when significant changes or incidents occur.

## 9. Tools and Technologies

The following tools cover different parts of Data Quality and Governance. They are complementary rather than direct replacements.

| Tool | Primary Use |
|---|---|
| Apache NiFi | Data ingestion and validation |
| Great Expectations | Data Quality validation |
| dbt | Data transformation and testing |
| Soda | Data Quality monitoring |
| Confluent Schema Registry | Schema management and compatibility |
| OpenLineage | Lineage metadata collection |
| DataHub | Data Catalog and lineage |
| Apache Atlas | Metadata governance |
| Prometheus | Metrics collection |
| Grafana | Monitoring and alerting |
