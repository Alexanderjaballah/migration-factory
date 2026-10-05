# Service B Java 21 Migration Mission

## Mission

Design and deliver the bounded migration of `service-b` from Java 17 to Java
21, with evidence sufficient for independent review and the factory promotion
gate.

This document is a plan only. At authoring time (2026-10-04, when first
drafted), no task had been executed, no repository had been cloned, no
Factory or Droid session had been started, and Service B had not been
modified; readiness has since been verified — see `metrics/` readiness
evidence.

## Governing Inputs

- `repos/registry.yaml`
- `orchestration/lifecycle.md`
- `orchestration/task-routing-policy.md`
- `policies/autonomy-policy.md`
- `validation/promotion-policy.md`
- `metrics/factory-metrics.md`

Registry facts at mission-design time:

| Field | Value |
| --- | --- |
| Workload | `service-b` |
| Repository | `git@github.com:Alexanderjaballah/tenantflow-saas-api.git` |
| Migration | Java 17 to Java 21 |
| Registered complexity | Medium |
| Registered risk | Medium |
| Lifecycle state | `readiness_assessed` |
| Readiness baseline | 32.4 |

The runtime upgrade makes the initial effective risk Medium. Authentication,
schema/data migration, or production infrastructure changes discovered during
analysis raise the affected task to High and may raise the remaining workload
controls to High. Agents may not lower that classification.

## Architecture Decision

The approved lifecycle order is `independent_review` → `pr_open` →
`ci_validated` → `promotion_gate` → `ready_for_human_merge`. This gives the
promotion gate remote CI evidence for the exact candidate revision, as required
by `validation/promotion-policy.md`.

## Dependency Graph

```text
T01 Repository and build analysis
 |
+---------------- Fan-out A ----------------+
|                 |             |           |
T02               T03           T04         T05
Readiness gaps    Dependencies  Security    Database/Flyway
|                 |             |           |
+----------------- Fan-in A ----------------+
                  |
          +-------+-------+
          |               |
         T06             T07
 Readiness remediation  Test strategy
          |               |
          +--- Fan-in B ---+
                  |
                 T08
          Migration planning
                  |
       G1 Human execution approval
                  |
                 T09
            Implementation
                  |
          +-------+-------+
          |               |
         T10             T11
 Build/test validation  Runtime/container validation
          |               |
          +--- Fan-in C ---+
                  |
                 T12
          Independent review
                  |
     G2 Human PR approval if High
                  |
                 T13
             PR creation
                  |
                 T14
             Remote CI
                  |
                 T15
          Final promotion gate
                  |
                 T16
        Metrics and handoff record
```

Fan-out A contains read-only analyses with separate evidence outputs. Fan-in A
must reconcile their conclusions before remediation and test design. T06 and
T07 may then run in parallel because they produce separate planning artifacts;
Fan-in B combines them into one migration plan. After implementation, T10 and
T11 may run in parallel only against the same immutable candidate revision and
only if both are validation-only. Fan-in C must collect both result sets before
independent review.

Implementation tasks are intentionally serialized. They are likely to touch
shared build files, dependency declarations, source, tests, container
configuration, or CI configuration.

## Tasks

### T01 — Repository and Build Analysis

- **Objective:** Establish the authoritative build, module, runtime, test,
  container, CI, authentication, and database migration inventory.
- **Dependencies:** None.
- **Parallel:** No; it creates the shared inventory for Fan-out A.
- **Expected files/components affected:** None; read-only inspection of build
  descriptors, wrapper/toolchain configuration, modules, source layout, test
  configuration, container definitions, CI workflows, security configuration,
  and migration directories.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** Approved repository revision, registry entry,
  prior readiness assessment, repository instructions, and build/CI
  documentation.
- **Required output/evidence:** Revision-pinned inventory, baseline build and
  test commands, affected-component map, and unknowns list. Inspection alone
  must not alter files.
- **Failure/escalation:** Context failures require authoritative repository or
  build documentation. Environment or permission failures go to the
  repository owner. Repeated analytical failure escalates to
  `frontier_reasoning`.
- **Human gate:** Repository access approval if not already granted; no
  execution approval yet.

### T02 — Readiness Gap Analysis

- **Objective:** Convert the 32.4 readiness baseline into an evidence-backed
  list of blockers and remediation requirements.
- **Dependencies:** T01.
- **Parallel:** Yes, in Fan-out A with T03, T04, and T05.
- **Expected files/components affected:** None; read-only readiness report,
  build inventory, repository instructions, and validation configuration.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** T01 inventory, original readiness assessment,
  scoring method, known blockers, and target-runtime requirements.
- **Required output/evidence:** Prioritized blocker register, remediation
  criteria, evidence gaps, and proposed post-remediation reassessment method.
- **Failure/escalation:** Missing assessment evidence is a context failure and
  must be obtained rather than inferred. Ambiguous blockers escalate to
  `frontier_reasoning`, then to a human if unresolved.
- **Human gate:** Human decision required for blockers that change scope,
  architecture, or risk.

### T03 — Dependency Compatibility Analysis

- **Objective:** Determine whether direct dependencies, plugins, build tools,
  and transitive constraints support the target runtime.
- **Dependencies:** T01.
- **Parallel:** Yes, in Fan-out A with T02, T04, and T05.
- **Expected files/components affected:** None; read-only dependency manifests,
  lock or resolution data, build plugins, and dependency reports.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** T01 build inventory, resolved dependency graph,
  release and compatibility evidence, and repository constraints.
- **Required output/evidence:** Compatibility matrix, required and optional
  changes, rejected alternatives, rationale, risk, and validation needed for
  each proposed dependency/build change.
- **Failure/escalation:** Unresolvable compatibility or conflicting vendor
  evidence escalates to `frontier_reasoning`. A major framework or API
  migration becomes `C4`; architecture or scope decisions escalate to a human.
- **Human gate:** Required if the proposed dependency path changes approved
  scope, architecture, or risk.

### T04 — Security and Authentication Impact Analysis

- **Objective:** Determine whether the runtime or dependency migration changes
  authentication, authorization, cryptography, secrets handling, security
  defaults, or exposed interfaces.
- **Dependencies:** T01.
- **Parallel:** Yes, in Fan-out A with T02, T03, and T05.
- **Expected files/components affected:** None; read-only security
  configuration, authentication/authorization code, dependency evidence,
  interface definitions, and security tests.
- **Complexity:** `C4` — Cross-component.
- **Risk:** Medium — read-only analysis only. Escalates to High
  automatically if any authentication, schema/data, destructive database,
  or production-infrastructure IMPLEMENTATION change becomes required.
- **Note:** Reclassification from High to Medium (read-only scope)
  human-approved on 2026-10-05; High controls re-apply immediately if the
  read-only boundary is crossed.
- **Recommended model class:** `frontier_reasoning`.
- **Required inputs/context:** T01 inventory, approved threat model or security
  requirements, authentication flows, security advisories, and dependency
  metadata captured by T01. T03 conclusions are reconciled at Fan-in A rather
  than consumed while T03 is running.
- **Required output/evidence:** Impact assessment, changed-defaults analysis,
  required security tests, findings with severity, and explicit statement of
  whether implementation touches authentication or other High-risk scope.
- **Failure/escalation:** Missing threat or authentication context is a context
  failure. Any proposed security behavior change, risk acceptance, or
  unresolved finding escalates to the security owner and a human.
- **Human gate:** Required for security risk acceptance and before execution of
  any authentication-related change.

### T05 — Database and Flyway Impact Analysis

- **Objective:** Determine whether the migration affects database drivers,
  persistence behavior, Flyway compatibility, schema validation, or migration
  execution.
- **Dependencies:** T01.
- **Parallel:** Yes, in Fan-out A with T02, T03, and T04.
- **Expected files/components affected:** None; read-only persistence
  configuration, database dependencies, schema definitions, Flyway
  configuration and migrations, and database tests.
- **Complexity:** `C4` — Cross-component.
- **Risk:** Medium — read-only analysis only. Escalates to High
  automatically if any authentication, schema/data, destructive database,
  or production-infrastructure IMPLEMENTATION change becomes required.
- **Note:** Reclassification from High to Medium (read-only scope)
  human-approved on 2026-10-05; High controls re-apply immediately if the
  read-only boundary is crossed.
- **Recommended model class:** `frontier_reasoning`.
- **Required inputs/context:** T01 inventory, supported database versions,
  migration history, driver/Flyway compatibility evidence, rollback
  constraints, and database test environment.
- **Required output/evidence:** Compatibility assessment, schema/data impact
  statement, required validation matrix, rollback concerns, and explicit
  statement of whether implementation changes migrations or production data.
- **Failure/escalation:** Missing database or migration history is a context
  failure. Any schema/data change, destructive behavior, uncertain rollback,
  or production-data risk escalates to database and human owners.
- **Human gate:** Required before any schema/data migration or production data
  change is included.

### T06 — Readiness Remediation

- **Objective:** Resolve or disposition every readiness blocker before the
  migration plan is approved.
- **Dependencies:** Fan-in A: T02, T03, T04, and T05.
- **Parallel:** Yes, with T07 after Fan-in A; no parallel modification of
  shared repository files is allowed.
- **Expected files/components affected:** Factory readiness artifacts only
  during planning. Any Service B remediation is deferred to T09 after approval.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium by default; High for remediation involving authentication,
  schema/data, or production infrastructure.
- **Recommended model class:** `general_engineering`, escalating to
  `frontier_reasoning` for cross-component blockers.
- **Required inputs/context:** All Fan-in A evidence, readiness scoring method,
  accepted scope, and owners for blocked decisions.
- **Required output/evidence:** Closed or explicitly escalated blocker list,
  remediation specification, reassessment evidence, and an updated readiness
  score only if actually measured.
- **Failure/escalation:** Planning failures return to the responsible analysis
  task. Scope expansion, unresolved High-risk blockers, or C5 decisions
  escalate to a human.
- **Human gate:** Required for scope, architecture, risk, or exception
  decisions.

### T07 — Independent Test Strategy

- **Objective:** Define validation before implementation, covering baseline
  behavior and migration-specific risks without weakening tests.
- **Dependencies:** Fan-in A: T02, T03, T04, and T05.
- **Parallel:** Yes, with T06 after Fan-in A.
- **Expected files/components affected:** Factory validation plan only; no
  Service B files during this task.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium, raised to High for authentication, schema/data, or
  production infrastructure validation.
- **Recommended model class:** `independent_reviewer` in fresh context where
  practical.
- **Required inputs/context:** Baseline commands and results, analysis
  findings, acceptance criteria, promotion policy, test environments, and CI
  requirements.
- **Required output/evidence:** Independent validation plan for clean target
  build, baseline tests, migration-specific tests, security/authentication,
  database/Flyway, runtime/container, remote CI, test-diff review, and exact
  candidate-revision attribution.
- **Failure/escalation:** Validation gaps return to the relevant analysis task.
  Tests must not be skipped, bypassed, or weakened. Unresolvable environment or
  coverage gaps escalate to a human owner.
- **Human gate:** Approval is included with the Medium-risk execution gate at
  T08.

### T08 — Migration Planning and Execution Approval

- **Objective:** Produce one bounded implementation plan and obtain approval
  before execution.
- **Dependencies:** Fan-in B: T06 and T07.
- **Parallel:** No; this integrates all prior evidence.
- **Expected files/components affected:** Factory specification, plan,
  validation plan, file ownership map, and rollback plan only.
- **Complexity:** `C4` — Cross-component.
- **Risk:** Medium unless prior analysis raises the effective risk to High.
- **Recommended model class:** `frontier_reasoning`.
- **Required inputs/context:** All prior task outputs, exact repository
  revision, autonomy policy, task-routing policy, lifecycle, and promotion
  requirements.
- **Required output/evidence:** Approved scope, ordered change list,
  dependency rationale, file/component ownership, validation commands,
  rollback plan, risk classification, effort estimate labeled as an estimate,
  and explicit exclusions.
- **Failure/escalation:** Conflicts return to the producing task. C5 decisions,
  unresolved risk, or scope ambiguity escalate to a human and block execution.
- **Human gate:** Required before T09. If risk is High, approval must be
  explicit and execution permissions must be constrained.

### T09 — Migration Implementation

- **Objective:** Implement only the approved Java 21 migration scope and its
  required tests.
- **Dependencies:** T08 human approval.
- **Parallel:** No; implementation is serialized to avoid collisions in
  shared build, dependency, source, test, container, and CI files.
- **Expected files/components affected:** Only the exact Service B files and
  components listed in the approved T08 file ownership map.
- **Complexity:** `C4` — Cross-component.
- **Risk:** Medium unless implementation includes authentication, schema/data,
  or production infrastructure, in which case it is High.
- **Recommended model class:** `frontier_reasoning`.
- **Required inputs/context:** Approved plan and revision, repository
  instructions, analysis matrices, validation plan, constrained permissions,
  and rollback approach.
- **Required output/evidence:** Candidate diff, change log mapped to plan
  steps, dependency/build rationale, new or updated tests, commands run,
  attempt history, and explicit confirmation that no unrelated scope was
  introduced.
- **Failure/escalation:** Bounded implementation failures may be retried with a
  changed hypothesis and recorded evidence. Repeated failure escalates model
  class or returns to planning. Scope expansion, architecture decisions, and
  unresolved High-risk changes escalate to a human.
- **Human gate:** T08 approval is mandatory before starting. New out-of-scope
  work requires a new approval.

### T10 — Target Build and Test Validation

- **Objective:** Validate the immutable candidate with a clean target-runtime
  build, baseline tests, and migration-specific tests.
- **Dependencies:** T09.
- **Parallel:** Yes, in validation fan-out with T11, if both use the same
  candidate revision and do not modify shared files.
- **Expected files/components affected:** None; validation outputs and reports
  only.
- **Complexity:** `C2` — Bounded.
- **Risk:** Medium.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** Candidate revision, T07 validation plan,
  baseline results, target toolchain, and test commands.
- **Required output/evidence:** Clean build output, runtime version, timestamp,
  passed/failed test counts, baseline and migration-specific results, and
  reviewed test diff proving no skip, bypass, or weakening.
- **Failure/escalation:** Validation failure returns to T09 with evidence.
  Environment failures go to the environment owner. Repeated unexplained
  failures escalate to `frontier_reasoning`; tests may not be weakened.
- **Human gate:** Required only if remediation changes approved scope or risk.

### T11 — Runtime and Container Validation

- **Objective:** Verify startup, runtime behavior, health checks, packaging,
  and container compatibility on the target runtime.
- **Dependencies:** T09.
- **Parallel:** Yes, in validation fan-out with T10 under the same immutable
  candidate constraint.
- **Expected files/components affected:** None; runtime logs, container
  metadata, and validation reports only.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium; High if production infrastructure changes are proposed.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** Candidate revision, runtime/container portion
  of T07, approved environment, startup contract, health checks, and relevant
  service dependency configuration.
- **Required output/evidence:** Target-runtime identity, image/build evidence,
  startup and health results, smoke-test results, logs, and configuration-diff
  review.
- **Failure/escalation:** Runtime failures return to T09. Environment failures
  go to the platform owner. Any production infrastructure change raises risk
  to High and requires human approval before implementation.
- **Human gate:** Required for production infrastructure changes or expanded
  environment permissions.

### T12 — Independent Review

- **Objective:** Review the integrated candidate and all validation evidence
  independently of implementation.
- **Dependencies:** Fan-in C: T10 and T11.
- **Parallel:** No; review requires the complete integrated evidence set.
- **Expected files/components affected:** None; review findings and report
  only.
- **Complexity:** `C4` — Cross-component.
- **Risk:** Medium by default; High if the candidate's effective risk has been
  raised.
- **Recommended model class:** `independent_reviewer`, using a different model
  and fresh context where practical.
- **Required inputs/context:** Approved plan, candidate diff, dependency
  explanations, test diff, all local validation, security/database analyses,
  and scope map.
- **Required output/evidence:** Independent report, P0/P1/P2 counts, finding
  evidence and disposition, scope-compliance result, and confirmation that no
  tests were weakened.
- **Failure/escalation:** P0/P1 findings block progress and return to T09 after
  human triage. Other unresolved findings require evidence-backed disposition.
  Security risk acceptance and disputed findings escalate to a human.
- **Human gate:** Required for risk acceptance or changes to approved scope;
  Medium-risk independent review must complete before promotion.

### T13 — Pull Request Creation

- **Objective:** Open a pull request for the exact reviewed candidate with all
  available evidence linked.
- **Dependencies:** T12.
- **Parallel:** No.
- **Expected files/components affected:** No additional Service B changes; pull
  request metadata only.
- **Complexity:** `C2` — Bounded.
- **Risk:** Medium by default; High if the candidate's effective risk has been
  raised.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** Reviewed candidate revision, approved scope,
  change summary, validation results, independent review, rollback plan, and
  repository contribution rules.
- **Required output/evidence:** Pull request URL, exact revision, linked
  evidence, reviewer assignments, and scope summary.
- **Failure/escalation:** Permission failures go to a human repository owner.
  Any candidate change invalidates prior revision-specific evidence and returns
  to the affected validation and review tasks.
- **Human gate:** Required before PR creation if effective risk is High.

### T14 — Remote CI Validation

- **Objective:** Obtain required remote CI results for the exact pull-request
  candidate.
- **Dependencies:** T13.
- **Parallel:** No at the task level; required CI jobs may run in parallel
  according to repository configuration, and this task fans them back into one
  CI result.
- **Expected files/components affected:** None; remote CI records only.
- **Complexity:** `C2` — Bounded.
- **Risk:** Medium by default; High if the candidate's effective risk has been
  raised.
- **Recommended model class:** `general_engineering`.
- **Required inputs/context:** Pull request revision, required-check policy,
  remote CI configuration, and expected validation matrix.
- **Required output/evidence:** CI URL, exact revision, status of every
  required job, test reports, and approved exception records if permitted.
- **Failure/escalation:** Product failures return to T09; flaky validation is
  recorded and retried only under policy; environment failures go to the CI
  owner. Required jobs may not be bypassed.
- **Human gate:** Required for any allowed exception; missing required CI
  evidence blocks promotion.

### T15 — Final Promotion Gate

- **Objective:** Apply `validation/promotion-policy.md` to the exact
  remote-validated candidate.
- **Dependencies:** T14 and completed T12 independent review.
- **Parallel:** No; all evidence must fan in before the decision.
- **Expected files/components affected:** None; promotion decision record only.
- **Complexity:** `C3` — Reasoning-heavy.
- **Risk:** Medium by default; High if the candidate's effective risk has been
  raised.
- **Recommended model class:** `independent_reviewer`.
- **Required inputs/context:** Clean target build, baseline and
  migration-specific tests, test-diff review, dependency/build explanations,
  scope review, independent findings, remote CI, security disposition, and
  exact-revision human approval.
- **Required output/evidence:** Exactly one decision:
  `PROMOTE`, `PROMOTE_WITH_CONDITIONS`, or `DO_NOT_PROMOTE`, with the complete
  evidence checklist and any condition owner and due date.
- **Failure/escalation:** Missing, stale, failed, or revision-mismatched
  evidence yields `DO_NOT_PROMOTE`. P0/P1 findings cannot be waived. Security
  acceptance, disputed evidence, or unresolved conditions escalate to a human.
- **Human gate:** Recorded human approval for the exact candidate and any
  conditions is required; a separate human merge action remains required.

### T16 — Metrics and Human-Merge Handoff

- **Objective:** Record observed mission results after every promotion-gate
  decision and, for `PROMOTE` or `PROMOTE_WITH_CONDITIONS`, prepare the
  `ready_for_human_merge` handoff without merging.
- **Dependencies:** T15 with any promotion decision: `PROMOTE`,
  `PROMOTE_WITH_CONDITIONS`, or `DO_NOT_PROMOTE`.
- **Parallel:** No.
- **Expected files/components affected:** Factory workload result and evidence
  records only; no Service B modification.
- **Complexity:** `C1` — Mechanical.
- **Risk:** Medium by default; High if the candidate's effective risk has been
  raised.
- **Recommended model class:** `efficient`.
- **Required inputs/context:** Session and runtime records, retry history,
  build/test/review/CI results, human gates and interventions, cost data if
  available, and promotion decision.
- **Required output/evidence:** Workload status and lifecycle state, total
  agent runtime, retries, session count, quality results, finding counts,
  approval/intervention records, and available economics fields. Estimates
  must be labeled; unavailable values remain unavailable. A
  `DO_NOT_PROMOTE` outcome must still produce the workload result record,
  including the failure decision, its reasons, and the metrics — the
  factory measures failures too. On `DO_NOT_PROMOTE` there is no merge
  handoff and the workload terminates at `promotion_gate`.
- **Failure/escalation:** Missing observed data remains unavailable and must
  not be invented. Inconsistent success evidence returns to T15. Metrics
  definition disputes escalate to the factory owner.
- **Human gate:** Human approval is required before merge. This task does not
  merge or mark the workload `completed`.

## Mission Completion Criteria

The mission reaches `ready_for_human_merge` only when:

- The approved implementation scope is complete.
- Target-runtime build and baseline and migration-specific tests pass.
- No test is weakened, skipped, or bypassed.
- Runtime and container validation pass.
- Dependency and build changes are explained.
- No unrelated scope is present.
- Independent review has no open P0/P1 finding.
- Remote CI passes for the exact candidate revision.
- Security findings are clear or explicitly accepted under policy.
- The final promotion decision is `PROMOTE` or
  `PROMOTE_WITH_CONDITIONS`.
- Required human approvals are recorded.
- Required workload metrics contain observed or clearly labeled estimated
  values only.

Merge and transition to `completed` remain human-controlled and are outside
this mission's autonomous execution scope.
