# Migration Workload Lifecycle

Every migration workload follows this lifecycle in order. Evidence must be
recorded before a workload advances to its allowed next state.

| State | Entry condition | Required evidence | Allowed next states | Human approval required |
| --- | --- | --- | --- | --- |
| `discovered` | A candidate repository and migration need have been identified. | Repository identifier, current runtime version, target runtime version, and workload owner. | `readiness_assessed` | No |
| `readiness_assessed` | Discovery information is complete and the repository is available for assessment. | Readiness score, assessment report, identified blockers, and assessment timestamp. | `readiness_remediated` | No |
| `readiness_remediated` | Readiness blockers and gaps have been identified from the assessment. | Remediation changes or decisions, resolved blocker list, and updated readiness evidence. | `spec_defined` | No |
| `spec_defined` | Readiness remediation is complete and the migration scope can be defined. | Approved scope, target requirements, constraints, assumptions, and acceptance criteria. | `plan_created` | No |
| `plan_created` | The migration specification is complete. | Ordered implementation plan, dependency changes, rollback approach, ownership, and estimated effort. | `validation_defined` | No |
| `validation_defined` | The implementation plan is complete and testable. | Validation plan covering builds, tests, compatibility checks, runtime checks, and success criteria. | `human_approved` | No |
| `human_approved` | The specification, implementation plan, and validation plan are ready for review. | Recorded human approval with approver identity, timestamp, and approved scope. | `executing` | Yes |
| `executing` | Human approval has been recorded and execution has been authorized. | Implementation changes, execution log, and results from required developer checks. | `independent_review` | No |
| `independent_review` | Execution is complete and all planned changes are available for independent inspection. | Independent review report, findings, and disposition of each finding. | `pr_open` | No |
| `pr_open` | Independent review is complete and all blocking findings are resolved. | Pull request URL, change summary, linked evidence, and reviewer assignments. | `ci_validated` | No |
| `ci_validated` | A pull request is open and its required CI checks have run. | Successful required CI checks, test reports, and any approved exception records. | `promotion_gate` | No |
| `promotion_gate` | Independent review and remote CI are complete and all blocking findings are resolved. | Gate checklist showing specification compliance, remote CI results, validation results, risk status, and rollback readiness. | `ready_for_human_merge` | No |
| `ready_for_human_merge` | The promotion gate has passed and review requirements are satisfied. | Final merge checklist, required reviewer approvals, and confirmation that no blocking findings remain. | `completed` | Yes, a human must authorize the merge |
| `completed` | A human-authorized merge is complete and post-merge verification has succeeded. | Merge reference, completion timestamp, post-merge verification results, and final readiness score. | None | Yes |

## Transition Rule

A workload may not skip directly from `executing` to `completed`. It must pass
through `independent_review`, `pr_open`, `ci_validated`, `promotion_gate`, and
`ready_for_human_merge`.
