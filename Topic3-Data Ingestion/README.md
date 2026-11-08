# Topic 3: Data Pipelines & Data Integration
## 1. Objective
Understand how data is collected from different sources, transferred, validated, transformed, and delivered to analytical systems through reliable data pipelines.
The focus is on understanding practical data integration architectures, commonly used tools, pipeline reliability, and the technical dependencies and risks that should be monitored by a Technical Program Manager (TPM).
## 2. Data Ingestion Fundamentals
Data ingestion is the process of collecting data from different sources and transferring it into a target platform for further processing and analysis.
Common ingestion methods include:
- **File-based ingestion:** Transferring files through SFTP or shared storage.
- **API-based ingestion:** Retrieving data from external systems through REST APIs.
- **Database ingestion:** Extracting records or database changes from operational databases.
- **Event-based ingestion:** Receiving events through messaging platforms such as Apache Kafka.
### Common Tools
| Tool | Main Purpose | Typical Use Case |
|---|---|---|
| Apache NiFi | Data ingestion, routing, and flow management | Receiving files from SFTP, validating and routing data |
| Kafka Connect | Connecting external systems to Kafka | Moving data between databases, Kafka, and other systems |
| Debezium | Change Data Capture (CDC) | Capturing database inserts, updates, and deletes |
| Logstash | Log collection and processing | Collecting and forwarding application logs |
**Practical distinction:**
- NiFi is useful when ingestion flows require visual configuration, routing, validation, and file handling.
- Kafka Connect is useful for standardized integrations between Kafka and external systems.
- Debezium is commonly used with Kafka Connect to capture database changes without repeatedly extracting entire tables.
- Logstash is primarily focused on log collection and processing.
Tool selection depends on data sources, volume, latency requirements, operational complexity, and the existing infrastructure.
## 3. ETL vs. ELT
Both ETL and ELT describe approaches for preparing data before analytical use.
### ETL (Extract, Transform, Load)
1. Extract data from source systems.
2. Transform and clean the data.
3. Load the processed data into the target system.
**Example:** Data is extracted from a DMS database, validated and transformed using a processing engine, and then loaded into an analytical database.
### ELT (Extract, Load, Transform)
1. Extract data from source systems.
2. Load the raw data into a data lake or warehouse.
3. Transform the data inside the target platform.
**Example:** Raw DMS metadata is first loaded into a data warehouse, and SQL-based transformations are performed afterward using dbt.
### Comparison
| Aspect | ETL | ELT |
|---|---|---|
| Transformation | Before loading | After loading |
| Target | Receives processed data | Receives raw data first |
| Processing location | External processing engine | Target platform |
| Typical use | Controlled preprocessing | Large-scale analytical processing |
| Main consideration | Processing capacity before loading | Storage and compute capacity in the target |
**Common technologies:**
- ETL / processing: Apache NiFi, Apache Spark.
- ELT transformation: dbt and SQL-based warehouse processing.
The choice depends on data volume, processing requirements, target platform capabilities, and data governance needs.
## 4. Batch vs. Stream Processing
### Batch Processing
Data is collected and processed periodically in groups.
Examples:
- Processing all documents received during the previous day.
- Generating daily reports.
- Loading database records every hour.
Typical tools:
- Apache Airflow for scheduling and orchestration.
- Apache Spark for large-scale batch processing.
- dbt for scheduled SQL transformations.
### Stream Processing
Data is processed continuously or near real-time as events arrive.
Examples:
- Processing document upload events.
- Tracking changes in document status.
- Monitoring system events as they occur.
Typical technologies:
- Apache Kafka for event streaming.
- Apache Flink for stateful stream processing.
- Spark Structured Streaming for stream processing.
### Comparison
| Aspect | Batch | Streaming |
|---|---|---|
| Processing | Periodic | Continuous |
| Latency | Minutes or hours | Seconds or milliseconds, depending on design |
| Complexity | Generally simpler | Requires additional operational considerations |
| Typical use | Reporting and historical analysis | Real-time monitoring and event processing |
A platform may use both approaches. For example, a DMS can process document metadata in batches while publishing document events through Kafka for near real-time consumers.
## 5. Full Load vs. Incremental Load
Data can be transferred using two common approaches.
### Full Load
The entire dataset is extracted and loaded into the target system.
Typical use cases:
- Initial data migration.
- Small datasets.
- Rebuilding a target dataset when necessary.
Main limitation:
- Can consume significant network, processing, and storage resources for large datasets.
### Incremental Load
Only new or changed records are transferred.
Common approaches:
- **Timestamp-based:** Selecting records updated after the last successful processing time.
- **Watermark-based:** Keeping track of the last processed timestamp or identifier.
- **CDC:** Capturing database changes such as inserts, updates, and deletes.
Example:
If the last successful load processed records up to 10:00 AM, the next load retrieves records changed after that point.
### Important Considerations
- A reliable watermark must be maintained.
- Late-arriving records should not be missed.
- Overlap windows can help capture records that arrive late or have delayed timestamps.
- Upsert operations should update existing records or insert new ones.
- Delete events must be handled explicitly when the source system supports deletion.
Incremental loading reduces unnecessary data transfer but requires careful handling of duplicates, updates, and deleted records.
## 6. Pipeline Orchestration and Scheduling
Orchestration manages the execution order and dependencies between pipeline tasks.
For example, a DMS pipeline may require the following sequence:
1. Receive files.
2. Validate file format and required fields.
3. Store files in Object Storage.
4. Register metadata.
5. Publish an event.
6. Process metadata in the analytical target.
The next step should not start until its required dependencies are satisfied.
### Common Orchestration Tools
| Tool | Main Purpose | Typical Use |
|---|---|---|
| Apache Airflow | Workflow orchestration and scheduling | Managing scheduled batch pipelines and task dependencies |
| Apache NiFi | Data flow management | Managing ingestion, routing, and file processing |
| Prefect | Workflow orchestration | Managing Python-based data workflows |
| Dagster | Data orchestration | Managing data assets, dependencies, and pipelines |
**Important distinction:**
Airflow schedules and coordinates tasks; it is not itself a data processing engine. NiFi manages data flows and ingestion, while Kafka handles event streaming. These tools may work together rather than replace one another.
## 7. Pipeline Reliability and Failure Recovery
A production pipeline must handle failures without losing data or creating inconsistent results.
### Retry
Retry means executing a failed operation again.
Typical use:
- Temporary network failures.
- Temporary database unavailability.
- Short-lived service interruptions.
Retries should have limits and backoff intervals to avoid continuously overloading an unavailable service.
### Idempotency
An operation is idempotent when repeating it does not create unintended additional effects.
Example:
If the same document event is delivered twice, the target system should not create two separate document records.
A common approach is using a unique `event_id` or a business key to identify already processed events.
### Replay and Reprocessing
- **Replay:** Reading previously published events again, for example from Kafka.
- **Reprocessing:** Processing data again, potentially with corrected logic or updated transformation rules.
- **Backfill:** Loading historical data that was missed or was not previously processed.
These operations require careful handling of duplicates and existing target records.
### Dead Letter Queue (DLQ)
A DLQ stores messages that cannot be successfully processed after the configured retry policy.
It helps separate permanently problematic messages from normal processing.
### Late and Invalid Data
A pipeline should have a defined strategy for:
- Missing required fields.
- Invalid file formats.
- Duplicate records.
- Late-arriving data.
- Unexpected schema changes.
- Partially processed records.
Invalid data may be routed to a quarantine area for investigation instead of blocking the entire pipeline.
## 8. Data Validation and Schema Evolution
Data validation checks whether incoming data meets expected requirements.
Examples:
- Required fields are present.
- Document IDs are valid.
- Timestamps follow the expected format.
- File sizes are within acceptable limits.
- Records do not violate defined business or technical rules.
### Schema Evolution
Schema evolution refers to changes in the structure of data over time.
For example, a new field such as `document_category` may be added to the document metadata.
Potential risks:
- Consumers may not recognize the new field.
- Required fields may be removed or renamed.
- Data types may change.
- Older and newer records may have different structures.
A data contract defines the expected structure, data types, and compatibility rules between producers and consumers.
Schema changes should be communicated and tested before deployment, especially when multiple downstream systems depend on the same data.
## 9. Practical DMS Data Pipeline Architecture
The following architecture illustrates how document files and metadata can be integrated into an analytical platform.
```text
          DMS Source Systems
                  |
          +-------+-------+
          |               |
       SFTP Files     REST API / DB
          |               |
          +-------+-------+
                  |
             Apache NiFi
                  |
        Validation & Metadata
                  |
          +-------+-------+
          |               |
      Object Storage   Metadata DB
       (MinIO/S3)     (PostgreSQL)
          |               |
          +-------+-------+
                  |
           Event Publishing
                Kafka
                  |
          Kafka Consumer
                  |
        Analytical Database
          / Data Warehouse
                  |
              BI / Reports
```
### Architecture Explanation
- **SFTP / REST API / Database:** Provide document files and metadata.
- **Apache NiFi:** Receives data, validates it, extracts metadata, and manages ingestion flows.
- **Object Storage:** Stores document files in MinIO or an S3-compatible storage system.
- **PostgreSQL:** Stores operational metadata, document references, and processing status.
- **Apache Kafka:** Publishes events such as `DocumentUploaded` or `DocumentUpdated`.
- **Kafka Consumer:** Receives events and updates the analytical target.
- **Analytical Database / Data Warehouse:** Stores data for reporting and analysis.
- **BI Tools:** Provide dashboards and analytical reports.
### Important Design Considerations
- Large document files should be stored in Object Storage, not directly inside Kafka messages.
- Kafka events can contain metadata and an `object_key` pointing to the stored file.
- The file should be stored successfully before publishing an event that indicates it is available.
- If Object Storage succeeds but metadata registration or event publishing fails, the system needs a recovery mechanism.
- Idempotency and reconciliation are important to prevent duplicate records and inconsistent states.
This architecture is a reference design. The actual implementation may use different tools depending on infrastructure and project requirements.
## 10. Common Failure Scenarios
| Scenario | Potential Impact | Possible Recovery |
|---|---|---|
| SFTP unavailable | New files are not received | Retry and monitor source availability |
| Invalid file received | File cannot be processed | Quarantine and report the error |
| Object Storage unavailable | File storage fails | Retry and monitor storage health |
| Metadata DB unavailable | Document status may not be registered | Retry with idempotent operations |
| Kafka publishing fails | Consumers do not receive the event | Retry or publish through a reliable recovery mechanism |
| Consumer fails | Kafka lag increases | Restore consumer and monitor lag |
| Duplicate event received | Duplicate processing | Use event IDs and idempotent writes |
| Schema changes unexpectedly | Consumer processing may fail | Validate contracts and coordinate compatibility |
A successful operation in one component does not necessarily mean the complete pipeline has succeeded. Each stage should have observable status and a defined recovery strategy.
## 11. TPM Technical Checklist
A TPM does not need to implement the pipeline but should understand its architecture, dependencies, and operational risks.
### Architecture and Dependencies
- What are the data sources and ingestion methods?
- Which tools are responsible for ingestion, processing, orchestration, and storage?
- Which components depend on successful completion of previous steps?
- Are there external systems or teams that may delay delivery?
### Capacity and Performance
- What is the expected daily data volume?
- What is the expected peak ingestion rate?
- What is the required processing latency?
- Can downstream systems handle the incoming data volume?
- Are there known bottlenecks in network, storage, database, or processing capacity?
### Reliability and Recovery
- What happens when a component becomes unavailable?
- Are retries configured, and what are their limits?
- How are duplicate records prevented?
- Is there a DLQ or quarantine mechanism?
- Can historical data be replayed or reprocessed?
- How is data consistency verified across components?
### Monitoring and Delivery Readiness
- Are ingestion failures and processing delays monitored?
- Is pipeline status visible to the responsible teams?
- Are alerting thresholds defined?
- Are recovery procedures documented and tested?
- Are data quality and completeness checks available?
## 12. Key Takeaways
- Data pipelines connect source systems to analytical platforms through ingestion, validation, transformation, and delivery.
- Tools such as NiFi, Kafka Connect, Debezium, Airflow, Spark, and dbt address different parts of the data integration process.
- ETL and ELT, as well as batch and streaming, are different architectural choices based on business and technical requirements.
- Incremental loading improves efficiency but requires reliable handling of updates, deletes, duplicates, and late data.
- Retry, idempotency, replay, backfill, and DLQ are essential concepts for reliable pipeline operations.
- A production-ready pipeline requires monitoring, data validation, recovery procedures, and clear ownership of dependencies.
- For a TPM, understanding component interactions, bottlenecks, failure scenarios, and delivery risks is more important than implementing the pipeline directly.
