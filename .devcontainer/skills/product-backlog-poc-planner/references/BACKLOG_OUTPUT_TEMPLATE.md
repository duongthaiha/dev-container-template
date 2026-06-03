# Backlog Output Template

Use this template when producing the final backlog.

## Assumptions

| Assumption | Why It Matters | Validation Needed |
|---|---|---|
| Example: Database service is not selected yet | Networking and Bicep modules depend on the service | Architecture decision |

## POC Goal

One paragraph that explains the thin slice. Keep it outcome-first: user, workflow, proof, and constraint.

## Epic Template

### Epic: `<name>`

**Goal:** What this epic proves or enables.

| Story | Owner Type | Acceptance Criteria | Notes |
|---|---|---|---|
| As a `<user>`, I want `<capability>` so that `<outcome>` | Product/Engineering/Architecture/Security/Data/DevOps | Given/When/Then or checklist criteria | Dependencies, risks, or decisions |

## Required Epic Set

### 1. Product Discovery and Scope

Purpose: turn the idea into a testable POC boundary.

Expected stories:

- Define POC success criteria and non-goals.
- Identify primary users and top workflows.
- Confirm demo scenario and stakeholder signoff.

### 2. User Experience and Workflow

Purpose: define what the user sees, does, approves, and trusts.

Expected stories:

- Map the current workflow and future POC workflow.
- Define conversation, UI, API, or autonomous behavior.
- Define human approval and exception paths.

### 3. Application/API Implementation

Purpose: implement the core product behavior.

Expected stories:

- Implement the minimum end-to-end user journey.
- Add orchestration, agent, or tool-calling behavior only where the scenario needs it.
- Capture failure states and user-facing recovery behavior.

### 4. Data and Integration

Purpose: connect the POC to real enough data without pretending the integration problem is solved.

Expected stories:

- Identify data sources and data contracts.
- Implement read paths and any required write/action paths.
- Add test data and data refresh expectations.

### 5. Azure Infrastructure and Deployment

Purpose: make the POC repeatable, not hand-built.

Expected stories:

- Create `infra/` Bicep modules for required Azure resources.
- Create `azure.yaml` mapping services to Azure resources.
- Support `azd provision`, `azd deploy`, and `azd up`.
- Document required environment variables and secret locations.

### 6. Private Networking and Security

Purpose: keep the database behind the right doors from the first sprint.

Expected stories:

- Add VNet/subnet design for app-to-database connectivity.
- Add private endpoint and private DNS requirements for the selected database.
- Disable public network access where supported by the selected database service.
- Add managed identity and RBAC stories for service-to-service access.

### 7. Observability and Validation

Purpose: prove the POC works and fails visibly.

Expected stories:

- Add application logging and correlation IDs.
- Add deployment validation steps.
- Add a demo validation script or checklist.
- Capture known limitations and next-phase risks.

### 8. Deployment and Success Criteria Verification

Purpose: prove the deployed POC actually meets the agreed success criteria.

Expected stories:

- Run `azd up` end-to-end against a clean environment and capture the result.
- Execute a post-deploy smoke test confirming the workload is live.
- Verify each POC success criterion against the deployed system, not a local build.
- Validate private connectivity from the deployed host, not a developer machine.

### 9. POC Demo Script and Guide

Purpose: make the POC demonstrable on demand by any presenter.

Expected stories:

- Prepare a step-by-step demo script driving the primary user workflow.
- Document demo prerequisites, environment setup, and required fixtures.
- Capture expected outputs so a presenter can spot a broken demo.

### 10. Documentation of Architecture Decisions, Workflow Diagrams, and Operational Runbooks

Purpose: make the POC buildable by the next team and operable in production-shaped form.

Expected stories:

- Document architecture decisions and their recorded defaults.
- Commit architecture and workflow diagrams.
- Write an operational runbook (deploy, rotate secrets, redeploy, teardown).
- Document what must change before production.
- Any lesson learnt from fixing issue during deployment and testing.



## Definition of Done

- The POC scenario works end-to-end.
- The backlog includes product, engineering, infrastructure, security, and validation work.
- Infrastructure is represented in Bicep.
- `azd` can provision and deploy the workload, or a story exists to close that gap.
- Database networking uses private endpoint/private DNS design, with public access disabled where supported.
- Deployment is verified and each POC success criterion is checked against the deployed system.
- A demo script and an operational runbook are documented, with architecture decisions and workflow diagrams committed.
- Open decisions are explicit and assigned.
