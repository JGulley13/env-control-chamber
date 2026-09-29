# EC Chamber Modeling Method

> **Disclaimer:** This document is AI-generated for each assignment to keep it aligned with the project's latest methods. Verify guidance against the current source files.

## Summary

**The system.** An enclosure for a hobbyist 3D printer. It holds temperature, humidity, and air quality within limits, and filters exhaust air.

**The method.** Sierra (`www.modelware.io`) and openCAESAR (`opencaesar.io`) are used unchanged. They supply the general vocabulary, units, and most editor templates. This project adds EC Chamber vocabulary, four authoring patterns with editors and business rules, and reusable analyses. The model itself lives under `src/model/`; the method lives under `src/method/`.

**Where to start.** Open [Overview.md](src/model/md/www.example.com/EC%20Chamber/Overview.md) in the OML Markdown Preview. It links to every page, in method order.

## Deliverable 4 at a Glance

| Criterion (minimum) | Met by | Where in the model |
|---|---|---|
| Description patterns (4) | Verification case, item flow, power demand, transition guard | See **Patterns and Editors** below |
| Description layout (folders/files + rationale) | One description per engineering concern | `src/model/oml/www.example.com/ec-chamber/` folders; rationale in **Description Layout** below |
| Editors (one per pattern) | 4 editor templates | [verification.md](src/method/md/www.modelware.io/sierra/operational-analysis/verification.md), [item-flows.md](src/method/md/www.example.com/method/system-analysis/item-flows.md), [power.md](src/method/md/www.example.com/method/system-analysis/power.md), [transitions.md](src/method/md/www.example.com/method/operational-analysis/transitions.md) |
| Business rules (≥ 2) | 3 SHACL business rules and 2 OML inference rules | The `sh:sparql` blocks in verification.md, power.md, and transitions.md; the `rule` blocks in [component.oml](src/method/oml/www.example.com/component.oml) and [environment.oml](src/method/oml/www.example.com/environment.oml) |
| Notebook (narrative + live/editor content) | Guidance prose above each editor, plus a live pathway table | The four editor templates; [connections.md](src/method/md/www.example.com/method/system-analysis/connections.md) |
| Compose templates (all editors reusable) | Every editor is a compose template; project pages hold no shapes | Pages: [Verifications](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/Verifications.md), [Connections](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Connections.md), [Power](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Power.md), [StateMachine1](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/statemachines/StateMachine1.md) |
| Dogfooding (≥ 10 via editors) | 11 instances from Assignment 4 | [verifications.oml](src/model/oml/www.example.com/ec-chamber/operational-analysis/verifications.oml) (VC1–VC5), [connections.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/connections.oml) (6 connections). See **Dogfooding Evidence** below |
| METHOD.md (what and why) | This document | — |

## What This Project Adds

**Vocabulary** in `src/method/oml/www.example.com/`:

| File | Adds |
|---|---|
| [component.oml](src/method/oml/www.example.com/component.oml) | `ElectricalLoad` and `PowerSource` aspects; `powerDemand` and `powerCapacity` in watts; `activeIn` (the operating states in which a load draws power); `structuralCapacity` in kg; the `PowerDependency` rule |
| [environment.oml](src/method/oml/www.example.com/environment.oml) | Chamber, sensor, actuator, filter, and condition concepts with limits (`upperLimit`, `setpoint`, …); `gauges`, `regulates`, and `requiresSensor` (which sensors a guard depends on); the `TemperatureMaintenance` rule |
| [stakeholder.oml](src/method/oml/www.example.com/stakeholder.oml) | `CriticalRequirement`, a requirement restricted to High priority (defined, not yet used) |
| [bundle.oml](src/method/oml/www.example.com/bundle.oml) | The method bundle: project vocabularies plus the Sierra vocabularies they build on |

The power roles are aspects, not concepts, because a load such as the Heater Bank is also a `PhysicalPart` and a `Heater`. Sibling concepts are disjoint in a bundle, so a concept made the model inconsistent.

**Templates:**

- Editors and the live pathway view are in `src/method/md/www.example.com/method/`.
- The analyses are in `src/method/md/www.example.com/method/analysis/`.
- One exception: [verification.md](src/method/md/www.modelware.io/sierra/operational-analysis/verification.md) sits in the Sierra-style path, but its shape and editor were written by this project for Assignment 4.

## Description Layout

The model is split by engineering concern, not by class. A file changes for one reason and has one likely owner. Files that add facts to existing components (mass, power, environment) use `ref instance`, so each component keeps one identity. The [model bundle](src/model/oml/www.example.com/ec-chamber/bundle.oml) includes every description.

| File | Concern | Why separate |
|---|---|---|
| `src/model/oml/www.example.com/ec-chamber/context-analysis/` | Stakeholders, concerns, missions, objectives | Change when needs change |
| [requirements.oml](src/model/oml/www.example.com/ec-chamber/operational-analysis/requirements.oml) | What the system must do | Changes when a requirement is agreed |
| [verifications.oml](src/model/oml/www.example.com/ec-chamber/operational-analysis/verifications.oml) | How each requirement is shown to be met | Changes as testing progresses |
| [items.oml](src/model/oml/www.example.com/ec-chamber/operational-analysis/items.oml) | Items that flow or trigger behavior | Shared by connections and state machines |
| [statemachines/sm1.oml](src/model/oml/www.example.com/ec-chamber/operational-analysis/statemachines/sm1.oml) | Operating modes and fault behavior | Controls engineer |
| [components.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/components.oml) | Decomposition | Architect |
| [masses.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/masses.oml) | Mass | Estimates mature separately |
| [power.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/power.oml) | Power demand and capacity | Electrical engineer |
| [connections.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/connections.oml) | Ports and item flows | Changes when an interface changes |
| [environment.oml](src/model/oml/www.example.com/ec-chamber/system-analysis/environment.oml) | Conditions, limits, and the sensors and actuators serving them | Limits change independently of structure |

## Patterns and Editors

Each pattern is a compose template with a SHACL shape. The same shape drives the editor UI, validates instances, and states the pattern.

| Pattern | Focal type | Required content | Business rule | Editor template → project page |
|---|---|---|---|---|
| **Verification case** | `VerificationCase` | One requirement or more; method and status from fixed lists; evidence; pass/fail criteria | A Passed or Failed case must cite evidence | [verification.md](src/method/md/www.modelware.io/sierra/operational-analysis/verification.md) → [Verifications](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/Verifications.md) |
| **Item flow** | `Connection` | Exactly one source port, one receiving port, one item | Cardinality only | [item-flows.md](src/method/md/www.example.com/method/system-analysis/item-flows.md) → [Connections](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Connections.md) |
| **Power demand** | `ElectricalLoad`, `PowerSource` | One non-negative demand per load and the states it is active in; one positive capacity per source | A component with children must not store its own demand; totals roll up | [power.md](src/method/md/www.example.com/method/system-analysis/power.md) → [Power](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Power.md) |
| **Transition guard** | `Condition` guarding a `Transition` | A condition expression; at least one guarded transition; the sensors it requires | A sensor-dependent guard must not guard a fault-triggered transition, or the fault would block its own response | [transitions.md](src/method/md/www.example.com/method/operational-analysis/transitions.md) → [StateMachine1](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/statemachines/StateMachine1.md) |

Design choices:

- **Power demand copies Sierra's mass pattern.** The hierarchy and the derived total are read-only; only the leaf value is editable. A passive part is recorded as `0`; a blank means "not estimated".
- **Transition guards extend Sierra instead of copying it.** States and transitions are authored in Sierra's State Machine editor on the same page. The project template adds only guards and the fault rule.
- **Messages are guidance.** Each `sh:message` says what to do, not just what is wrong.

**Sierra patterns used unchanged.** The model also follows these Sierra patterns through Sierra's own editors. They are listed so a reviewer can tell what the analyses build on; they are not project contributions.

| Pattern | What to look for | Project page |
|---|---|---|
| Stakeholders, concerns, missions, objectives | Who needs what, and the objectives derived from those needs | Context Analysis pages (Stakeholders, Concerns, Missions, Objectives) |
| Capabilities, entities, items | What the system must be able to do, and who does it | Operational Analysis pages (Capabilities, Entities, Items) |
| Requirements | R1–R9, which the verification cases trace to | [Requirements](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/Requirements.md) |
| Component decomposition | The containment tree that every rollup sums over | [Components](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Components.md) |
| Mass rollup | One mass per leaf part. Composites must not store their own mass (Sierra rule); totals are derived. The chamber's `structuralCapacity` is a project property, entered as OML text because Sierra's editor has no column for it | [Masses](src/model/md/www.example.com/EC%20Chamber/System%20Analysis/Masses.md) |
| State machine | States and transitions of `sm1`; the project guard editor sits below it | [StateMachine1](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/statemachines/StateMachine1.md) |

Six pre-existing pages have no model description yet, so they render empty: Activities, Processes, Scenarios 1–3, and StateMachine2. They are not part of this method's patterns.

## Business Rules

| Rule | Kind | Defined in |
|---|---|---|
| Passed or Failed verification needs evidence | SHACL | [verification.md](src/method/md/www.modelware.io/sierra/operational-analysis/verification.md) |
| No stored demand on a composite; demand ≥ 0; capacity > 0 | SHACL | [power.md](src/method/md/www.example.com/method/system-analysis/power.md) |
| No sensor-dependent guard on a fault transition | SHACL | [transitions.md](src/method/md/www.example.com/method/operational-analysis/transitions.md) |
| A chamber whose sensor gauges a temperature maintains that temperature | OML inference (fires) | [environment.oml](src/method/oml/www.example.com/environment.oml) |
| The receiver of an energy connection depends on its provider | OML inference (**infers nothing**; see Known Limits) | [component.oml](src/method/oml/www.example.com/component.oml) |

Each project SHACL rule was tested by breaking it on purpose: a composite given its own demand, a negative demand, and a sensor guard added to a fault transition. `oml validate` reported each violation with its message, and passed again after the change was reverted.

## Analysis Layer

This belongs to Deliverable 5. Each analysis is a compose template in `src/method/md/www.example.com/method/analysis/` that answers one question with plain SPARQL. The only script is the power margin. Project pages supply only the inputs.

| Template | Question | Inputs | Used on |
|---|---|---|---|
| [power-budget.md](src/method/md/www.example.com/method/analysis/power-budget.md) | Power demand per operating state; is the supply sufficient? | none | Dashboard |
| [mass-rollup.md](src/method/md/www.example.com/method/analysis/mass-rollup.md) | Total mass; is it within structural capacity? | none | Dashboard |
| [threshold-impact.md](src/method/md/www.example.com/method/analysis/threshold-impact.md) | Which verification cases must be re-run if a limit rises? | `condition`, `increase` | Dashboard |
| [sensor-failure.md](src/method/md/www.example.com/method/analysis/sensor-failure.md) | Which transitions does a sensor failure block? | `sensor` | Dashboard, Sensor Failures, StateMachine1 |
| [verification-coverage.md](src/method/md/www.example.com/method/analysis/verification-coverage.md) | Is every requirement on its way to being verified? | none | Dashboard |

Pages: [Analysis Dashboard](src/model/md/www.example.com/EC%20Chamber/Analysis/Dashboard.md), [Sensor Failures](src/model/md/www.example.com/EC%20Chamber/Analysis/Sensor%20Failures.md), [StateMachine1](src/model/md/www.example.com/EC%20Chamber/Operational%20Analysis/statemachines/StateMachine1.md). Findings are in [ANALYSIS.md](ANALYSIS.md).

**Scope comes from the calling page's `ontology:`.** A template pin has no effect under compose. A scope that is too narrow does not error; it silently returns fewer rows. For that reason the analyses are compose templates on bundle-scoped pages, not navigation templates.

## Dogfooding Evidence

These instances were created through the project's editors in Assignment 4:

| Editor | Instances | Count |
|---|---|---:|
| Verification | `VC1`–`VC5` | 5 |
| Item Flows | `Conn_TempSensor_to_Controller`, `Conn_HumiditySensor_to_Controller`, `Conn_AirQuality_to_Controller`, `Conn_PowerSupply_to_Heater`, `Conn_PowerSupply_to_AHU`, `Conn_AirFilter_outflow` | 6 |
| **Total** | | **11** |

The instances added since were **written as OML text, not entered through an editor**:

- 12 loads and the power source in `power.oml`;
- the guards in `sm1.oml`;
- `VC6`.

They pass the editors' validation. They count as dogfooding only once they are edited through the Power and StateMachine1 pages.

## Validation and Known Limits

From the repository root:

- `oml lint`
- `oml validate`
- `oml reason`
- `oml render -m src/model/md -b <out> -c src/model/oml/www.example.com/ec-chamber/bundle.oml`

All four currently pass.

- **Illustrative values.** The power demands and capacity, which states each load is active in, the condition limits, and the `sm1` state machine were authored for this deliverable as plausible placeholders, not from datasheets or a controller design. So is the chamber's 90 kg `structuralCapacity` in `masses.oml`.
- **`PowerDependency` infers nothing,** although every antecedent is present. `TemperatureMaintenance` fires, so the fault is in the rule's `Connection(p1, c, p2)` predicate. No analysis relies on the rule.
- **Sierra's System Dashboard mass rollup shows blank totals.** It casts the quantity node instead of its value. It is left unchanged upstream; the project's mass-rollup template does the arithmetic correctly.
- **Sierra's `TransitionShape` rejects `Initial` and `Final` states.** Its validation doesn't apply the subclass axioms. `sm1` also types `Start` and `Off` as `State` as a workaround.
- **Static render does not run scripts or exercise editors.** Check the following in the Preview:
  - editing a demand on the Power page updates `power.oml` and the rolled-up total;
  - breaking the guard rule on StateMachine1 shows its message;
  - the power-margin script renders on the Dashboard.

## Reuse Guidance

1. Use the project method bundle; extend Sierra in your own namespace rather than copying it. Use aspects for roles that combine with other types.
2. Split descriptions by concern and add facts with `ref instance`.
3. Compose editor templates on the page for the description they write to. Compose analysis templates on a bundle-scoped page, passing your own `condition` and `sensor` IRIs.
4. Author facts; derive totals, margins, and coverage in views.
5. Record an unanswerable question as a diagnosed gap: vocabulary, pattern, process, or outside the model.
6. Run all four checks before publishing, and state which instances were entered through editors.
