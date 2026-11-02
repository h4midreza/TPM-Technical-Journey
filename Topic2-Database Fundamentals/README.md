# Database Fundamentals, OLTP, OLAP & Data Modeling

## 1. Objective

Understand relational database fundamentals, distinguish between operational and analytical workloads, and evaluate how database architecture and data modeling decisions affect the performance, scalability, and reliability of a Data Platform.

This topic uses a Document Management System (DMS) as a practical example to explore database design and architecture decisions.

## 2. Scenario: Document Management System (DMS)

The organization operates a DMS that allows users to upload, organize, search, access, and manage documents.

### Core Capabilities

- User and role management
- Document and folder management
- Document upload and download
- Document versioning
- Access control and permissions
- Document search and retrieval
- User activity tracking through audit logs
- Usage and operational reporting

### Workload Characteristics

The DMS supports two main workload types:

- **Operational Workload:** Handles daily user activities such as document uploads, updates, permission changes, and version management.
- **Analytical Workload:** Supports reporting and analysis, such as monthly document uploads, user activity trends, document distribution, and storage consumption.

## 3. Relational Database Fundamentals

A relational database organizes data into structured tables connected through defined relationships.

### Core Concepts

- **Table:** Stores data related to a specific entity, such as Users or Documents.
- **Row:** Represents an individual record.
- **Column:** Represents an attribute of an entity.
- **Schema:** Defines the database structure, including tables, columns, data types, constraints, and relationships.
- **Primary Key (PK):** Uniquely identifies each record in a table.
- **Foreign Key (FK):** References a primary key in another table to establish relationships and maintain referential integrity.

### Main Operational Tables

| Table | Purpose |
|---|---|
| Users | Stores user information |
| Folders | Manages folder structures |
| Documents | Stores document metadata |
| Document_Versions | Tracks document versions |
| Permissions | Manages user access rights |
| Audit_Logs | Records user activities |

Document files such as PDF and Word are typically stored in Object Storage or File Storage rather than directly in relational database tables. The database maintains metadata such as file name, type, size, and storage path.

### Database Relationships

- Users to Documents: One-to-Many
- Folders to Documents: One-to-Many
- Documents to Document_Versions: One-to-Many
- Users to Documents through Permissions: Many-to-Many

## 4. OLTP vs. OLAP

### OLTP (Online Transaction Processing)

OLTP systems handle daily operational transactions and support frequent read and write operations.

DMS examples:

- Uploading a document
- Updating document metadata
- Creating folders
- Changing access permissions
- Registering a new document version

### OLAP (Online Analytical Processing)

OLAP systems support analytical queries, reporting, and aggregation across large volumes of data.

DMS examples:

- Monthly document upload statistics
- User activity analysis
- Document distribution by category
- Storage consumption reports
- Identification of the most active users

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

### Why Separate OLTP and OLAP Workloads?

Running complex analytical queries directly against an operational database can consume significant CPU, memory, and I/O resources.

This may affect transaction response times and reduce application performance.

A common architectural approach is to separate operational and analytical workloads by transferring operational data into a dedicated Data Warehouse.

## 5. Row-Oriented vs. Column-Oriented Storage

Database storage organization affects how efficiently data can be accessed and processed.

### Row-Oriented Storage

Data is stored record by record, with the attributes of each row stored together.

It is generally suitable for:

- Transaction processing
- Inserting and updating individual records
- Retrieving complete records
- Operational database workloads

### Column-Oriented Storage

Data is organized by columns, allowing queries to read only the required attributes instead of processing entire records.

It is generally suitable for:

- Large-scale analytical queries
- Aggregations and statistical calculations
- Reporting across large datasets
- Data Warehouse workloads

### Comparison

| Characteristic | Row-Oriented | Column-Oriented |
|---|---|---|
| Storage Organization | By rows | By columns |
| Best Fit | OLTP | OLAP |
| Individual Record Access | Efficient | Usually less efficient |
| Large-Scale Aggregations | Less efficient | Highly efficient |
| Common Usage | Operational databases | Analytical databases |

The actual performance depends on database technology, indexing, compression, query patterns, and workload characteristics.

## 6. Data Modeling

Data modeling defines how data is structured, organized, and connected to support application operations and analytical requirements.

### Normalization

Normalization organizes data into related tables to reduce unnecessary duplication and maintain data consistency.

For example, user information is stored in the Users table, while document information is stored separately in the Documents table. The relationship is established through User_ID.

Normalization is generally preferred in operational databases.

### Denormalization

Denormalization introduces controlled data duplication to improve query performance and simplify analytical reporting.

For example, an analytical dataset may include user names and document categories directly alongside document activity records to reduce the need for repeated joins.

Denormalization is commonly used in analytical environments.

### Fact Tables and Dimension Tables

**Fact Table**

Stores measurable business events and activities.

Example: Fact_Document_Activity

- Activity_ID
- User_ID
- Document_ID
- Date_ID
- Activity_Type
- File_Size

**Dimension Tables**

Store descriptive information used to categorize and analyze facts.

Examples:

- Dim_User: User details and roles
- Dim_Document: Document types and categories
- Dim_Date: Day, month, quarter, and year

### Star Schema

Star Schema is a common Data Warehouse modeling approach in which a central Fact Table connects to multiple Dimension Tables.

It supports analytical queries such as:

- Number of uploaded documents per month
- User activity by department
- Storage consumption by document category
- Most active users over a selected period

## 7. Database Performance and Architecture

Several techniques help improve database performance and support growing workloads.

### Indexing

Indexes improve data retrieval by allowing the database to locate records without scanning the entire table.

DMS example:

Creating an index on Owner_ID can improve document searches based on document ownership.

Indexes require additional storage and may increase the cost of insert and update operations.

### Partitioning

Partitioning divides a large table into smaller logical sections based on a selected key.

DMS example:

The Audit_Logs table can be partitioned by month or year to improve queries over specific time periods and simplify data retention and archival.

### Replication

Replication maintains copies of database data across multiple database servers.

A common architecture includes:

- Primary Database: Handles write operations.
- Read Replicas: Handle read requests and distribute read workload.

DMS example:

User transactions are processed by the primary database, while selected read requests can be served by read replicas.

Replication may introduce data synchronization delays, so read-after-write consistency requirements must be considered.

### Performance Techniques Comparison

| Technique | Primary Purpose | DMS Example |
|---|---|---|
| Indexing | Faster data retrieval | Searching documents by Owner_ID |
| Partitioning | Managing large tables | Partitioning Audit_Logs by month |
| Replication | Distributing read workload | Serving read requests through replicas |

## 8. Proposed DMS Database Architecture

The proposed architecture separates operational transactions from analytical processing.

### Architecture Components

| Component | Responsibility |
|---|---|
| DMS Application | Handles user requests and business operations |
| Operational Database | Stores current transactional data |
| Object Storage | Stores document files |
| ETL / ELT Pipeline | Extracts, transforms, and loads operational data |
| Data Warehouse | Stores structured analytical data |
| BI & Reporting | Provides analytical reports and dashboards |

### Architecture Diagram

<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/442c8263-c33d-4b2d-a703-335f7ecde473" />

### Data Processing Flow

1. Users interact with the DMS Application.
2. Document metadata and transactional information are stored in the Operational Database.
3. Actual document files are stored in Object Storage.
4. Operational data is extracted and transferred through ETL / ELT processes.
5. The Data Warehouse organizes data for analytical workloads.
6. BI and Reporting tools use the Data Warehouse to generate dashboards and reports.

## 9. Technical Decision Note

### Decision 1: Relational Database for Operational Workloads

A relational database is selected for operational data because the DMS requires structured data, defined relationships, transaction management, and referential integrity.

A normalized data model helps reduce duplication and maintain consistency.

### Decision 2: Object Storage for Document Files

Document files are stored separately from relational database records.

This approach supports scalable file storage and separates file management from transactional data management.

### Decision 3: Data Warehouse for Analytical Workloads

A dedicated Data Warehouse is used for historical reporting and analytical queries.

This separation prevents complex analytical workloads from directly affecting operational transaction performance.

### Decision 4: Column-Oriented Storage for Analytical Processing

Column-oriented storage is preferred for analytical workloads because queries often process selected columns across large datasets and perform aggregations.

### Decision 5: Star Schema for Analytical Data Modeling

A Star Schema is selected to simplify analytical queries and support reporting through Fact and Dimension Tables.

### Decision 6: Performance Optimization

Indexing, partitioning, and replication are considered according to workload requirements.

- Indexing improves frequently used search queries.
- Partitioning supports large tables and time-based data management.
- Replication distributes read workloads and can improve read scalability.

### Key Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Separate OLTP and OLAP | Workload isolation | Additional infrastructure and maintenance |
| Object Storage | Scalable file management | Requires separate storage integration |
| Denormalized Analytical Model | Faster analytical queries | Data duplication |
| Column-Oriented Storage | Efficient analytical processing | Less suitable for frequent individual record updates |
| Replication | Read scalability and availability | Replication lag and synchronization complexity |
| Partitioning | Better management of large datasets | Additional design and maintenance complexity |

## 10. Conclusion

Database architecture should be designed according to application requirements, workload characteristics, data volume, and performance expectations.

For the proposed DMS:

- Relational databases support operational transactions.
- Object Storage manages document files.
- Data Warehouses support analytical workloads.
- Normalization maintains operational data consistency.
- Fact and Dimension Tables support analytical modeling.
- Indexing, partitioning, and replication help improve database performance and scalability.

Separating operational and analytical workloads provides a foundation for a scalable and maintainable Data Platform.
