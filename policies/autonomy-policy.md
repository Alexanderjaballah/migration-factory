# Migration Factory Autonomy Policy

This policy defines how much autonomy agents receive based on workload risk.
The assigned tier applies to the approved scope of the workload.

## Principles

- **Risk determines autonomy.**
- **Humans own architecture and risk decisions.**
- **Agents may execute only within approved scope.**
- **Out-of-scope work requires escalation, not silent expansion.**

## Low Risk

Low-risk work includes:

- Documentation
- Test-only changes
- Formatting
- Minor non-functional cleanup

Agents may:

- Plan
- Execute
- Validate
- Open a pull request

Human approval is required only before merge.

## Medium Risk

Medium-risk work includes:

- Runtime upgrades
- Dependency migrations
- Contained configuration changes

Agents may:

- Plan
- Execute after human approval
- Validate
- Open a pull request

The following controls are required:

- An independent validation plan
- Human approval before execution
- Independent review before promotion
- Human approval before merge

## High Risk

High-risk work includes:

- Authentication
- Payments
- Schema or data migrations
- Production infrastructure

Agents may plan and execute only under the controls below. Execution must stay
within the explicitly approved plan and constrained permissions.

The following controls are required:

- An explicit, human-approved plan
- Constrained execution permissions
- Independent validation
- Security review
- Human approval before pull request creation
- Human approval before merge

## Escalation

If an agent discovers work outside the approved scope, a higher-risk condition,
or a required architectural or risk decision, it must stop the affected work
and escalate to a human. The agent must not broaden scope or lower the assigned
risk tier on its own.
