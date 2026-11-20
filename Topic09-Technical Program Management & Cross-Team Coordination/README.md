# Topic 9: Technical Program Management & Cross-Team Coordination

## 1. Objective

Understand how to coordinate technical work across Data Engineering, DevOps, Database, Network, Security, and Consumer teams while managing dependencies, risks, and delivery commitments.

This topic focuses on technical program management, dependency mapping, work breakdown, delivery planning, risk management, technical decision-making, ownership, and cross-team coordination.

The goal is to understand how to organize and coordinate complex technical initiatives, identify delivery constraints, manage uncertainty, and communicate progress and risks from a Technical Program Manager (TPM) perspective.

## 2. Practical Scenario: Data Platform Delivery

Assume a Data Platform project requires several teams to deliver an end-to-end data pipeline.

The target architecture includes:

- Source Systems: Provide operational data.
- Network: Provides secure connectivity between systems.
- Storage: Provides infrastructure for storing raw and processed data.
- NiFi: Handles data ingestion and routing.
- Kafka: Transfers events between services.
- Document Processor: Validates and processes incoming data.
- ClickHouse: Stores analytical data.
- Monitoring Platform: Monitors infrastructure, applications, and data pipelines.
- Consumer Teams: Access and use the processed data.

Several teams must coordinate their work.

| Team | Main Responsibility |
|---|---|
| Data Engineering | Ingestion, processing, and data integration |
| DevOps / Platform | Kubernetes, deployment, and infrastructure |
| Database | PostgreSQL and ClickHouse configuration |
| Network | Connectivity, routing, firewall, and DNS |
| Security | Access control, certificates, and security approval |
| Consumer Teams | Data access, integration, and validation |

A delay in one team's deliverable can affect other teams and the overall delivery timeline.

The TPM coordinates dependencies, milestones, risks, responsibilities, and delivery commitments across these teams.

## 3. Technical Program Management Fundamentals

Technical Program Management (TPM) focuses on coordinating complex technical initiatives involving multiple teams, systems, and dependencies.

Unlike managing a single team's tasks, TPM requires understanding how different technical workstreams depend on one another.

### Core Responsibilities

- **Planning:** Define scope, milestones, and delivery expectations.
- **Coordination:** Align teams and manage cross-team dependencies.
- **Dependency Management:** Identify work that depends on external teams or infrastructure.
- **Risk Management:** Identify and track potential delivery risks.
- **Decision Management:** Ensure technical decisions are documented and communicated.
- **Progress Tracking:** Monitor milestones, blockers, and delivery commitments.
- **Escalation:** Raise issues that cannot be resolved at the working-team level.

### Example

The Data Engineering team cannot start NiFi integration because Network has not provided access to the source system.

The TPM should:

1. Identify the missing network access as a dependency.
2. Confirm the responsible Network team and required access details.
3. Understand the impact on the ingestion schedule.
4. Coordinate a committed delivery date.
5. Identify alternative work that can proceed in parallel.
6. Escalate if the delay threatens a critical milestone.

The TPM does not necessarily configure the firewall but must ensure that the dependency is resolved or its impact is managed.

## 4. Technical Dependency Mapping

A dependency is a relationship where one task, component, or deliverable requires another to be completed or available.

Dependency Mapping makes these relationships visible.

### Types of Dependencies

- **Technical Dependency:** One technical component requires another component.
- **Team Dependency:** One team's work depends on another team's deliverable.
- **Infrastructure Dependency:** Application deployment requires infrastructure readiness.
- **External Dependency:** A third-party provider or external system must provide something.
- **Data Dependency:** A downstream process requires data from an upstream source.

### Example Dependency Map

```text
Source System Access
        |
        v
Network Connectivity
        |
        v
NiFi Ingestion
        |
        v
Kafka Integration
        |
        v
Document Processing
        |
        v
ClickHouse Integration
        |
        v
Data Validation
        |
        v
Consumer Integration
```

Additional dependencies:

```text
Security Approval ----+
                      |
Storage Provisioning -+--> Data Platform Deployment
                      |
Kubernetes Readiness -+
                      |
Monitoring Setup -----+
```

### Dependency Management

For each dependency, identify:

- Required deliverable.
- Responsible team.
- Team or component waiting for it.
- Expected completion date.
- Current status.
- Impact if delayed.

A dependency map helps identify bottlenecks and understand how a delay in one component can affect downstream work.

## 5. Work Breakdown Structure (WBS)

A Work Breakdown Structure decomposes a large project into smaller, manageable work packages.

It helps teams understand the scope of work and organize delivery.

### Example: Data Platform WBS

| WBS ID | Work Package | Responsible Team |
|---|---|---|
| 1.0 | Infrastructure Preparation | DevOps |
| 1.1 | Kubernetes Cluster Readiness | DevOps |
| 1.2 | Storage Provisioning | Storage |
| 1.3 | Network Connectivity | Network |
| 2.0 | Data Ingestion | Data Engineering |
| 2.1 | Source Integration | Data Engineering |
| 2.2 | NiFi Configuration | Data Engineering |
| 3.0 | Data Processing | Data Engineering |
| 3.1 | Kafka Integration | Data Engineering |
| 3.2 | Document Processing | Data Engineering |
| 4.0 | Analytical Storage | Database |
| 4.1 | ClickHouse Deployment | Database / DevOps |
| 4.2 | Data Integration | Data Engineering |
| 5.0 | Monitoring | DevOps / Data Engineering |
| 5.1 | Infrastructure Monitoring | DevOps |
| 5.2 | Pipeline Monitoring | Data Engineering |
| 6.0 | Integration and Acceptance Testing | All Relevant Teams |

### Work Package Characteristics

A good work package should have:

- Clear scope.
- Defined owner.
- Expected deliverable.
- Acceptance criteria.
- Estimated effort or duration.
- Identified dependencies.

WBS helps define what needs to be delivered before assigning dates and tracking progress.

## 6. Deliverables, Milestones, and Acceptance Criteria

These concepts are related but have different purposes.

### Deliverable

A specific output produced by a task or work package.

Examples:

- Kubernetes Cluster ready.
- NiFi ingestion flow configured.
- ClickHouse database deployed.
- Monitoring dashboard available.
- Data integration completed.

### Milestone

A significant point in the project that represents the completion of an important stage or achievement.

Examples:

- Infrastructure Ready.
- First Successful Data Ingestion.
- End-to-End Pipeline Operational.
- User Acceptance Testing Completed.
- Production Go-Live.

### Acceptance Criteria

Conditions that must be satisfied before a deliverable can be accepted as complete.

Example: NiFi Ingestion Acceptance Criteria:

- Source connectivity is established.
- Required records can be ingested.
- Data validation rules are applied.
- Failed records are handled correctly.
- Ingestion metrics are available.
- Integration tests pass.

A task being marked as completed does not automatically mean its deliverable meets the acceptance criteria.

## 7. Critical Path and Parallel Work

### Critical Path

The Critical Path is the longest sequence of dependent activities that determines the earliest possible project completion date.

Tasks on the critical path generally have zero scheduling float. A delay in one of these tasks can delay the overall delivery date.

### Example

```text
Source Access (3 days)
        |
        v
Network Setup (2 days)
        |
        v
NiFi Integration (4 days)
        |
        v
Kafka Integration (3 days)
        |
        v
Document Processing (5 days)
        |
        v
ClickHouse Integration (3 days)
        |
        v
End-to-End Testing (4 days)
```

In this simplified sequence, all tasks depend on the completion of the previous task.

### Parallel Work

Not every activity needs to wait for another activity.

For example, the following work may happen in parallel:

- Kubernetes Cluster preparation.
- Security review.
- Source data analysis.
- Monitoring architecture design.
- ClickHouse schema design.

Parallel execution can reduce the overall delivery time, provided that dependencies and resource constraints are respected.

### TPM Considerations

- Identify the critical path.
- Identify tasks that can run in parallel.
- Track tasks with limited scheduling flexibility.
- Monitor dependencies that could delay critical milestones.
- Recalculate the delivery plan when important assumptions change.

## 8. Risk, Issue, Blocker, and Assumption

These terms describe different project conditions and should not be treated as interchangeable.

### Risk

An uncertain event that may affect project delivery in the future.

Example: Network connectivity may not be approved before the planned integration date.

### Issue

A problem that has already occurred and requires action.

Example: The Kubernetes Cluster is unavailable because of a failed infrastructure deployment.

### Blocker

A condition that prevents a task or team from making progress.

Example: Data Engineering cannot start source integration because required firewall access has not been provided.

A blocker can also be an issue, but the term emphasizes its impact on progress.

### Assumption

Something considered true for planning purposes but not yet confirmed.

Example: The TPM assumes that the source system can provide at least 100,000 records per hour.

If the assumption is incorrect, the project plan may need to change.

### Comparison

| Concept | Meaning | Example |
|---|---|---|
| Risk | Potential future problem | Network approval may be delayed |
| Issue | Existing problem | Network approval has failed |
| Blocker | Prevents progress | Integration cannot start |
| Assumption | Unverified planning condition | Source can handle required data volume |

## 9. Technical Estimation and Uncertainty

Technical estimation predicts the time, effort, or resources required to complete technical work.

Estimates depend on factors such as:

- Technical complexity.
- Team capacity.
- Dependencies.
- Availability of infrastructure.
- Integration requirements.
- Unknown technical constraints.

### Estimation Approaches

**Expert Estimation:** Estimates are provided by engineers or technical leads based on experience.

**Analogous Estimation:** Uses similar completed work as a reference.

**Three-Point Estimation:** Uses three values:

- Optimistic: Best-case duration.
- Most Likely: Expected duration.
- Pessimistic: Worst-case duration.

A common PERT estimate is:

`Expected Duration = (Optimistic + 4 × Most Likely + Pessimistic) / 6`

### Example

For ClickHouse Integration:

- Optimistic: 3 days.
- Most Likely: 5 days.
- Pessimistic: 10 days.

Expected duration:

`(3 + 4 × 5 + 10) / 6 = 5.5 days`

This estimate helps represent uncertainty rather than treating the five-day estimate as a guaranteed commitment.

### TPM Considerations

- Understand the assumptions behind estimates.
- Separate effort from calendar duration.
- Consider external dependencies and team availability.
- Avoid presenting uncertain estimates as fixed commitments.
- Track estimate changes and their causes.

## 10. Change Management and Scope Control

Change Management controls changes to project scope, requirements, architecture, and delivery commitments.

Changes may originate from:

- New business requirements.
- Technical constraints.
- Security requirements.
- Infrastructure limitations.
- Integration discoveries.
- Performance or scalability needs.

### Example

During Data Platform integration, the team discovers that the existing PostgreSQL configuration cannot support the expected workload.

The team proposes migrating part of the workload to a different database architecture.

Before accepting the change, the TPM should assess:

- Technical impact.
- Additional implementation effort.
- Infrastructure requirements.
- Dependencies on other teams.
- Testing requirements.
- Delivery timeline impact.
- Operational and maintenance costs.

### Scope Control

A change should be evaluated and approved before being incorporated into the delivery baseline.

Important elements include:

- Change description.
- Business or technical justification.
- Impact analysis.
- Required approvals.
- Updated scope and schedule.
- Communication to affected teams.

Uncontrolled scope changes can cause delays, increased costs, and unclear delivery expectations.

## 11. Architecture Decision Records (ADR)

An Architecture Decision Record documents an important technical decision and its context.

ADRs help teams understand why an architectural choice was made and what alternatives were considered.

### Typical ADR Structure

1. **Title:** Short name of the decision.
2. **Status:** Proposed, Accepted, Rejected, or Superseded.
3. **Context:** Problem or requirement.
4. **Decision:** Selected technical approach.
5. **Alternatives:** Other options considered.
6. **Consequences:** Benefits, limitations, and trade-offs.

### Example: Kafka Message Partitioning

**Context:**

DMS needs to preserve the processing order of events belonging to the same document.

**Options:**

- Use `document_id` as the Kafka message key.
- Distribute messages without a key.
- Use a custom partitioning strategy.

**Decision:**

Use `document_id` as the message key.

**Reason:**

Events for the same document are assigned to the same partition, preserving their partition-level ordering.

**Consequences:**

- Ordering is maintained within each partition for a given document key.
- Uneven key distribution may create partition hotspots.
- Changing partition count can affect key-to-partition mapping.

### TPM Role

The TPM does not necessarily make every technical decision but should ensure that important decisions:

- Have a clear owner.
- Are documented.
- Consider relevant alternatives.
- Include technical consequences.
- Are communicated to affected teams.

## 12. Ownership and Responsibility Matrix (RACI)

RACI defines team responsibilities for project activities.

It helps prevent unclear ownership and delays caused by assuming another team is responsible.

### RACI Definitions

- **R - Responsible:** Performs the work.
- **A - Accountable:** Ultimately owns the result and approves completion.
- **C - Consulted:** Provides input or expertise.
- **I - Informed:** Receives progress updates.

Each activity should normally have one clearly identified Accountable owner.

### Example RACI Matrix

| Activity | Data Eng. | DevOps | Database | Network | Security | TPM |
|---|---|---|---|---|---|---|
| Source Integration | R | I | I | C | C | A |
| Kubernetes Setup | C | R/A | I | C | C | I |
| Database Provisioning | C | C | R/A | I | C | I |
| Firewall Configuration | C | I | I | R/A | C | I |
| Security Approval | C | C | C | C | R/A | I |
| Project Coordination | C | C | C | C | C | R/A |

The exact RACI assignment depends on organizational authority and team ownership.

RACI should define responsibility for outcomes, not just list teams that participate.

## 13. Release and Environment Dependencies

Software delivery usually involves multiple environments:

- Development
- Testing
- Staging
- Production

A release may depend on several conditions being satisfied.

### Example Release Dependencies

Before deploying Metadata API to Production:

- Required database migrations are ready.
- Production database is available.
- Container image has been approved.
- Kubernetes configuration is validated.
- Secrets and certificates are available.
- Network access is configured.
- Monitoring and alerting are ready.
- Rollback strategy is defined.

### Release Sequencing

The order of deployment matters.

For example, when adding a new database field:

1. Apply a backward-compatible database migration.
2. Deploy the updated application.
3. Validate application behavior.
4. Enable the new functionality.
5. Remove obsolete schema elements only after compatibility is confirmed.

Poor release sequencing can cause application failures or data inconsistency.

## 14. External Infrastructure and Service Dependencies

Projects often rely on resources and services managed outside the immediate delivery team.

Examples include:

- Firewall and network approvals.
- Third-party APIs.
- DNS and certificate services.
- Storage infrastructure.
- Kubernetes Cluster availability.
- Identity and Access Management.
- External data sources.
- Vendor-provided software or infrastructure.

### External Dependency Register

| Dependency | Responsible Team / Provider | Required By | Potential Impact |
|---|---|---|---|
| Source System Access | Source Team | NiFi Integration | Ingestion delay |
| Firewall Rules | Network | Source Integration | Connectivity failure |
| Storage Provisioning | Storage Team | Document Processing | Deployment blocker |
| Database Access | Database Team | Application Integration | Application unavailable |
| Security Approval | Security Team | Production Release | Release delay |
| DNS and Certificates | Network / Security | Service Exposure | Connectivity or TLS failure |

### TPM Responsibilities

- Confirm external ownership.
- Track required delivery dates.
- Validate readiness before dependent work starts.
- Identify alternative approaches where possible.
- Escalate delayed dependencies.

## 15. Impact Analysis for Schema, API, and Architecture Changes

Technical changes can affect multiple teams and systems.

Before accepting a change, its impact must be understood.

### Schema Changes

Example: Adding a mandatory field to a PostgreSQL table.

Potential impacts:

- Existing applications may fail if they do not provide the field.
- Data ingestion pipelines may require updates.
- Downstream analytical models may need changes.
- Historical records may require backfilling.

### API Changes

Example: Changing the Metadata API response structure.

Potential impacts:

- Consumer applications may break.
- Integration tests need updates.
- API documentation must be revised.
- Version compatibility must be considered.

### Architecture Changes

Example: Replacing an existing message broker or changing the data ingestion architecture.

Potential impacts:

- Infrastructure requirements.
- Network and security configuration.
- Application integration.
- Data migration.
- Operational monitoring.
- Team responsibilities and support processes.

### Impact Analysis Checklist

Before approving a change, identify:

- Affected components.
- Dependent systems and teams.
- Backward compatibility.
- Required migrations.
- Testing requirements.
- Deployment sequencing.
- Rollback and recovery options.
- Delivery timeline impact.

## 16. Risk Mitigation and Recovery Planning

Risk Mitigation defines actions that reduce the probability or impact of potential problems.

Recovery Planning defines how service or project delivery can be restored when a problem occurs.

### Risk Register

A Risk Register tracks identified risks and their management plans.

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Infrastructure not ready | Medium | High | Validate readiness before integration |
| Poor source data quality | High | High | Add validation and data profiling |
| Insufficient processing capacity | Medium | High | Load testing and resource planning |
| Integration incompatibility | Medium | High | Contract and integration testing |
| Delayed security approval | Medium | Medium | Start security review early |
| Failed Production Release | Low | High | Rollback plan and release validation |

### Risk Mitigation vs. Recovery

**Mitigation:**

Reduces the probability or impact of a risk before it becomes a problem.

Example: Perform load testing before Production deployment.

**Recovery:**

Restores the system or delivery after a failure has occurred.

Example: Roll back to the previous application version after a failed release.

### Recovery Planning

A recovery plan should define:

- Recovery owner.
- Recovery steps.
- Required dependencies.
- Rollback or failover procedure.
- Data recovery requirements.
- Communication and escalation process.
- Validation criteria after recovery.

## 17. Technical Status Reporting and Escalation

Technical status reporting communicates project progress, delivery risks, blockers, and decisions required from stakeholders.

A useful status report should distinguish between completed work, current issues, potential risks, and upcoming actions.

### Example Technical Status Report

**Project:** Data Platform Delivery

**Overall Status:** At Risk

**Completed:**
- Kubernetes Cluster provisioned.
- PostgreSQL deployed.
- Initial NiFi configuration completed.

**In Progress:**
- Kafka integration.
- Document Processor integration testing.
- Monitoring dashboard configuration.

**Blockers:**
- Production firewall access has not been approved.
- Source integration testing cannot be completed.

**Risks:**
- Delayed network approval may affect the End-to-End Testing milestone.
- Current processing capacity has not yet been validated under expected workload.

**Decisions Needed:**
- Confirm the Production firewall approval date.
- Confirm whether additional processing capacity will be provisioned.

**Next Actions:**
- Network team to complete firewall configuration.
- Data Engineering to continue Kafka integration in parallel.
- DevOps to validate monitoring and resource configuration.
- TPM to reassess the delivery timeline after dependency confirmation.

### Escalation

Escalation is required when:

- A blocker cannot be resolved at the working-team level.
- A critical milestone is at risk.
- A dependency has no confirmed owner or delivery date.
- A major technical decision requires management approval.
- Delivery commitments are likely to be affected.

A good escalation should clearly describe the problem, impact, responsible owner, required decision, and latest date by which action is needed.

