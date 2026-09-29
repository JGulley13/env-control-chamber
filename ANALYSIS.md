# EC Chamber Analysis

> **Disclaimer:** This document is AI-generated for each assignment to keep it aligned with the project's latest methods. Verify guidance against the current source files.

Deliverable 5: each methodology question ends in an answer or a diagnosed gap. The figures below are a snapshot; the live pages recompute on every render and are authoritative.

## Where Each Criterion Is Met

| Criterion | Minimum | Where to look |
|---|---|---|
| Methodology questions | One query or diagnosed gap per Module 1 question | The four Module 1 questions below, plus one supporting question, each with its query, evidence, and finding |
| Gap detection | One orphan query per relevant pattern | A "Gap check" table in each pattern's template (see **Gap Detection** below) |
| Dashboard | Narrative + 3 view types | [Analysis Dashboard](src/model/md/www.example.com/EC%20Chamber/Analysis/Dashboard.md): chart, tree, table, matrix, and graph, each with a question and an interpretation |
| Computed analysis | One script-based analysis | Power sufficiency: a 15-line Python block in [power-budget.md](src/method/md/www.example.com/method/analysis/power-budget.md) |
| View template | One compose, navigation, or call template | Five compose templates in `src/method/md/www.example.com/method/analysis/`. `sensor-failure` is reused on the Dashboard, [Sensor Failures](src/model/md/www.example.com/EC%20Chamber/Analysis/Sensor%20Failures.md) (three sensors), and [StateMachine1](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/statemachines/StateMachine1.md) |
| ANALYSIS.md | Question → evidence → finding | This document |

## Module 1 Questions

### Electrical Engineer: total power across the state modes, and is the supply sufficient?

- **Query:** [power-budget.md](src/method/md/www.example.com/method/analysis/power-budget.md). A chart shows demand per state; the Python block finds the peak state and the margin.
- **Evidence:**

  | State | Idle | Preheating | Regulating | Alarm | SafeShutdown |
  |---|---|---|---|---|---|
  | Demand (W) | 12.5 | 324.5 | 381.5 | 381.5 | 24.5 |

  The peak is 381.5 W against a 500 W supply, a margin of 118.5 W.
- **Finding:** answered. The supply is sufficient in every state.
- **Gap (model data):** six loads (12.5 W) have no modeled power feed. The feed gap check in the same template lists them.

### Mechanical Engineer: total mass, and does it exceed structural capacity?

- **Query:** [mass-rollup.md](src/method/md/www.example.com/method/analysis/mass-rollup.md): a tree rollup plus a capacity check table.
- **Evidence:** the system totals 77.6 kg against a structural capacity of 90 kg.
- **Finding:** answered. The mass is within capacity with a 12.4 kg margin (86% of capacity used).

### Systems Engineer: which verification activities must be re-run if the temperature threshold rises by 15 °C?

- **Query:** [threshold-impact.md](src/method/md/www.example.com/method/analysis/threshold-impact.md), with inputs `condition` = ChamberTemperature and `increase` = 15.
- **Evidence:**

  | Case | Status | Must re-run | Limit (current → new) |
  |---|---|---|---|
  | VC1 | In Progress | **Yes** | 60 → 75 °C |
  | VC6 | Not Started | No | 60 → 75 °C |

- **Finding:** answered. VC1 must be re-run. The new limit of 75 °C also equals the critical threshold, which leaves no warning band.
- **Gap (vocabulary):** VC4, alarm response, depends on the limit but cites no sensor as evidence, so it is not found. Requirements should reference the condition they constrain.

### Controls Engineer: which transitions are blocked if an environmental sensor fails?

- **Query:** [sensor-failure.md](src/method/md/www.example.com/method/analysis/sensor-failure.md), run for each sensor on [Sensor Failures](src/model/md/www.example.com/EC%20Chamber/Analysis/Sensor%20Failures.md).
- **Evidence:**

  | Failed sensor | Blocked transitions |
  |---|---|
  | Temperature | Idle→Preheating, Preheating→Regulating, Alarm→Regulating, SafeShutdown→Idle |
  | Humidity | Alarm→Regulating, SafeShutdown→Idle |
  | Air quality | Alarm→Regulating, SafeShutdown→Idle |

  For every sensor, Preheating, Regulating, and Alarm keep a fault path to SafeShutdown.
- **Finding:** answered. A failed sensor can never block the move to SafeShutdown, which is consistent with R7.

## Supporting Question

Not a Module 1 question. It is on the Dashboard because the Systems Engineer's re-run answer is only as complete as verification coverage.

### Is every requirement on its way to being verified?

- **Query:** [verification-coverage.md](src/method/md/www.example.com/method/analysis/verification-coverage.md), with three checks:
  - orphans: requirements with no case;
  - near miss: requirements with cases, none passed;
  - coverage: a requirement × status matrix.
- **Evidence:**
  - R6, R8 (High), and R9 have no case.
  - R1, R3, R4, and R7 have cases, but none has passed.
  - Only R2 and R5 have a passing case.
- **Finding:** answered. R8 is the priority gap: a High-priority requirement with no verification case.

## Gap Detection

One orphan (missing-relationship) query per pattern, each labeled "Gap check" in its template:

| Pattern | Gap check | Template | Current result |
|---|---|---|---|
| Verification case | Requirements with no verification case | [verification-coverage.md](src/method/md/www.example.com/method/analysis/verification-coverage.md) | R6, R8 (High), R9 |
| Item flow | Loads with demand but no power feed | [power-budget.md](src/method/md/www.example.com/method/analysis/power-budget.md) | 6 loads |
| Power demand | Loads with demand but no operating state | [power-budget.md](src/method/md/www.example.com/method/analysis/power-budget.md) | None |
| Transition guard | Guards that name no sensor | [transitions.md](src/method/md/www.example.com/method/operational-analysis/transitions.md) | None |

An empty result is a clean result: the check ran and found nothing recorded as missing.

## Diagnosed Gaps

| Gap | Source | Action |
|---|---|---|
| Six loads with no power feed | Model data | Add energy flows in the Item Flows editor |
| Requirements do not reference conditions (VC4 missed) | Vocabulary | Add a Requirement → condition relation |
| R6, R8, R9 have no verification case | Process | Add verification cases |
| `PowerDependency` rule infers nothing | Pattern | Fix the rule's `Connection` predicate |
| Power values, limits, state assignments, and structural capacity are placeholders | Outside the model | Replace them with datasheet values and the real control design |
