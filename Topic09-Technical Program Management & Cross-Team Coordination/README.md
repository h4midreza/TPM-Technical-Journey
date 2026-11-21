# Topic 9: Technical Program Management & Cross-Team Coordination

## 1. Objective & Practical Scenario

### Objective

The objective of this topic is to understand how a Technical Program Manager (TPM) coordinates complex technical projects involving multiple engineering teams, infrastructure dependencies, and interconnected deliverables.

The TPM maintains an end-to-end view of technical delivery, coordinates dependencies and decisions, manages risks, and ensures that individual team outputs become an integrated solution.

### Practical Scenario: DMS Delivery

Throughout this topic, we use a Document Management System (DMS) as the main scenario.

The platform receives documents from external systems, validates them, stores document files and metadata, and makes them available for downstream services and reporting.

**High-Level Architecture**

```text
External Systems
       |
       v
      SFTP
       |
       v
      NiFi
       |
       v
     MinIO
       |
       v
     Kafka
       |
       v
Document Processor
       |
       +------> PostgreSQL (Metadata)
       |
       v
   Data Warehouse
       |
       v
       BI
```

The project involves several technical teams:

| Team | Main Responsibility |
|---|---|
| Development | Metadata API and Document Processor |
| Data Engineering | NiFi flows, Kafka integration and data processing |
| Database | PostgreSQL schema, performance and backup |
| Platform / Infrastructure | Kubernetes, storage, networking and deployment |
| Network / Security | Network access, firewall rules and security requirements |

The TPM coordinates these teams to ensure that their deliverables work together as one production-ready platform.

The architecture and team ownership are illustrative and may vary depending on the actual project.

## 2. TPM Fundamentals & Cross-Team Coordination

A Technical Program Manager coordinates technical delivery across teams. The role focuses on dependencies, delivery progress, risks, decisions, and integration rather than implementing individual components.

### TPM vs. Project Manager vs. Tech Lead

| Role | Primary Focus |
|---|---|
| TPM | Technical dependencies, cross-team coordination, integration and delivery |
| Project Manager | Overall project planning, resources, schedule and coordination |
| Tech Lead | Technical design and engineering execution within a team |

These responsibilities may overlap depending on the organization's structure.

### Practical Example

The Development team is implementing the Document Processor, but Kafka integration requires:

- Data Engineering to define the event structure.
- Platform to prepare Kafka topics and access.
- Database to provide the required metadata schema.

Each team owns its technical work, while the TPM coordinates dependencies and ensures that the integration can be completed.

### Responsibility Management: RACI

RACI clarifies who performs, owns, contributes to, or receives information about an activity.

| Role | Meaning |
|---|---|
| R - Responsible | Performs the work |
| A - Accountable | Owns the outcome and approves completion |
| C - Consulted | Provides input or expertise |
| I - Informed | Receives updates |

Example:

| Activity | Development | Data Engineering | Platform | TPM |
|---|---|---|---|---|
| Define event schema | A | R | C | I |
| Create Kafka topics | C | C | R/A | I |
| Implement consumer | R/A | C | I | I |
| Coordinate integration | R | R | R | A |

This is an illustrative allocation. Actual accountability should follow the organization's ownership model. Each activity should have clear accountability.

**TPM responsibilities:**

- Clarify team responsibilities and ownership.
- Identify missing or overlapping responsibilities.
- Coordinate cross-team activities.
- Ensure that dependencies and decisions have owners.
- Maintain visibility of overall technical delivery.

## 3. Project Planning & Technical Dependencies

Project planning starts by breaking down the overall scope into manageable activities and identifying the relationships between them.

### Work Breakdown Structure (WBS)

WBS divides a large project into smaller work packages that can be assigned, estimated, and tracked.

Example: DMS project

```text
1. Requirements and Architecture
   1.1 Define requirements
   1.2 Design system architecture
   1.3 Define integration contracts

2. Infrastructure
   2.1 Prepare Kubernetes resources
   2.2 Configure Kafka
   2.3 Prepare MinIO storage
   2.4 Configure network access

3. Data Ingestion
   3.1 Configure SFTP connectivity
   3.2 Implement NiFi flows
   3.3 Implement validation rules

4. Application Development
   4.1 Implement Metadata API
   4.2 Implement Document Processor
   4.3 Integrate Kafka and MinIO

5. Database
   5.1 Design metadata schema
   5.2 Implement database objects
   5.3 Validate query performance

6. Integration and Testing
   6.1 Component testing
   6.2 Integration testing
   6.3 End-to-end testing
   6.4 Performance testing

7. Production Readiness
   7.1 Configure monitoring
   7.2 Validate security
   7.3 Prepare deployment and rollback
   7.4 Production release
```

A work package should have a clear scope, owner, expected output, dependencies, estimate, and acceptance criteria.

### Technical Dependency Mapping

Dependencies describe relationships between tasks, teams, components, or external requirements.

Common types include:

- **Technical:** Document Processor requires Kafka topics and an agreed event schema.
- **Team:** Development needs the database schema before completing metadata persistence.
- **Infrastructure:** The application requires Kubernetes resources and network connectivity.
- **External:** DMS requires an SFTP endpoint from an external document provider.

Example:

| Activity | Dependency | Owner |
|---|---|---|
| Define Kafka event contract | Architecture agreement | Development + Data Engineering |
| Create Kafka topics | Topic and access requirements | Platform |
| Prepare database schema | Metadata requirements | Database |
| Implement Document Processor | Event contract and API requirements | Development |
| Configure NiFi ingestion | SFTP access and validation rules | Data Engineering |
| Deploy services | Infrastructure readiness | Platform |

### Deliverables, Milestones & Acceptance Criteria

These concepts are related but serve different purposes.

- **Deliverable:** A specific output produced by a team.
- **Milestone:** An important point in the project that represents meaningful progress or completion.
- **Acceptance Criteria:** Conditions that must be satisfied before a deliverable is accepted.

Example:

| Deliverable | Acceptance Criteria | Milestone |
|---|---|---|
| Kafka environment | Topics, ACLs and connectivity verified | Infrastructure Ready |
| Metadata API | Required endpoints implemented and tested | Development Completed |
| NiFi pipeline | Documents received and validated successfully | Ingestion Ready |
| Document Processor | Documents processed and metadata persisted correctly | Processing Ready |
| Integrated DMS | End-to-end workflow passes agreed tests | Integration Completed |

A task should not be considered complete merely because implementation has finished. Completion should be evaluated against agreed acceptance criteria.

**TPM responsibilities:**

- Break down project scope into manageable work packages.
- Identify dependencies between teams and components.
- Assign owners and expected delivery dates.
- Define measurable milestones and acceptance criteria.
- Track dependencies before they become blockers.

## 4. Scheduling & Delivery Management

Once activities and dependencies are identified, the TPM needs to understand how they affect the overall delivery timeline.

### Critical Path & Parallel Work

The Critical Path is the longest sequence of dependent activities that determines the minimum possible project duration.

A delay in a critical-path activity can directly delay the overall project.

Example:

```text
Architecture Approval
        |
        v
Infrastructure Preparation
        |
        v
Kafka and Storage Ready
        |
        v
Application Integration
        |
        v
End-to-End Testing
        |
        v
Production Release
```

Some activities can run in parallel:

```text
             Architecture Approval
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Database    Infrastructure  Data Contract
       Design      Preparation    Definition
          |            |            |
          +------------+------------+
                       |
                       v
                  Integration
                       |
                       v
                 End-to-End Test
                       |
                       v
                Production Release
```

The actual critical path depends on task durations and dependency relationships.

**Key concepts:**

- **Critical Path:** Activities that determine the earliest possible project completion.
- **Float (Slack):** How long an activity can be delayed without delaying the overall project.
- **Parallel Execution:** Performing independent activities simultaneously to reduce total delivery time.

### Technical Estimation

Technical estimation predicts the effort or duration required to complete a task.

Two important concepts:

- **Effort:** Amount of work, usually measured in person-hours or person-days.
- **Duration:** Calendar time required to complete the work.

For example, a task may require 10 person-days of effort, but its duration may be shorter if multiple engineers work on it in parallel. However, dependencies and coordination overhead can limit this reduction.

Common estimation methods:

- **Expert Judgment:** Estimate based on engineering experience.
- **Analogous Estimation:** Use a similar completed task as a reference.
- **Three-Point Estimation:** Use optimistic, most likely, and pessimistic estimates.

A common weighted estimate is:


E = (O + 4M + P) / 6


Example: Kafka integration

- Optimistic: 3 days
- Most likely: 5 days
- Pessimistic: 9 days

E = (3 + (4 * 5) + 9) / 6 = 5.33

The estimated duration is approximately 5.3 days. This is a planning estimate, not a delivery guarantee.

Estimates should clarify whether they include development, code review, testing, integration, deployment, and defect fixing.

**TPM responsibilities:**

- Identify activities that affect the final delivery date.
- Track critical-path dependencies.
- Identify activities that can run in parallel.
- Understand assumptions behind technical estimates.
- Track estimate changes and their impact on milestones.
- Communicate uncertainty instead of treating early estimates as fixed commitments.

## 5. Risk, Issue & Dependency Management

Technical projects contain uncertainties, existing problems, and dependencies that may affect delivery.

### Risk, Issue, Blocker & Assumption

| Term | Meaning | DMS Example |
|---|---|---|
| Risk | A potential future problem | Kafka throughput may be insufficient |
| Issue | A problem that has already occurred | NiFi flow fails on invalid documents |
| Blocker | A problem preventing progress | Development cannot continue without Kafka ACLs |
| Assumption | Something considered true but not yet confirmed | External SFTP will be available on the agreed date |

A blocker is often a high-impact issue, but not every issue is a blocker.

### RAID Log

RAID stands for Risks, Assumptions, Issues, and Dependencies.

It provides a centralized view of items that may affect project delivery.

| Type | Description | Owner | Action |
|---|---|---|---|
| Risk | Kafka throughput may be insufficient | Platform | Perform load testing |
| Assumption | External SFTP endpoint will be available | Integration Owner | Confirm connectivity |
| Issue | NiFi flow fails on invalid documents | Data Engineering | Investigate validation logic |
| Dependency | Document Processor requires Kafka ACLs | Platform | Configure access |

Recommended tracking fields:

- ID and type.
- Description and impact.
- Probability (for risks).
- Owner and target date.
- Status.
- Mitigation or resolution plan.

### Risk Mitigation & Recovery Planning

Common risk response strategies:

- **Avoid:** Change the plan to eliminate the risk.
- **Mitigate:** Reduce its probability or impact.
- **Transfer:** Transfer some responsibility to another party.
- **Accept:** Acknowledge the risk and monitor it.

Example: Kafka throughput may be insufficient for the expected document volume. A mitigation plan could include load testing, capacity planning, and monitoring before production release.

Recovery planning defines how teams respond when a failure occurs.

A recovery plan should identify:

- Detection method.
- Responsible owner.
- Immediate response.
- Recovery procedure.
- Data consistency checks.
- Communication and escalation path.
- Criteria for returning to normal operation.

**DMS example:**

A document is successfully uploaded to MinIO, but metadata persistence fails in PostgreSQL.

Potential consequences:

- An object exists without a metadata record.
- The document cannot be found through normal application queries.
- Storage and database state become inconsistent.

Possible recovery measures include retrying metadata persistence, tracking processing status, reconciling stored objects with metadata records, and identifying orphaned objects.

**TPM responsibilities:**

- Track risks before they become issues.
- Ensure every issue or blocker has an owner and resolution plan.
- Monitor dependencies and target dates.
- Coordinate mitigation and recovery planning.
- Escalate risks that threaten major milestones.

## 6. Change & Technical Decision Management

Changes to requirements, architecture, or project scope should be evaluated before implementation.

### Change Management & Scope Control

Example:

During DMS development, a new requirement is introduced to support an additional document format.

This may affect:

- NiFi validation rules.
- Document Processor implementation.
- Metadata schema.
- Storage requirements.
- Integration testing.
- Delivery timeline.

A typical change management process:

```text
Change Request
      |
      v
Requirement Clarification
      |
      v
Technical Impact Analysis
      |
      v
Effort and Timeline Estimation
      |
      v
Scope / Cost / Risk Evaluation
      |
      v
Approval or Rejection
      |
      v
Plan and Schedule Update
      |
      v
Implementation and Validation
```

A change request should include:

- Description and reason for the change.
- Expected benefit.
- Affected components and teams.
- Technical dependencies.
- Estimated effort and duration.
- Impact on scope, cost, timeline, and risks.
- Alternatives and approval status.

**Scope creep** occurs when additional requirements are introduced gradually without proper evaluation or approval.

The TPM should make changes visible, coordinate impact assessments, obtain appropriate approval, and update the delivery plan.

### Architecture Decision Record (ADR)

An ADR documents an important technical decision, its context, alternatives, and consequences.

Examples:

- Choosing Kafka instead of RabbitMQ.
- Selecting MinIO as object storage.
- Choosing PostgreSQL for metadata.
- Deciding between synchronous and asynchronous processing.

Typical ADR structure:

```markdown
# ADR-001: Use Kafka for Document Processing Events

## Status
Accepted

## Context
The DMS platform requires asynchronous communication
between document ingestion and document processing.

## Decision
Kafka will be used as the messaging platform.

## Alternatives
- RabbitMQ
- Direct synchronous API calls

## Consequences

Positive:
- Supports asynchronous processing.
- Provides message retention and replay.
- Supports consumer groups.

Negative:
- Requires broker infrastructure and monitoring.
- Introduces operational complexity.
- Requires careful partition and consumer management.

## Owners
Development, Data Engineering, Platform
```

ADRs help preserve decision history, explain technical trade-offs, clarify ownership, and support future architecture changes.

**TPM responsibilities:**

- Ensure significant technical decisions are documented.
- Coordinate reviews with relevant technical stakeholders.
- Make alternatives and consequences visible.
- Track required approvals.
- Ensure decisions are communicated to affected teams.

The technical decision itself should be owned by the appropriate technical stakeholders.

## 7. Technical Impact Analysis

A technical change can affect multiple components, teams, and delivery activities. Impact analysis helps identify these consequences before implementation.

### Database Schema Changes

Example: A new `document_category` field is added to the DMS metadata table.

Potential impacts:

- Database schema migration.
- Metadata API changes.
- Document Processor changes.
- Existing records and default values.
- BI queries and reporting.
- Data validation and integration tests.

### API Changes

Example: The Metadata API changes a response field from `document_id` to `id`.

Potential impacts:

- Document Processor integration.
- External API consumers.
- API documentation.
- Automated tests.
- Backward compatibility.
- Deployment sequence.

### Architecture Changes

Example: The project changes from synchronous document processing to asynchronous processing through Kafka.

Potential impacts:

- Kafka infrastructure.
- Producer and consumer implementation.
- Retry and error handling.
- Message ordering.
- Monitoring and alerting.
- Processing latency.
- Operational complexity.

### Impact Analysis Matrix

| Area | Questions |
|---|---|
| Application | Which services require changes? |
| Database | Are schema or migration changes required? |
| Data Pipeline | Are ingestion or transformation flows affected? |
| Infrastructure | Are additional resources or configurations needed? |
| Integration | Which upstream and downstream systems are affected? |
| Security | Are permissions or security controls changing? |
| Testing | Which test scenarios must be repeated? |
| Delivery | What is the impact on milestones and release dates? |
| Cost | Are additional infrastructure or operational costs expected? |

### TPM Responsibilities

The TPM coordinates the assessment and ensures that relevant teams provide their technical input.

The assessment should clarify:

- Why the change is needed.
- Which components and teams are affected.
- What additional work and dependencies are introduced.
- What alternatives exist.
- What happens if the change is not implemented.
- How the change affects scope, cost, risk, and delivery.

The result should provide enough information for stakeholders to make an informed decision.

## 8. Infrastructure, Integration & Release Coordination

A software release requires more than completed application code. Infrastructure, environment configuration, database changes, access, and deployment processes must also be ready.

### Environments

| Environment | Purpose |
|---|---|
| Development | Initial development and component testing |
| Testing / QA | Functional and integration testing |
| Staging / Pre-Production | Production-like validation |
| Production | Live service environment |

Not every organization uses all these environments.

### External Infrastructure and Service Dependencies

Projects often depend on infrastructure and services owned by other teams or external organizations.

Common examples:

| Dependency | Requirement |
|---|---|
| Network | Connectivity between DMS and external SFTP |
| Firewall | Required source and destination ports |
| DNS | Internal service name resolution |
| Storage | MinIO capacity |
| Kubernetes | Cluster resources |
| Kafka | Topics and ACLs |
| Database | PostgreSQL provisioning and access |
| Security | Certificates and credentials |
| External Vendor | API or SFTP availability |

**Example:**

The DMS project needs to receive documents from an external system through SFTP.

The integration cannot be tested until the endpoint is available, network routes and firewall rules are configured, credentials are exchanged, and connectivity is verified.

These activities should be tracked as project dependencies with owners and expected dates.

### Release Dependencies

Before deploying the Document Processor, the following may be required:

- Container image available in the registry.
- Kubernetes deployment configuration prepared.
- Kafka connectivity and ACLs configured.
- PostgreSQL schema changes applied.
- MinIO access configured.
- Secrets and environment variables available.
- Monitoring and alerting configured.
- Rollback procedure defined.

### Database and Application Compatibility

Application releases can fail when database changes are not compatible with the deployed application version.

For example, removing a database column while the previous application version still depends on it may cause production failures.

One safer approach for some schema changes is the Expand-and-Contract pattern:

1. Add the new schema structure while keeping the old one.
2. Deploy application code that supports the transition.
3. Migrate or backfill data if required.
4. Verify that the new structure is being used.
5. Remove the old structure in a later release.

### TPM Responsibilities

- Identify external and infrastructure dependencies early.
- Confirm owners, lead times, and required approvals.
- Track environment and configuration readiness.
- Coordinate application and database releases.
- Ensure deployment and rollback plans exist.
- Verify testing and acceptance criteria.
- Coordinate release windows across teams.

**Key point:** A successful CI pipeline does not guarantee a successful production release. The deployed system must also be healthy, compatible, and operational.

## 9. Technical Status Reporting & Escalation

Technical status reporting communicates actual delivery conditions, not just the number of completed tasks.

A useful report explains progress, blockers, risks, dependencies, and their impact on the project.

### Status Report Structure

**1. Overall Status**

- Green: Progress is on track.
- Amber: Risks or delays require attention.
- Red: A significant issue threatens delivery or a major milestone.

Status definitions should be agreed upon within the organization.

**2. Progress:** Completed and ongoing work.

**3. Blockers and Issues:** What is preventing progress and who owns the resolution.

**4. Risks:** Potential problems that may affect future delivery.

**5. Dependencies:** Pending activities from other teams or external parties.

**6. Decisions Required:** Decisions that need technical or management approval.

**7. Next Steps:** Important upcoming activities.

### Example: DMS Weekly Status Report

```text
Project: DMS

Overall Status: Amber

Progress:
- PostgreSQL schema design completed.
- Metadata API development is ongoing.
- NiFi ingestion flow is partially implemented.

Blockers:
- Kafka ACL configuration is still pending.
- Development cannot complete consumer integration
  until the required access is provided.

Risks:
- Delayed Kafka availability may affect integration testing.

Dependencies:
- Platform team: Kafka topic and ACL configuration.
- Network team: External SFTP connectivity.

Decisions Required:
- Confirmation of the initial production storage capacity.

Next Steps:
- Complete Kafka integration.
- Validate SFTP connectivity.
- Start end-to-end testing after infrastructure readiness.
```

### Escalation

Escalation means involving a higher level of authority or another responsible stakeholder when an issue cannot be resolved through normal coordination.

Escalation is appropriate when:

- A critical dependency is repeatedly delayed.
- Teams cannot agree on ownership.
- A technical decision blocks multiple teams.
- Scope, budget, or timeline changes require approval.
- A production risk requires management attention.

A good escalation should include:

- What happened?
- What is the impact?
- Which milestone or deliverable is affected?
- What actions have already been taken?
- What decision or support is required?
- By when is the decision needed?

**TPM responsibility:** Communicate project health accurately, highlight delivery risks early, and escalate issues with clear evidence and a specific request for support.

## 10. Hands-on & Deliverables

### Hands-on Scenario: DMS Integration Delay

The DMS project is approaching its integration testing milestone.

The following situation has occurred:

- The Database team has completed the metadata schema.
- Development has completed most of the Document Processor implementation.
- Data Engineering has completed the NiFi ingestion flow.
- The Platform team has not configured Kafka ACLs.
- The Network team has not completed external SFTP connectivity.
- A new requirement has been introduced to support an additional document format.

These conditions may affect integration testing and the production delivery date.

### Tasks

**1. Dependency Analysis**

Identify the dependencies preventing integration testing. For each dependency, identify the responsible team and expected output.

**2. Impact Analysis**

Evaluate the new document format requirement. Consider application, data pipeline, database, testing, scope, and timeline impacts.

**3. Risk and Issue Management**

Classify the following:

- Kafka ACL configuration is pending.
- External SFTP connectivity is unavailable.
- The new document format may increase processing time.

Determine which are risks, issues, blockers, or assumptions where applicable.

**4. Delivery Planning**

Identify which activities can continue in parallel and which activities are on the critical path.

**5. Coordination and Escalation**

Prepare a short status report explaining:

- Current project status.
- Main blockers.
- Delivery risks.
- Affected milestones.
- Required decisions or support.

The objective is to demonstrate how a TPM maintains end-to-end delivery visibility and coordinates teams without taking ownership of their technical implementation.

### Expected Deliverables

| Deliverable | Purpose |
|---|---|
| Dependency Map | Shows technical, team, and infrastructure dependencies |
| WBS | Breaks the project into manageable work packages |
| Milestone Plan | Defines major delivery stages and completion conditions |
| Acceptance Criteria | Defines how deliverables are evaluated |
| Critical Path View | Identifies activities that determine project duration |
| RAID Log | Tracks risks, assumptions, issues, and dependencies |
| Estimation Summary | Documents effort, duration, assumptions, and uncertainty |
| Change Request | Records and evaluates scope or technical changes |
| ADR | Documents important architecture decisions |
| RACI Matrix | Clarifies ownership and responsibilities |
| Release Readiness Plan | Tracks environment, deployment, and operational readiness |
| Impact Analysis | Evaluates the consequences of technical changes |
| Risk Mitigation Plan | Defines preventive and recovery actions |
| Technical Status Report | Communicates progress, blockers, risks, and decisions |

### Final Outcome

After completing this topic, a TPM should be able to:

- Coordinate technical delivery across multiple teams.
- Break down complex projects into manageable activities.
- Understand and track technical dependencies.
- Manage milestones, estimates, and critical-path activities.
- Identify and manage risks, issues, and blockers.
- Evaluate scope changes and technical decisions.
- Coordinate infrastructure, integration, and production releases.
- Communicate project health and escalate delivery risks.
- Maintain an end-to-end view of technical delivery.

The main goal is to ensure that individual team outputs become one integrated, reliable, and operational solution.
