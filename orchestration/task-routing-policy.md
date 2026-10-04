# Migration Factory Task Routing Policy

> **Risk determines autonomy. Complexity determines model class. Dependencies determine parallelism. Evidence determines promotion.**

This policy defines how a migration workload is decomposed and routed. It uses
the Low, Medium, and High risk controls in
`policies/autonomy-policy.md`. Routing decisions must not weaken those
controls.

## Task Decomposition

Before execution, the workload must be decomposed into tasks with:

- A single, bounded objective
- Explicit inputs and expected outputs
- Owned files, modules, or configuration
- Acceptance and validation criteria
- Dependencies on other tasks
- A complexity class
- A risk class
- A model class
- Required approvals and escalation conditions

Tasks should be small enough to validate independently but large enough to
produce a meaningful result. Discovery, planning, implementation, validation,
review, and promotion evidence must remain distinguishable, even when some of
them are performed in one session.

## Complexity Classes

| Class | Definition | Typical work | Default model class |
| --- | --- | --- | --- |
| `C1` — Mechanical | Deterministic work with a clear transformation and little interpretation. | Documentation, version references, formatting, deterministic edits. | `efficient` |
| `C2` — Bounded | Well-scoped engineering work with known patterns and limited dependencies. | Dependency updates, configuration changes, straightforward tests. | `general_engineering` |
| `C3` — Reasoning-heavy | Work requiring diagnosis, trade-off analysis, or iterative remediation within a contained area. | Debugging, compatibility remediation, contained refactoring. | `general_engineering`, escalating to `frontier_reasoning` when needed |
| `C4` — Cross-component | Work spanning components or requiring coordinated interface changes. | Framework migrations, API changes, work across multiple modules. | `frontier_reasoning` |
| `C5` — Architectural | Ambiguous or cross-repository work requiring architectural decisions. | Architecture changes, broad redesigns, and cross-repository coordination. | `frontier_reasoning` for analysis; human architectural approval before execution |

The highest complexity required by any essential part of a task determines its
class. A task must be split when it combines independently executable work with
materially different complexity.

## Risk Classes

Each task receives a Low, Medium, or High risk class using
`policies/autonomy-policy.md`.

| Risk | Routing effect |
| --- | --- |
| Low | The agent may plan, execute, validate, and open a pull request. Human approval is required before merge. |
| Medium | Human approval is required before execution, an independent validation plan is required, independent review must occur before promotion, and human approval is required before merge. |
| High | An explicit human-approved plan, constrained execution permissions, independent validation, security review, human approval before pull request creation, and human approval before merge are required. |

A task's effective risk is the higher of its own risk and the workload's
assigned risk. Agents may raise risk when new evidence warrants it, but they
must not lower risk or remove controls without human approval.

## Model Classes

- `efficient`: Performs deterministic, low-ambiguity work where the expected
  transformation and validation are explicit.
- `general_engineering`: Performs bounded software engineering, testing,
  debugging, and remediation that require normal codebase reasoning.
- `frontier_reasoning`: Handles high ambiguity, deep diagnosis,
  cross-component reasoning, and architectural analysis.
- `independent_reviewer`: Reviews completed work from a fresh perspective and
  evaluates scope, correctness, validation, security implications, and
  promotion evidence.

Prefer Factory Router by default. Complexity determines the minimum model
capability; risk determines autonomy and approval requirements, not model
capability. Independent review should use a different model and fresh context
where practical. A reviewer must not rely only on the executing agent's
conclusions.

## Parallelism

Tasks may run in parallel only when all of the following are true:

- They have no unmet dependency on one another.
- They do not modify the same files or shared configuration.
- Neither task consumes an unstable output from the other.
- Their validation can be performed independently.
- Parallel execution does not violate a risk control or approval gate.

Tasks must run sequentially when one task produces an input, decision, schema,
interface, configuration, or validation baseline required by another. Tasks
that are likely to modify the same files, shared build configuration, lock
files, generated artifacts, or common interfaces must also be serialized.

The factory may use a fan-out / fan-in pattern:

1. Fan out independent discovery, analysis, implementation, or validation
   tasks.
2. Collect every result and its evidence at a defined synchronization point.
3. Resolve conflicts, failures, and inconsistent conclusions.
4. Fan in to one integrated result.
5. Run integration validation and independent review on the combined change.

Parallel success does not establish integration success. Promotion relies on
evidence from the integrated candidate revision.

## Escalation

Execution may escalate from `efficient` to `general_engineering`, or from
`general_engineering` to `frontier_reasoning`, when:

- A task fails repeatedly despite a valid environment and clear inputs.
- The observed complexity exceeds the assigned class.
- Diagnosis requires broader context or non-obvious trade-offs.
- Conflicting evidence cannot be resolved at the current model class.

Escalation must preserve the original scope, evidence, and attempt history. A
stronger model is not permission to expand scope or bypass approval gates.

Execution must escalate to a human when:

- A `C5` task requires an architectural decision.
- The work exceeds approved scope or risk.
- Required permissions or approvals are unavailable.
- Repeated failures remain unresolved after appropriate model escalation.
- Security, data integrity, production impact, or business trade-offs require
  risk acceptance.
- Independent findings cannot be resolved with objective evidence.

## Failure Recovery

| Failure category | Recovery path |
| --- | --- |
| Context | Identify missing, stale, or conflicting context; retrieve authoritative repository, specification, and policy evidence; restart with a corrected context package. Escalate to a human if authoritative context cannot be established. |
| Planning | Stop execution; revise decomposition, dependencies, acceptance criteria, and risk classification; obtain any newly required approval before resuming. Use `frontier_reasoning` for unresolved C3/C4 planning and human approval for C5 decisions. |
| Implementation | Preserve logs and the failing diff; diagnose within the approved scope; retry with a bounded correction. After repeated failure, escalate model class. Escalate unresolved or scope-expanding changes to a human. |
| Validation | Treat the workload as not validated; determine whether the defect is in the implementation, test, or validation plan; correct it without weakening, skipping, or bypassing checks; rerun the full affected validation. Escalate unresolved conflicts to an independent reviewer or human. |
| Environment | Stop retries that cannot change the outcome; record the environment, command, and error; restore or request the required toolchain, service, or capacity; rerun from a clean, known state. Escalate infrastructure changes or persistent failures to a human owner. |
| Permission | Stop the affected action; request the minimum required authorization from an authorized human. Never bypass, broaden, or reuse permissions for an unapproved purpose. |
| Coordination | Pause conflicting tasks; establish ownership and a synchronization point; serialize shared-file or shared-interface changes; reconcile outputs, then rerun integration validation and independent review. Escalate unresolved ownership or priority conflicts to a human. |

Retries must be bounded and recorded. Repeating the same action without new
evidence, a changed hypothesis, or a corrected environment is not a recovery
strategy.
