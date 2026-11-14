# Topic 2: Database Fundamentals, OLTP, OLAP & Data Modeling

## 1. Objective

Understand relational database fundamentals, database transactions, operational and analytical workloads, and data modeling concepts.

This topic focuses on how database architecture and design decisions affect application performance, data consistency, scalability, and reliability.

A Document Management System (DMS) is used as a practical example to explore database design, storage choices, workload separation, and analytical architecture.

The goal is to understand the technical decisions, their trade-offs, and their impact on a Data Platform from a Technical Program Manager (TPM) perspective.

## 2. Scenario: Document Management System (DMS)

An organization operates a DMS that allows users to upload, organize, search, access, and manage documents.

### Core Capabilities

- User and role management
- Document and folder management
- Document upload and download
- Document versioning
- Access control and permissions
- Document search and retrieval
- User activity tracking through audit logs
- Operational reporting and historical analysis

### Workload Characteristics

The DMS supports two major workload types:

**Operational Workload**

Handles daily application operations, including:

- Uploading documents
- Updating document metadata
- Creating folders
- Changing access permissions
- Registering new document versions
- Searching for individual documents

These operations usually require fast responses and reliable transaction processing.

**Analytical Workload**

Supports reporting and analysis, including:

- Monthly document upload statistics
- User activity trends
- Document distribution by category
- Storage consumption analysis
- Historical usage reports
- Identification of the most active users

These operations may scan large datasets and perform aggregations across historical records.

The two workloads have different performance and database design requirements.

## 3. Relational Database Fundamentals

A relational database organizes structured data into tables connected through defined relationships.

### Core Concepts

- **Table:** Stores data related to a specific entity, such as Users or Documents.
- **Row:** Represents an individual record.
- **Column:** Represents an attribute of an entity.
- **Schema:** Defines the database structure, including tables, columns, data types, constraints, and relationships.
- **Primary Key (PK):** Uniquely identifies each record in a table.
- **Foreign Key (FK):** References a record in another table and helps maintain referential integrity.
- **Constraint:** Enforces rules on stored data, such as NOT NULL, UNIQUE, and CHECK.

### Main Operational Tables

| Table | Purpose |
|---|---|
| Users | Stores user information |
| Folders | Manages folder structures |
| Documents | Stores document metadata |
| Document_Versions | Tracks document versions |
| Permissions | Manages document access rights |
| Audit_Logs | Records user activities |

### Example Table Structure

A simplified DMS database could use the following structure:

**Users**

```text
user_id (PK)
name
email
department
created_at
```

**Documents**

```text
document_id (PK)
created_by (FK)
owner_id (FK)
folder_id (FK)
title
status
created_at
updated_at
```

**Document_Versions**

```text
version_id (PK)
document_id (FK)
version_number
object_key
file_size
created_at
```

**Permissions**

```text
user_id (FK)
document_id (FK)
permission_type
created_at
```

The Permissions table connects Users and Documents and manages their access relationships.

### Database Relationships

- Users to Documents: One-to-Many, based on document creation or ownership.
- Folders to Documents: One-to-Many.
- Documents to Document_Versions: One-to-Many.
- Users to Documents through Permissions: Many-to-Many.

For example, one user can create multiple documents, while one document can have multiple versions and can be shared with multiple users.

### Where Are Document Files Stored?

Document files such as PDF, Word, and images are commonly stored in Object Storage rather than directly inside relational database tables.

The database stores metadata and a reference to the actual file.

Example:

```text
PostgreSQL:
  document_id = 101
  file_name = contract.pdf
  object_key = documents/2026/contract-101.pdf
  file_size = 5 MB

MinIO:
  documents/2026/contract-101.pdf
  [Actual PDF Content]
```

This approach separates document metadata and transactional operations from large file storage.

### Database Transactions and ACID

A database transaction is a group of operations treated as one logical unit.

For example, when a new document is registered, the application may need to:

1. Insert a record into Documents.
2. Create the initial record in Document_Versions.
3. Register associated metadata.

These database operations should succeed together or fail together.

The ACID properties define important transaction guarantees:

| Property | Meaning | DMS Example |
|---|---|---|
| Atomicity | All operations succeed or none do | Document and initial version are registered together |
| Consistency | Database rules remain valid | A document cannot reference a non-existent user |
| Isolation | Concurrent transactions do not create invalid interference | Two users updating metadata do not produce inconsistent results |
| Durability | Committed changes survive failures | A successfully committed document record remains stored after a restart |

**Practical consideration:**

ACID transactions apply within the database transaction boundary. They do not automatically make operations across PostgreSQL, MinIO, and Kafka atomic.

For example, a file may be uploaded successfully to MinIO while the PostgreSQL transaction fails. Such cross-system consistency requires additional recovery mechanisms, reconciliation, or patterns such as Transactional Outbox.

## 4. OLTP vs. OLAP

OLTP and OLAP describe two different workload patterns and their processing requirements.

They are not necessarily two different database products.

### OLTP (Online Transaction Processing)

OLTP systems handle day-to-day application transactions and support frequent reads and writes.

Typical characteristics:

- Frequent INSERT, UPDATE, and DELETE operations
- Fast response for individual transactions
- Structured and relational data
- Transaction consistency
- High concurrency from application users

DMS examples:

- Uploading a document
- Updating document metadata
- Changing document permissions
- Creating folders
- Registering a new document version

**Example Query:**

Find approved documents belonging to a specific user.

```sql
SELECT document_id, title, status
FROM Documents
WHERE owner_id = 125
  AND status = 'Approved';
```

A typical technology for this workload is PostgreSQL.

### OLAP (Online Analytical Processing)

OLAP systems support analytical queries, reporting, and aggregations across large datasets.

Typical characteristics:

- Large-scale data scans
- Aggregations and complex queries
- Historical data analysis
- Read-intensive workloads
- Queries across multiple tables or datasets

DMS examples:

- Monthly document upload statistics
- User activity analysis
- Document distribution by category
- Storage consumption reports
- Most active users over a selected period

**Example Query:**

Calculate the number of documents created each month.

```sql
SELECT
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS document_count
FROM Documents
GROUP BY 1
ORDER BY 1;
```

Typical technologies include ClickHouse, Snowflake, BigQuery, and Amazon Redshift.

### OLTP vs. OLAP Comparison

| Characteristic | OLTP | OLAP |
|---|---|---|
| Primary Purpose | Daily operations | Data analysis and reporting |
| Workload | Transactional | Analytical |
| Operations | Frequent reads and writes | Mostly read-intensive |
| Query Complexity | Usually simple | Often complex |
| Data Scope | Current operational data | Historical and aggregated data |
| Response Requirement | Fast transaction response | Efficient analytical processing |
| Typical Users | Application users | Analysts and managers |
| DMS Example | Uploading a document | Monthly upload reports |
| Typical Database | PostgreSQL | ClickHouse |

### Why Separate OLTP and OLAP Workloads?

Running complex analytical queries directly against an operational database can consume significant CPU, memory, disk I/O, and database connections.

This may increase transaction latency and affect the DMS application.

A common architecture separates these workloads:

- PostgreSQL handles operational transactions.
- Data is transferred to an analytical database.
- BI and reporting systems query the analytical database instead of the operational database.

**Important:** Separation does not always require two different database products, but dedicated analytical infrastructure is common when workloads and data volumes grow.

## 5. Row-Oriented vs. Column-Oriented Storage

The way a database organizes data internally affects how efficiently it can read, write, and process information.

### Row-Oriented Storage

Data is organized by records, with the attributes of each row stored together.

Example:

```text
Document 1:
101 | Contract.pdf | Approved | 5 MB

Document 2:
102 | Invoice.pdf  | Pending  | 2 MB

Document 3:
103 | Report.pdf   | Approved | 8 MB
```

It is generally suitable for:

- Transaction processing
- Inserting and updating individual records
- Retrieving complete records
- Operational database workloads

Typical example: PostgreSQL.

### Column-Oriented Storage

Data is organized by columns, allowing analytical queries to read only the required attributes.

Conceptually:

```text
document_id:
101, 102, 103

title:
Contract.pdf, Invoice.pdf, Report.pdf

status:
Approved, Pending, Approved

file_size:
5 MB, 2 MB, 8 MB
```

It is generally suitable for:

- Large-scale analytical queries
- Aggregations and statistical calculations
- Reporting across large datasets
- Data Warehouse workloads

Typical example: ClickHouse.

### Comparison

| Characteristic | Row-Oriented | Column-Oriented |
|---|---|---|
| Storage Organization | By rows | By columns |
| Best Fit | Transactional workloads | Analytical workloads |
| Individual Record Access | Generally efficient | Often less efficient |
| Large-Scale Aggregations | May require reading many fields | Can read only required columns |
| Common Usage | PostgreSQL | ClickHouse |

**Practical Example:**

Suppose the DMS contains 50 million document records.

A report needs only the total file size of all documents.

A column-oriented database can read the file_size column without reading unrelated fields such as title, owner_id, or status.

Columnar storage also commonly benefits from compression because values within a column often have similar data types and patterns.

The actual performance depends on the database engine, indexes, compression, query patterns, and workload.

Row-oriented and column-oriented storage are common choices for OLTP and OLAP respectively, but the two concepts are not strict equivalents.

## 6. Data Modeling

Data modeling defines how data is structured, organized, and connected to support application operations and analytical requirements.

### Normalization

Normalization organizes data into related tables to reduce unnecessary duplication and maintain data consistency.

Example:

Instead of storing user information repeatedly in every document record, user details are stored in the Users table and referenced through User_ID.

Benefits:

- Reduces data duplication.
- Improves data consistency.
- Makes updates easier to manage.
- Helps maintain referential integrity.

Normalization is generally preferred in operational databases.

### Denormalization

Denormalization introduces controlled data duplication to improve query performance and simplify analytical reporting.

For example, an analytical dataset may store user department and document category alongside document activity records.

This can reduce repeated joins when generating reports.

Trade-offs:

- Faster analytical queries in some workloads.
- More storage usage.
- Duplicated values must be updated or refreshed consistently.

Denormalization is commonly used in analytical environments.

### Fact Tables and Dimension Tables

A common Data Warehouse modeling approach separates measurable events from descriptive information.

**Fact Table**

Stores measurable business events and activities.

Example: `Fact_Document_Activity`

```text
activity_id
user_id
document_id
date_id
activity_type
file_size
```

**Dimension Tables**

Store descriptive information used to categorize and analyze facts.

Examples:

- `Dim_User`: User name, department, and role.
- `Dim_Document`: Document type and category.
- `Dim_Date`: Day, month, quarter, and year.

### Fact Table Grain

Grain defines exactly what one row in a Fact Table represents.

For example:

One row in `Fact_Document_Activity` represents one activity performed on one document.

If a document is viewed five times, the table may contain five activity records.

This definition matters because incorrect aggregation can produce misleading results.

For example, summing document file sizes across all activity events could count the same document multiple times. The analytical model must define which events represent a new document or version and which are simply views or downloads.

### Star Schema

Star Schema is a common Data Warehouse modeling approach in which a central Fact Table connects to multiple Dimension Tables.

```text
                 Dim_User
                    |
                    |
Dim_Date ---- Fact_Document_Activity ---- Dim_Document
```

It supports analytical queries such as:

- Number of uploaded documents per month.
- User activity by department.
- Storage consumption by document category.
- Most active users over a selected period.

The Star Schema simplifies many reporting queries and provides a consistent analytical structure.

## 7. Database Performance and Architecture

Several techniques help improve database performance and support growing workloads.

### Indexing

Indexes help the database locate records without scanning the entire table.

DMS example:

If users frequently search for documents by owner and status, an index on the relevant columns may improve retrieval performance.

Example:

```sql
CREATE INDEX idx_documents_owner_status
ON Documents (owner_id, status);
```

A composite index can support queries that filter by multiple columns.

However, column order matters and should match actual query patterns.

**Trade-offs:**

- Faster retrieval for supported queries.
- Additional storage consumption.
- More work during INSERT, UPDATE, and DELETE operations.

Indexes should be designed based on actual workload and query execution plans rather than created on every column.

### Query Execution Plan

A database query may be slow even when appropriate indexes exist.

A Query Execution Plan shows how the database intends to retrieve and process the requested data.

In PostgreSQL, `EXPLAIN ANALYZE` can be used to inspect actual query execution behavior.

It helps identify:

- Full table scans.
- Index usage.
- Expensive joins.
- Sorting operations.
- High-cost execution steps.

This is useful when a frequently used DMS search query becomes slower as the database grows.

### Partitioning

Partitioning divides a large table into smaller logical sections based on a selected key.

DMS example:

The Audit_Logs table can be partitioned by month:

```text
Audit_Logs
 ├── audit_logs_2026_09
 ├── audit_logs_2026_10
 └── audit_logs_2026_11
```

Benefits:

- Better management of large tables.
- More efficient queries that target specific partitions.
- Easier archival or removal of old data.
- Improved operational management of historical data.

Partitioning is not automatically faster for every query. It is most useful when the table is large and queries can benefit from partition pruning.

### Replication

Replication maintains copies of database data across multiple database servers.

A common architecture includes:

- **Primary Database:** Handles write operations.
- **Read Replicas:** Serve selected read requests and distribute read workload.

DMS example:

The primary PostgreSQL database handles document uploads and metadata changes, while reporting or read-heavy application requests may use read replicas when appropriate.

Important considerations:

- Replication lag may cause a replica to return stale data.
- Read-after-write requirements may require reading from the primary.
- Failover requires proper configuration and operational procedures.
- Replication is not a replacement for independent backups.

### Performance Techniques Comparison

| Technique | Primary Purpose | DMS Example |
|---|---|---|
| Indexing | Faster data retrieval | Searching by Owner_ID |
| Composite Index | Supporting multi-column filters | Searching by Owner_ID and Status |
| Query Plan Analysis | Finding expensive query operations | Investigating slow document searches |
| Partitioning | Managing large tables | Partitioning Audit_Logs by month |
| Replication | Distributing read workload and supporting availability | Serving selected reads through replicas |

## 8. Proposed DMS Database Architecture

The proposed architecture separates operational transactions from analytical processing and stores document files independently.

### Architecture Components

| Component | Responsibility |
|---|---|
| DMS Application | Handles user requests and business operations |
| PostgreSQL | Stores operational metadata and transactional information |
| MinIO / Object Storage | Stores original document files |
| Debezium | Captures PostgreSQL changes through CDC |
| Kafka | Distributes and retains change events |
| Consumer / Sink | Processes events and delivers analytical data |
| ClickHouse | Stores data for analytical queries |
| BI & Reporting | Provides dashboards and reports |

### Proposed Architecture Diagram

```text
                  DMS Application
                         |
              +----------+----------+
              |                     |
              v                     v
         PostgreSQL               MinIO
        (OLTP Database)       (Object Storage)
              |
              |
       Transactional Outbox
              |
              v
           Debezium
              |
         Kafka Connect
              |
              v
             Kafka
              |
              v
       Consumer / Sink
              |
              v
          ClickHouse
        (OLAP Database)
              |
              v
          BI / Reports
```

### Data Processing Flow

1. Users interact with the DMS Application.
2. The application uploads original document files to MinIO.
3. Document metadata and version information are registered in PostgreSQL.
4. An Outbox event is written in the same PostgreSQL transaction as the related metadata changes.
5. Debezium captures committed Outbox records from PostgreSQL.
6. Kafka distributes the events to downstream consumers.
7. A Consumer or Sink processes the events and updates ClickHouse.
8. BI and Reporting tools query ClickHouse for analytical results.

### Important Architecture Considerations

**Operational and analytical isolation**

Analytical queries should not unnecessarily compete with application transactions for PostgreSQL resources.

**CDC instead of repeated full extraction**

Debezium can capture database changes from PostgreSQL transaction logs, reducing the need to repeatedly scan complete tables.

**Transactional Outbox**

Updating PostgreSQL and publishing to Kafka are separate operations. A Transactional Outbox records the event in the same database transaction as the business change. Debezium can then publish committed Outbox records to Kafka.

This reduces the risk of metadata being updated successfully while the corresponding event is never published.

**Object Storage consistency**

MinIO and PostgreSQL do not share one ACID transaction.

If a file upload succeeds but metadata registration fails, an orphan object may remain in MinIO. Reconciliation and cleanup procedures are needed to detect and handle such cases.

**Idempotent Consumers**

Kafka messages may be delivered more than once. Consumers should use unique identifiers or other idempotency controls to prevent duplicate effects in ClickHouse.

**Independent Backups**

Database backups and Object Storage protection should be designed separately. Replication alone does not protect against every type of data loss or corruption.

This is a reference architecture. The actual implementation may use Batch ETL/ELT, Kafka Connect Sink Connectors, or other integration mechanisms depending on project requirements.

## 9. Technical Decision Notes

Database architecture decisions should be based on workload characteristics, consistency requirements, performance expectations, and operational constraints.

### Decision 1: Relational Database for Operational Workloads

A relational database is appropriate for operational DMS data because the application requires:

- Structured records and relationships.
- Transaction management.
- Constraints and referential integrity.
- Consistent updates to related records.
- Efficient retrieval of individual records.

A normalized data model helps reduce duplication and maintain consistency.

**Typical choice:** PostgreSQL.

### Decision 2: Object Storage for Document Files

Document files are stored separately from relational database records.

This approach supports scalable file storage and separates file management from transactional data management.

**Typical choices:** MinIO, Amazon S3, or Ceph Object Gateway.

### Decision 3: Data Warehouse for Analytical Workloads

A dedicated analytical database is used for historical reporting and large-scale queries.

This separation reduces the impact of analytical workloads on operational transactions.

**Typical choices:** ClickHouse, Snowflake, BigQuery, or Amazon Redshift.

### Decision 4: Column-Oriented Storage for Analytical Processing

Column-oriented storage is often preferred for analytical workloads because queries commonly scan selected columns across large datasets and perform aggregations.

The final choice should consider query patterns, compression, concurrency, and the analytical database capabilities.

### Decision 5: Star Schema for Analytical Data Modeling

A Star Schema provides a structured way to organize analytical data through Fact and Dimension Tables.

It simplifies reporting and supports consistent analysis across different business dimensions.

### Decision 6: Performance Optimization

Indexing, partitioning, and replication should be considered according to workload requirements.

- Indexing improves frequently used search queries.
- Partitioning helps manage very large tables and time-based data.
- Replication distributes read workloads and can support availability.
- Query execution plans help identify performance bottlenecks.

### Key Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Separate OLTP and OLAP | Workload isolation | Additional infrastructure and maintenance |
| Object Storage | Scalable file management | Requires separate storage integration |
| Denormalized Analytical Model | Simplifies and can accelerate analytical queries | Data duplication and refresh complexity |
| Column-Oriented Storage | Efficient analytical processing | Less suitable for frequent individual record updates |
| Replication | Read scalability and availability | Replication lag and synchronization complexity |
| Partitioning | Better management of large tables | Additional design and maintenance complexity |
| CDC-Based Integration | Captures changes without repeated full extraction | Requires CDC infrastructure and operational monitoring |

## 10. Conclusion

Database architecture should be designed according to application requirements, workload characteristics, data volume, consistency needs, and performance expectations.

For the proposed DMS:

- Relational databases support operational transactions and maintain data relationships.
- ACID transactions help preserve consistency across related database operations.
- Object Storage manages large document files independently from metadata.
- OLTP and OLAP workloads have different performance requirements and are often separated.
- Row-oriented storage is generally suitable for transactional workloads, while column-oriented storage is commonly used for analytical processing.
- Normalization supports operational consistency.
- Fact and Dimension Tables support analytical modeling.
- Indexing, partitioning, and replication address different performance and scalability requirements.
- CDC and event-driven integration can transfer operational changes to analytical systems.

The key architectural principle is to select database technologies and data models based on how the data is created, updated, stored, queried, and consumed—not simply on the amount of data.
