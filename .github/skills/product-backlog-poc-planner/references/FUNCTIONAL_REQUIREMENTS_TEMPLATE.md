# Functional Requirements Template

Use this template to define what the POC must do before creating backlog stories.

## Writing Rules

- Assign stable identifiers in the form `FR-001`, `FR-002`, and so on.
- Describe observable user or system behavior, not implementation tasks.
- Keep requirements implementation-neutral unless the user has confirmed a technology constraint.
- Make each requirement atomic enough to validate independently.
- Record unresolved behavior as a question or decision item rather than inventing an answer.
- Use acceptance evidence that can be demonstrated or tested with realistic, varied inputs.

## Functional Requirements

| ID | Capability | Actor | Trigger | Required Behavior | Inputs | Outputs | Business Rules | Priority | Acceptance Evidence | Dependencies | Status / Open Decision |
|---|---|---|---|---|---|---|---|---|---|---|---|
| FR-001 | `<capability>` | `<user, system, or service>` | `<event that starts the behavior>` | `<observable behavior the product must perform>` | `<required input or precondition>` | `<result, action, or state change>` | `<validation, authorization, approval, or processing rules>` | Must/Should/Could | `<demo, test, or measurable proof>` | `<other requirement, data source, API, or decision>` | Confirmed/Open/Deferred |

## Functional Coverage Check

Before generating the backlog, confirm:

- Every primary user workflow has at least one functional requirement.
- Required read, write, approval, exception, and recovery behaviors are represented.
- Each external integration has explicit inputs, outputs, and failure behavior.
- Functional requirements do not hide unresolved architecture or product decisions.
- Each confirmed functional requirement can map to one or more backlog stories.

