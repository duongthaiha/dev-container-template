# Non-Functional Requirements Template

Use this template to define measurable quality and operational expectations before creating backlog stories.

## Writing Rules

- Assign stable identifiers in the form `NFR-001`, `NFR-002`, and so on.
- State a measurable target or threshold and how it will be verified.
- Apply requirements only to relevant POC scope; do not force every category into every backlog.
- Separate mandatory constraints from desired targets.
- Record an unknown threshold as a question or decision item rather than using vague terms such as "fast," "secure," or "scalable."
- Align Azure security, networking, identity, deployment, and observability requirements with the skill guardrails.

## Non-Functional Requirements

| ID | Quality Attribute | Scope | Target / Threshold | Measurement Method | Priority | Rationale | Dependencies | Status / Open Decision |
|---|---|---|---|---|---|---|---|---|
| NFR-001 | `<security, performance, reliability, etc.>` | `<workflow, component, API, data, or environment>` | `<measurable target or mandatory constraint>` | `<test, metric, audit, review, or deployment evidence>` | Must/Should/Could | `<why the target matters to the POC>` | `<service, requirement, policy, or decision>` | Confirmed/Open/Deferred |

## Quality Attribute Prompts

Use only the categories relevant to the POC:

| Category | Example Questions |
|---|---|
| Security and privacy | What identity, authorization, secret handling, data protection, and private-networking controls are mandatory? |
| Performance | Which operations have latency, throughput, concurrency, or processing-time targets? |
| Reliability and resiliency | What availability, retry, recovery, failure isolation, or data durability behavior must be proven? |
| Scalability | What load or growth boundary must the POC demonstrate without redesign? |
| Observability and auditability | Which logs, metrics, traces, correlation IDs, alerts, and audit records must exist? |
| Accessibility and usability | Which accessibility standard, supported device, or usability outcome applies? |
| Operability and supportability | What deployment, rollback, diagnostics, secret rotation, teardown, or runbook capability is required? |
| Deployment repeatability | What must `azd`, Bicep, validation, and clean-environment deployment prove? |
| Cost | What budget, consumption ceiling, idle-resource policy, or cost-reporting target applies? |
| Compliance and data governance | Which residency, retention, classification, approval, or regulatory constraints apply? |

## Non-Functional Coverage Check

Before generating the backlog, confirm:

- Every non-functional requirement has a target or explicit mandatory constraint.
- Every target has a named measurement method or acceptance evidence.
- Relevant Azure guardrails are represented as NFRs or explicit enabler/decision items.
- Vague or unknown quality expectations remain open questions or decisions.
- Each confirmed non-functional requirement can map to backlog acceptance criteria, an enabler story, or a validation story.

