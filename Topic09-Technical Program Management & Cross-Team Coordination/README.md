# Topic 9: Project & Technical Delivery Management

## 1. Objective & Practical Scenario

### Objective

Understand how technical projects are planned, coordinated, monitored, and delivered across multiple teams.

This topic focuses on project planning, technical dependencies, scheduling, risk and change management, impact analysis, integration coordination, and delivery reporting.

The goal is to understand how a Technical Program Manager (TPM) coordinates technical teams, manages cross-team dependencies, identifies delivery risks, and ensures that separate technical activities contribute to a successful project outcome.

### Practical Scenario: DMS Project

Assume a Document Management System (DMS) project with the following high-level data flow:

```text
External Systems
       |
      SFTP
       |
      NiFi
       |
     MinIO
       |
     Kafka
       |
Document Processor
       |
   PostgreSQL
       |
      DWH
       |
       BI
```

The project involves several technical teams:

| Team | Main Responsibility |
|---|---|
| Development | Document Processor and Metadata API |
| Data Engineering | NiFi flows, Kafka integration, and data processing |
| Database | PostgreSQL schema, performance, and backup |
| Platform / Infrastructure | Kubernetes, storage, and infrastructure configuration |
| Network / Security | Network connectivity, firewall rules, and access control |

The project requires coordination between these teams because the completion of one component may depend on the availability of another.

For example, the Document Processor may be developed and ready for testing, but integration testing cannot be completed until the required Kafka access and connectivity are available.

The TPM coordinates these activities, manages dependencies, tracks delivery progress, and helps ensure that the complete solution is delivered rather than treating each team's output as an independent result.

---

## 2. TPM Fundamentals & Cross-Team Coordination

### 2.1 PM vs. TPM vs. Tech Lead

These roles have different primary responsibilities, although their boundaries may vary across organizations.

| Role | Primary Responsibility |
|---|---|
| Project Manager (PM) | Project planning, schedule, resources, budget, and overall delivery coordination |
| Technical Program Manager (TPM) | Technical dependencies, cross-team coordination, technical risks, and integrated delivery |
| Technical Lead | Technical design, implementation approach, engineering decisions, and technical quality |

**Example:**

In the DMS project:

- The Development Tech Lead is responsible for the Document Processor implementation.
- The Project Manager tracks the overall project schedule and resources.
- The TPM coordinates dependencies between Development, Data Engineering, Database, and Platform.

The TPM does not replace technical leads or make every engineering decision. The focus is on ensuring that technical activities are aligned and that cross-team issues do not prevent delivery.

### 2.2 Cross-Team Coordination

Cross-Team Coordination ensures that different teams can work together toward shared project milestones.

Key responsibilities include:

- Defining responsibilities and expected outputs.
- Identifying dependencies between teams.
- Coordinating delivery dates and integration activities.
- Following up on blockers and cross-team issues.
- Ensuring alignment on technical requirements and interfaces.
- Tracking decisions and communicating relevant changes.

For example, Development and Data Engineering must agree on the Kafka Event Contract before integrating their respective components.

### 2.3 RACI Matrix

RACI is a responsibility assignment model used to clarify involvement in project activities.

| Role | Meaning |
|---|---|
| R – Responsible | Performs the activity |
| A – Accountable | Owns the final result and is answerable for it |
| C – Consulted | Provides input or expertise |
| I – Informed | Receives updates about the activity |

Example:

| Activity: Configure Kafka ACLs | RACI |
|---|---|
| Platform Engineer | R |
| Platform Lead | A |
| Development | C |
| Data Engineering | C |
| TPM | I |

Each activity should normally have one clearly identified Accountable role to avoid ambiguity in ownership.

---

## 3. Project Planning & Technical Dependencies

### 3.1 Work Breakdown Structure (WBS)

A Work Breakdown Structure divides a project into smaller, manageable activities and deliverables.

It helps define the work required, assign ownership, estimate effort, and track progress.

Example: DMS project activities.

| Activity | Expected Output | Responsible Team |
|---|---|---|
| SFTP Integration | External connectivity ready | Network |
| NiFi Flow Development | Ingestion flow configured | Data Engineering |
| Object Storage Setup | MinIO environment ready | Platform |
| Kafka Integration | Required Topics and integration ready | Data Engineering / Development |
| Document Processor | Processing service ready | Development |
| Metadata Database | Database schema ready | Database |
| DWH Integration | Data available in analytical storage | Data Engineering |
| BI Integration | Reports and dashboards available | BI / Development |

Activities should have clear outputs. For example, "Work on Kafka" is too broad. "Configure required Kafka Topics and ACLs" is more specific and trackable.

### 3.2 Dependency Mapping

A dependency exists when an activity requires another activity, output, or external condition before it can start or finish.

Dependencies can exist between teams, technical components, environments, or external systems.

Example:

| Activity | Dependency | Responsible Team |
|---|---|---|
| NiFi Ingestion | SFTP connectivity | Network / Data Engineering |
| Document Processor Integration | Kafka access and Event Contract | Platform / Development / Data Engineering |
| Metadata Registration | PostgreSQL schema and connectivity | Database / Development |
| Integration Testing | Required components and interfaces ready | All relevant teams |

Dependencies should be identified early because delays in one activity may affect several downstream activities.

### 3.3 Milestones & Deliverables

These concepts are related but have different meanings:

- **Deliverable:** A specific output produced by the project.
- **Milestone:** An important point in the project schedule representing the completion of a stage or achievement of a major objective.

Example:

| Deliverable | Milestone |
|---|---|
| SFTP configuration | External connectivity ready |
| NiFi ingestion flow | Ingestion completed |
| Document Processor | Processing service ready |
| Integration test results | Integration testing completed |

### 3.4 Acceptance Criteria

Acceptance Criteria define the conditions that must be satisfied before an output can be considered complete and accepted.

They make completion measurable and reduce ambiguity between teams.

For example, SFTP integration may have these acceptance criteria:

- Connection to the external server is successful.
- A test file can be received.
- The received file is placed in the expected location.
- Connection failures can be identified and logged.

A task should not be considered complete only because the responsible team reports that implementation has finished. The agreed acceptance criteria should also be satisfied.

---

## 4. Scheduling & Delivery Management

Scheduling translates project activities and dependencies into a realistic delivery plan.

The TPM coordinates estimates, monitors schedule progress, and identifies activities that may affect important project milestones.

### 4.1 Critical Path

The Critical Path is the longest sequence of dependent activities that determines the minimum time required to complete a project.

Activities on the Critical Path directly affect the project completion date. A delay in one of these activities may delay the entire project if there is no available schedule flexibility.

Example:

```text
SFTP Connectivity
       |
NiFi Ingestion Validation
       |
Kafka Integration
       |
Integration Testing
       |
Production Release
```

This is an illustrative dependency chain. The actual Critical Path depends on activity durations and the complete dependency network.

**Float** is the amount of time an activity can be delayed without delaying a dependent activity or the overall project completion date.

Activities with zero Total Float are considered critical in the schedule.

### 4.2 Parallel Work

Not all project activities need to be completed sequentially. Independent activities can run in parallel to reduce the overall delivery time.

For example, in the DMS project:

- Network can prepare SFTP connectivity.
- Platform can configure Kafka ACLs.
- Development can continue unit and mock-based testing.
- Development and Data Engineering can agree on the Event Contract.

These activities can progress in parallel, provided their own prerequisites are satisfied.

Parallel work reduces waiting time but does not eliminate dependencies. Integration Testing still requires the relevant components and interfaces to be ready.

### 4.3 Effort vs. Duration

Effort and Duration are different measures:

- **Effort:** Total work required to complete an activity, usually measured in person-hours or person-days.
- **Duration:** Calendar time between the start and finish of an activity.

For example, an activity requiring six person-days of effort may take approximately six working days with one person or three working days with two people, assuming the work can be divided effectively.

Adding more people does not always reduce duration proportionally because some activities have dependencies, coordination overhead, or limited parallelism.

### 4.4 Estimation

The TPM coordinates estimation with technical owners and uses the estimates to build and maintain the project schedule.

One estimation method is Three-Point Estimation, which considers three possible scenarios:

| Estimate | Meaning |
|---|---|
| Optimistic (O) | Duration under favorable conditions |
| Most Likely (M) | Expected duration under normal conditions |
| Pessimistic (P) | Duration if potential difficulties occur |

A common weighted estimate is:


E = (O + 4M + P) / 6


Example: Kafka integration

- Optimistic: 3 days
- Most likely: 5 days
- Pessimistic: 9 days

E = (3 + (4 * 5) + 9) / 6 = 5.33

The estimated duration is approximately 5.3 days. This is a planning estimate, not a delivery guarantee.

Estimates should be reviewed when assumptions, technical conditions, or project dependencies change.

---

## 5. Risk, Issue & Dependency Management

This area focuses on identifying uncertainties, managing current problems, and tracking dependencies that may affect project delivery.

### 5.1 Risk, Issue, Blocker & Assumption

| Concept | Definition |
|---|---|
| Risk | An uncertain future event that may affect the project |
| Issue | A problem that has already occurred and requires action |
| Blocker | An issue or condition preventing an activity from progressing |
| Assumption | Something accepted as true for planning purposes but not yet confirmed |

Example:

- Risk: External SFTP access may not be available on time.
- Issue: The SFTP connection is currently failing.
- Blocker: Ingestion testing cannot proceed without the required connectivity.
- Assumption: The external system will be available before the planned integration date.

A Risk may become an Issue when the uncertain event occurs. However, not every Issue is necessarily a Blocker.

### 5.2 RAID Log

A RAID Log is used to record and track four categories of project information:

- **Risks:** Potential future problems.
- **Assumptions:** Unconfirmed conditions used for planning.
- **Issues:** Existing problems requiring resolution.
- **Dependencies:** Activities or outputs that rely on other teams, systems, or external conditions.

A RAID Log should normally include:

| Field | Purpose |
|---|---|
| ID | Unique reference |
| Category | Risk, Assumption, Issue, or Dependency |
| Description | Details of the item |
| Impact / Priority | Potential or actual effect |
| Owner | Person or team responsible for follow-up |
| Action / Mitigation | Planned response |
| Due Date | Target resolution or review date |
| Status | Current state |

The log provides visibility and ensures that important items are assigned, monitored, and followed up.

### 5.3 Risk Response Strategies

After identifying a Risk, an appropriate response strategy should be selected.

| Strategy | Meaning |
|---|---|
| Avoid | Change the plan to eliminate the risk |
| Mitigate | Reduce the likelihood or impact of the risk |
| Transfer | Shift responsibility for the risk or its consequences to another party |
| Accept | Acknowledge the risk and proceed with an appropriate response plan if needed |

The selected strategy depends on the likelihood, impact, cost, and available alternatives.

### 5.4 Recovery Planning

A Recovery Plan defines the actions required to recover from a problem or reduce its impact on project delivery.

It may include:

- Identifying the root cause or current limitation.
- Assigning owners to corrective actions.
- Finding alternative ways to continue independent work.
- Adjusting task priorities or sequencing.
- Assessing the impact on milestones and delivery dates.
- Communicating revised expectations to stakeholders.

Recovery planning should define specific actions, owners, and expected outcomes rather than simply recording that a delay has occurred.

---

## 6. Change & Technical Decision Management

Projects may receive new requirements or technical changes during implementation. These changes can affect project scope, schedule, cost, architecture, and team responsibilities.

Change Management ensures that proposed changes are evaluated, approved, documented, and incorporated into the delivery plan.

### 6.1 Change Management

A change should not automatically become part of the project scope without understanding its impact.

The change management process generally includes:

1. Identify and describe the requested change.
2. Understand the reason and expected outcome.
3. Assess technical and delivery impacts.
4. Review required effort, resources, risks, and dependencies.
5. Obtain the appropriate approval.
6. Update the project plan and coordinate implementation.

### 6.2 Change Request (CR)

A Change Request formally describes a proposed change and provides the information needed for review and approval.

A Change Request typically includes:

| Field | Description |
|---|---|
| Change Description | What needs to change |
| Reason | Why the change is required |
| Expected Outcome | What the change should achieve |
| Impact Analysis | Affected components, teams, and dependencies |
| Effort & Timeline | Estimated additional work and schedule impact |
| Risks | Potential technical and delivery risks |
| Approval | Review and approval status |

A typical process is:

```text
Change Request
      |
Impact Analysis
      |
Review & Approval
      |
Project Plan Update
      |
Implementation
```

Approved changes should be reflected in the project scope, schedule, and relevant delivery documents.

### 6.3 Technical Decision Management

Technical projects often have multiple possible solutions for the same problem.

Technical decisions may affect architecture, performance, scalability, security, cost, and future maintenance.

The relevant Tech Lead or Architect normally owns the technical assessment and decision. The TPM coordinates the decision process, ensures relevant teams are consulted, and tracks the decision and its delivery implications.

### 6.4 Architecture Decision Record (ADR)

An Architecture Decision Record documents an important architectural decision and the reasoning behind it.

A typical ADR includes:

| Section | Purpose |
|---|---|
| Context | Problem or situation requiring a decision |
| Decision | Selected solution or approach |
| Alternatives | Other options considered |
| Consequences | Benefits, limitations, and possible side effects |
| Status | Proposed, Accepted, Superseded, or other agreed status |

For example, an ADR may document why a DMS architecture stores original files in MinIO and publishes only the Object Key through Kafka rather than sending large file contents in Kafka messages.

ADRs provide traceability and help teams understand why a particular technical approach was selected.

---

## 7. Technical Impact Analysis

Technical Impact Analysis evaluates how a proposed change or technical issue may affect system components, integrations, infrastructure, and project delivery.

Its purpose is to identify the required work, affected teams, dependencies, risks, and potential impact on scope, timeline, and cost before implementation or replanning.

### 7.1 Impact Areas

A technical change should be evaluated across the relevant areas:

| Area | Key Considerations |
|---|---|
| Application | Services, APIs, application logic, and interfaces |
| Database & Data | Schema changes, data structures, and data compatibility |
| Data Pipeline | Data formats, processing logic, and downstream consumers |
| Infrastructure | Compute, storage, network, and capacity requirements |
| Integration | Dependencies between internal and external systems |
| Testing & Operations | Test coverage, monitoring, support, and recovery requirements |
| Project Delivery | Effort, resources, dependencies, milestones, and cost |

Not every change affects all areas. The analysis should focus on the components and teams relevant to the requested change.

### 7.2 Impact Analysis Process

A structured impact analysis typically includes:

1. **Identify the Change:** Understand what is changing and why.
2. **Identify Affected Components:** Determine which systems, interfaces, and teams may be affected.
3. **Collect Technical Assessments:** Obtain required changes, effort estimates, dependencies, and risks from technical owners.
4. **Evaluate Delivery Impact:** Assess the effect on scope, timeline, resources, and cost.
5. **Document Findings:** Record assumptions, open questions, risks, and required decisions.

### 7.3 Impact Analysis Output

The result should provide a consolidated view of:

- Affected components and teams.
- Required activities and responsible owners.
- Technical dependencies and risks.
- Estimated effort and additional resource requirements.
- Expected impact on project milestones and delivery dates.
- Decisions or approvals required before implementation.

**TPM Responsibility:**

The TPM coordinates the analysis and consolidates the findings. Technical teams assess implementation details and provide estimates, while the relevant decision-makers approve changes to scope, resources, or delivery commitments.

---

## 8. Infrastructure, Integration & Release Coordination

This area focuses on coordinating the technical readiness of environments, infrastructure, integrations, and production releases.

The TPM ensures that required activities are planned, dependencies have clear owners, and technical readiness is confirmed before moving to the next delivery stage.

### 8.1 Environment & Infrastructure Readiness

Applications depend on properly configured environments and infrastructure.

Key readiness areas include:

| Area | Readiness Considerations |
|---|---|
| Environments | Development, Test, Staging, and Production availability |
| Compute & Storage | Required resources and persistent storage |
| Network | Connectivity, DNS, firewall rules, and external access |
| Security | Credentials, permissions, certificates, and access control |
| External Dependencies | Availability of external systems and required services |
| Monitoring | Logs, metrics, and operational visibility |

Environment readiness should be confirmed before integration and release activities begin.

### 8.2 Integration Readiness

Integration readiness means that the components involved in a workflow can communicate and exchange data according to agreed technical contracts.

Key considerations include:

- API, Event, and Data Contract compatibility.
- Required endpoints, Topics, and access permissions.
- Network connectivity and authentication.
- Availability of test data and test environments.
- Agreed integration scenarios and expected results.
- Clear ownership of integration issues.

Component-level testing does not guarantee successful integration. Interfaces, configurations, and dependencies must also be validated.

### 8.3 Release Coordination

Release coordination ensures that all required technical and operational activities are aligned before production deployment.

Key release considerations include:

| Area | Required Consideration |
|---|---|
| Release Scope | Approved features, fixes, and changes |
| Deployment Plan | Deployment sequence, versions, and dependencies |
| Compatibility | Application, API, and Database Schema compatibility |
| Validation | Required tests and acceptance results |
| Rollback | Rollback approach, criteria, and responsible owner |
| Operations | Monitoring, support ownership, and incident handling |

**Release Readiness:**

A release should proceed only when the agreed readiness criteria are satisfied and the required technical owners have confirmed their responsibilities.

**TPM Responsibility:**

The TPM coordinates readiness confirmations, tracks unresolved dependencies, aligns deployment activities, and communicates release risks. Technical teams remain responsible for implementation, validation, and operational approval within their areas.

---

## 9. Technical Status Reporting & Escalation

Technical Status Reporting communicates project progress, delivery confidence, technical risks, and decisions required from stakeholders.

A useful status report should focus on actual delivery progress and expected outcomes rather than only reporting completed tasks.

### 9.1 Status Report Structure

A project status report should include:

| Section | Information |
|---|---|
| Overall Status | Current project health and delivery confidence |
| Progress | Completed work and milestone progress |
| Schedule | Planned vs. forecast completion dates |
| Key Risks & Issues | Important risks, blockers, and unresolved issues |
| Dependencies | Open cross-team or external dependencies |
| Changes & Decisions | Approved changes and pending technical decisions |
| Next Steps | Upcoming activities, owners, and target dates |
| Support Required | Decisions or management actions needed |

### 9.2 RAG Status

RAG (Red, Amber, Green) provides a simple way to communicate project health.

| Status | Meaning |
|---|---|
| Green | Delivery remains within the agreed plan, with no major unresolved threats. |
| Amber | A risk, dependency, or schedule variance may affect delivery, but mitigation or recovery actions are available. |
| Red | A major issue or dependency threatens an agreed milestone or delivery commitment, and requires significant intervention. |

RAG status should be based on evidence, impact, and the current delivery forecast, not simply the percentage of completed tasks.

### 9.3 Escalation

Escalation means raising an issue to a higher management or decision-making level when it cannot be resolved effectively within the current level of responsibility.

Common escalation triggers include:

- A critical blocker remains unresolved.
- A dependency is delayed and threatens a major milestone.
- Teams cannot reach agreement on ownership or priorities.
- Additional resources or management decisions are required.
- An issue threatens the agreed scope, budget, security, or delivery date.

An effective escalation should communicate:

1. **Issue:** What has happened?
2. **Impact:** How does it affect the project?
3. **Actions Taken:** What has already been done?
4. **Options:** What possible solutions are available?
5. **Decision Required:** What support or decision is needed, from whom, and by when?

**TPM Responsibility:**

The TPM identifies when escalation is necessary, provides an evidence-based summary, and ensures that the required decision or action is followed up.

---

## 10. Hands-on & Deliverables

### Practical Scenario: DMS Integration Delay

The DMS project is approaching its Integration Testing milestone.

Current situation:

- Database team has completed the Metadata Schema.
- Development team has completed most of the Document Processor implementation.
- Data Engineering has completed the NiFi ingestion flow.
- Platform has not configured Kafka ACLs.
- Network has not completed external SFTP connectivity.
- A new requirement has been introduced to support an additional document format.

The TPM must assess the delivery situation, coordinate the involved teams, and determine the actions required to protect the project timeline.

### Expected Deliverables

The exercise should produce the following project artifacts:

| Deliverable | Purpose |
|---|---|
| WBS & Ownership Matrix | Define activities, expected outputs, and responsible teams |
| Dependency Map & Schedule | Identify dependencies, parallel activities, and potential Critical Path |
| RAID Log | Track risks, assumptions, issues, and dependencies with owners |
| Change Request & Impact Analysis | Evaluate the new document format requirement and its delivery impact |
| Integration & Release Readiness Plan | Track environment, integration, testing, and release prerequisites |
| Technical Status Report | Communicate project health, delivery forecast, blockers, and required decisions |

An ADR should also be created when the change requires a significant technical or architectural decision.

The objective is to produce a coordinated delivery plan with clear ownership, realistic estimates, managed dependencies, and transparent project status.


