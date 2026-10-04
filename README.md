# Migration Factory

The Migration Factory industrializes autonomous engineering for repeatable
enterprise migrations.

> **Risk determines autonomy. Complexity determines model class. Dependencies
> determine parallelism. Evidence determines promotion.**

## Operating Model

- **Workload registry:** `repos/registry.yaml` records repositories, migration
  targets, risk, complexity, readiness, and lifecycle status.
- **Lifecycle:** `orchestration/lifecycle.md` defines the evidence-gated path
  from discovery through human-authorized completion.
- **Task decomposition and routing:**
  `orchestration/task-routing-policy.md` defines task boundaries,
  dependencies, parallelism, complexity classes, model classes, and
  escalation.
- **Autonomy policy:** `policies/autonomy-policy.md` sets agent permissions and
  human approval requirements by risk.
- **Promotion policy:** `validation/promotion-policy.md` defines the evidence
  required to promote a migration.
- **Metrics and economics:** `metrics/factory-metrics.md` defines workload and
  factory-level outcome, quality, human-involvement, and economic measures.

## Workloads

- **Service A** is the completed migration proof and is currently recorded as
  `ready_for_human_merge`.
- **Service B** is the first Factory Mission workload. Its Java 21 migration
  design is documented in
  `orchestration/service-b-java21-mission.md`; the Mission has not been
  executed.

Factory Missions and Droids provide execution and orchestration. This
repository defines their shared operating model rather than implementing a
custom controller.
