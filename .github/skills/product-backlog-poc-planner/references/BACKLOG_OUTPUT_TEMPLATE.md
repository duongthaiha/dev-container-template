# Backlog Output Template

Use this template when producing the final backlog.

## Assumptions

| Assumption | Why It Matters | Validation Needed |
|---|---|---|
| Example: Database service is not selected yet | Networking and Bicep modules depend on the service | Architecture decision |

## POC Goal

One paragraph that explains the thin slice. Keep it outcome-first: user, workflow, proof, and constraint.

## Functional Requirements

Use [Functional Requirements Template](FUNCTIONAL_REQUIREMENTS_TEMPLATE.md). Assign stable `FR-###` identifiers and define the observable behavior the POC must provide.

| ID | Capability | Actor | Trigger | Required Behavior | Inputs | Outputs | Business Rules | Priority | Acceptance Evidence | Dependencies | Status / Open Decision |
|---|---|---|---|---|---|---|---|---|---|---|---|
| FR-001 | `<capability>` | `<user, system, or service>` | `<event that starts the behavior>` | `<observable behavior>` | `<input or precondition>` | `<result or state change>` | `<validation, authorization, approval, or processing rules>` | Must/Should/Could | `<demo or test proof>` | `<requirement, integration, or decision>` | Confirmed/Open/Deferred |

## Non-Functional Requirements

Use [Non-Functional Requirements Template](NON_FUNCTIONAL_REQUIREMENTS_TEMPLATE.md). Assign stable `NFR-###` identifiers and make every target measurable or explicitly mandatory.

| ID | Quality Attribute | Scope | Target / Threshold | Measurement Method | Priority | Rationale | Dependencies | Status / Open Decision |
|---|---|---|---|---|---|---|---|---|
| NFR-001 | `<quality attribute>` | `<workflow, component, or environment>` | `<measurable target or constraint>` | `<test, metric, audit, or evidence>` | Must/Should/Could | `<why it matters>` | `<service, requirement, policy, or decision>` | Confirmed/Open/Deferred |

## Epic Template

### Epic: `<name>`

**Goal:** What this epic proves or enables.

| Story ID | Story | Requirement IDs | Owner Type | Acceptance Criteria | Notes |
|---|---|---|---|---|---|
| US-001 | As a `<user>`, I want `<capability>` so that `<outcome>` | FR-001, NFR-001 | Product/Engineering/Architecture/Security/Data/DevOps | Given/When/Then or checklist criteria | Dependencies, risks, or decisions |

Every implementation or validation story must reference at least one `FR-###` or `NFR-###`. Architecture, discovery, and decision work that does not implement a confirmed requirement must be explicitly labeled `ENABLER` or `DECISION` in the Requirement IDs column.

## Required Epic Set

Sequenced so the dev team can deploy and test incrementally: environment first, then data, then application code.

### 1. Product Discovery and Scope

Purpose: turn the idea into a testable POC boundary.

Expected stories:

- Define POC success criteria and non-goals.
- Identify primary users and top workflows.
- Confirm the functional and non-functional requirements that the backlog will implement.
- Confirm demo scenario and stakeholder signoff.

### 2. Azure Infrastructure and Deployment

Purpose: make the POC repeatable, not hand-built. Stand up the environment early so every subsequent epic can deploy and validate incrementally.

Expected stories:

- Create `infra/` Bicep modules for required Azure resources.
- Create `azure.yaml` mapping services to Azure resources.
- Support `azd provision`, `azd deploy`, and `azd up`.
- Document required environment variables and secret locations.

### 3. Private Networking and Security

Purpose: keep the database behind the right doors from the first sprint. Wire up networking and identity before data flows through the system.

Expected stories:

- Add VNet/subnet design for app-to-database connectivity.
- Add private endpoint and private DNS requirements for the selected database.
- Disable public network access where supported by the selected database service.
- Add managed identity and RBAC stories for service-to-service access.

### 4. API Contracts and Mocks

Purpose: lock the interface of **every API/service** in the system before any of them are fully built, so each consumer (frontend or another service) can progress against a stable contract and a mock while the real implementations are still in flight. Establish contracts right after the environment and networking foundation.

Expected stories:

- Inventory every API/service in the system and define an explicit, versioned contract for **each** (e.g. OpenAPI/Swagger for REST, schema for GraphQL/gRPC); commit each contract as the shared source of truth for that API.
- Implement a mock for **each** API that serves contract-valid, varied responses (including error and edge-case responses), not a single hardcoded payload.
- Provide a per-API configuration toggle (e.g. environment variable or build flag) so each consumer can target the mock or the true implementation of each upstream API independently, without code changes.
- Add contract validation/tests so each mock and its real API are both verified against the same contract and cannot drift, including service-to-service contracts.

### 5. Data and Integration

Purpose: connect the POC to real enough data without pretending the integration problem is solved. Runs against live infra from Epic 2–3 and fulfills the contracts from Epic 4.

Expected stories:

- Identify data sources and data contracts.
- Implement read paths and any required write/action paths.
- Add test data and data refresh expectations.

### 6. User Experience and Workflow

Purpose: define what the user sees, does, approves, and trusts. Finalize UX design before application implementation begins. Consumers (the frontend and any calling service) are built against the API contracts and can run against the mock or true implementation per API.

Expected stories:

- Map the current workflow and future POC workflow.
- Define conversation, UI, API, or autonomous behavior.
- Build each consumer against the committed API contracts, defaulting to the mocks and switchable to the true implementation per API via the configuration toggle.
- Define human approval and exception paths.

### 7. Application/API Implementation

Purpose: implement the core product behavior on top of the infrastructure, data, contracts, and UX foundations from Epics 2–6. Each real API must satisfy the same contract its mock implements.

Expected stories:

- Implement the minimum end-to-end user journey.
- Add orchestration, agent, or tool-calling behavior only where the scenario needs it.
- Capture failure states and user-facing recovery behavior.

> **Implementation integrity:** Stories must produce real behavior, not test-passing shortcuts. Do not hardcode expected outputs, stub responses, or special-case known test inputs just to make tests or acceptance criteria go green. Acceptance criteria must assert genuine behavior against realistic and varied inputs (including unseen ones), so a hardcoded answer would fail. If a real implementation is out of POC scope, mark it as an explicit decision or a deferred story rather than faking the result.

### 8. Observability and Validation

Purpose: prove the POC works and fails visibly.

Expected stories:

- Add application logging and correlation IDs.
- Add deployment validation steps.
- Add a demo validation script or checklist.
- Add tests that exercise real behavior against varied and previously unseen inputs, so an implementation that hardcodes outputs for known cases would fail.
- Capture known limitations and next-phase risks.

### 9. Deployment and Success Criteria Verification

Purpose: prove the deployed POC actually meets the agreed success criteria.

Expected stories:

- Run `azd up` end-to-end against a clean environment and capture the result.
- Execute a post-deploy smoke test confirming the workload is live.
- Verify each POC success criterion against the deployed system, not a local build.
- Validate private connectivity from the deployed host, not a developer machine.

### 10. Documentation of Architecture Decisions, Workflow Diagrams, and Operational Runbooks

Purpose: make the POC buildable by the next team and operable in production-shaped form. Capture decisions while they are fresh, before demo preparation.

Expected stories:

- Document architecture decisions and their recorded defaults.
- Commit architecture and workflow diagrams.
- Write an operational runbook (deploy, rotate secrets, redeploy, teardown).
- Document what must change before production.
- Any lesson learnt from fixing issue during deployment and testing.

### 11. POC Demo Script and Guide

Purpose: make the POC demonstrable on demand by any presenter. Final epic — everything it references is already built and documented.

Expected stories:

- Prepare a step-by-step demo script driving the primary user workflow.
- Document demo prerequisites, environment setup, and required fixtures.
- Capture expected outputs so a presenter can spot a broken demo.


## Requirements Traceability Matrix

Use this matrix after generating the backlog. Every requirement must map to implementation, validation, an explicit decision, or a documented deferral.

| Requirement ID | Requirement Summary | Backlog Story IDs | Coverage Status | Validation Evidence / Notes |
|---|---|---|---|---|
| FR-001 | `<functional behavior>` | US-001, US-004 | Covered/Decision/Deferred/Gap | `<how coverage will be proven>` |
| NFR-001 | `<quality target or constraint>` | US-003, US-009 | Covered/Decision/Deferred/Gap | `<measurement or validation>` |

Coverage rules:

- No confirmed `FR-###` or `NFR-###` may remain unmapped.
- No implementation or validation story may reference a requirement that is absent from the requirements sections.
- `Decision` and `Deferred` entries must identify an owner and next action in the backlog.
- Any `Gap` means the backlog is incomplete and must be revised before finalization.


## Definition of Done

- The POC scenario works end-to-end against real inputs, with no hardcoded outputs, stubbed responses, or test-input special-casing used to satisfy tests or acceptance criteria.
- Functional and non-functional requirements are documented with stable `FR-###` and `NFR-###` identifiers.
- Functional requirements define observable behavior and non-functional requirements define measurable targets or mandatory constraints.
- Every implementation and validation story references at least one requirement; non-requirement work is explicitly labeled as an enabler or decision.
- The traceability matrix maps every requirement to backlog stories, an owned decision, or an explicit deferral, with no remaining gaps.
- The backlog includes product, engineering, infrastructure, security, and validation work.
- Infrastructure is represented in Bicep.
- `azd` can provision and deploy the workload, or a story exists to close that gap.
- Database networking uses private endpoint/private DNS design, with public access disabled where supported.
- Every API/service has its own committed, versioned contract; each API ships a mock serving contract-valid responses; and every consumer can switch between mock and true implementation per API via configuration without code changes.
- Deployment is verified and each POC success criterion is checked against the deployed system.
- A demo script and an operational runbook are documented, with architecture decisions and workflow diagrams committed.
- Open decisions are explicit and assigned.
