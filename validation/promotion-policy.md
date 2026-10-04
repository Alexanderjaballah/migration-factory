# Migration Factory Promotion Policy

> **Agent confidence does not ship code. Evidence does.**

A migration workload may pass the factory promotion gate only when the evidence
below is complete, current, attributable to the reviewed change, and available
for inspection. Assertions without supporting artifacts do not count as
evidence.

## Required Checks

| Check | Minimum passing evidence |
| --- | --- |
| Target runtime/build succeeds | A successful clean build using the target runtime, including the command, runtime version, timestamp, and build output or report. |
| Baseline tests still pass | Results showing that the pre-existing test suite passes against the migration change. |
| Migration-specific tests pass | Results from tests that exercise the migration's changed behavior, compatibility requirements, and acceptance criteria. |
| No tests are weakened, skipped, or bypassed | A reviewed test diff and test report showing no removed assertions, reduced coverage expectations, new skips, exclusions, bypasses, or equivalent weakening. Any intentional test change must strengthen or preserve validation and include a documented rationale. |
| Dependency/build changes are explained | A complete diff of dependency and build configuration changes, with the purpose, compatibility impact, and risk of each change documented. |
| No unrelated scope is introduced | A scope review mapping every changed file to the approved migration specification, with no unrelated feature, refactor, or cleanup work. |
| Independent review has no P0/P1 findings | An independent review report showing zero open P0 or P1 findings. P0 and P1 findings cannot be waived at this gate. |
| CI passes remotely | A link to the remote CI run for the exact candidate revision, showing every required job completed successfully. |
| Security findings are clear or explicitly accepted | A security review showing no open findings, or a written acceptance for each remaining non-P0/P1 finding that names the risk owner, rationale, scope, compensating controls, and expiration or remediation date. |
| Human approval is required before merge | A recorded approval from an authorized human that identifies the exact candidate revision. The workload must not merge without it. |

Evidence from a different revision, incomplete runs, local-only CI substitutes,
or undocumented exceptions does not satisfy a required check.

## Decision Model

The promotion gate has exactly three possible outcomes.

### `PROMOTE`

Applies only when every required check has passing evidence, no security
findings remain open, no conditions or exceptions remain, and the required
human approval is recorded for the exact candidate revision.

### `PROMOTE_WITH_CONDITIONS`

Applies only when every required build, test, scope, independent-review, and
remote-CI check has passing evidence; there are no open P0 or P1 findings; and
the only remaining items are explicitly accepted non-P0/P1 security findings
or other non-blocking obligations. Every condition must have a named owner,
documented rationale, due or expiration date, and recorded human approval for
the exact candidate revision and its conditions.

### `DO_NOT_PROMOTE`

Applies whenever the requirements for `PROMOTE` or
`PROMOTE_WITH_CONDITIONS` are not fully met. This includes missing or stale
evidence, any failed required build or test, weakened or bypassed tests,
unexplained dependency or build changes, unrelated scope, failed remote CI, an
open P0 or P1 finding, an unaccepted security finding, or missing human
approval.
