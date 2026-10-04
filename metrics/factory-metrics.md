# Migration Factory Metrics

The Migration Factory records the following minimum metrics for every
workload. Factory-level metrics are calculated from these workload results over
a stated reporting period.

## Principles

- **Measure outcomes, not token consumption.**
- **Estimates must be clearly labeled as estimates.**
- **A migration only counts as successful after remote CI and promotion gates pass.**

## Workload Metrics

### Execution

| Metric | Definition |
| --- | --- |
| Workload status | The workload's current overall status when the result is recorded. |
| Current lifecycle state | The workload's current state from the standard migration lifecycle. |
| Total agent runtime | The cumulative elapsed agent execution time across all sessions, reported in seconds. |
| Retries | The number of repeated execution attempts after an unsuccessful or incomplete attempt. |
| Number of agent sessions | The count of distinct agent sessions used for the workload. |

### Quality

| Metric | Definition |
| --- | --- |
| Build result | The final target build result and the candidate revision to which it applies. |
| Tests passed / failed | The number of passing and failing tests from the final validation run. |
| Independent review result | The final outcome of the required independent review. |
| Number of P0 / P1 / P2 findings | Separate counts of independent-review findings at each severity. |
| PR accepted / rejected | Whether the pull request was accepted or rejected. It remains pending until either outcome occurs. |

### Human Involvement

| Metric | Definition |
| --- | --- |
| Number of human approval gates | The count of required human approval decisions encountered by the workload. |
| Number of human interventions during execution | The count of unplanned human actions needed while the agent was executing. |
| Intervention reason | A categorized reason and short explanation for each human intervention. |

### Economics

| Metric | Definition |
| --- | --- |
| Estimated manual engineering effort | The estimated engineering hours the approved workload would require without autonomous execution. |
| Autonomous execution time | The elapsed time spent in autonomous execution, reported in seconds. |
| Agent cost where available | The attributable agent execution cost and currency, when cost data is available. |
| Cost per successful migration | The total attributable agent cost divided by successful migrations. For a single workload, record this only after the workload qualifies as successful. |
| Estimated engineering hours released | The estimated manual engineering effort avoided or made available for other work, less direct human effort spent on the workload. |

All estimated fields must be labeled as estimates in reports and must document
their estimation method. Missing cost data must remain unavailable rather than
being replaced with an invented value.

## Factory-Level Metrics

| Metric | Definition |
| --- | --- |
| Migration throughput | The number of successful migrations during the stated reporting period. |
| Success rate | Successful migrations divided by workloads that reached a final promotion decision during the reporting period. |
| Human intervention rate | Workloads requiring at least one execution intervention divided by workloads that entered `executing` during the reporting period. |
| Average time to PR | Mean elapsed time from `discovered` to `pr_open` for workloads that opened a pull request during the reporting period. |
| Average cost per successful migration | Total attributable agent cost for successful migrations divided by the number of successful migrations with available cost data. Reports must disclose cost-data coverage. |
| Percentage reaching `ready_for_human_merge` | Workloads reaching `ready_for_human_merge` divided by workloads that entered `executing` during the reporting period. |

Each factory-level report must state its reporting period, population,
denominators, exclusions, and cost-data coverage.

## Success Rule

A workload is successful only when remote CI passes for the exact candidate
revision and the workload receives a passing factory promotion-gate decision.
Local validation alone does not qualify a migration as successful.
