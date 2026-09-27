# EC Chamber Modeling Method

> **Disclaimer:** This document is AI-generated for each assignment to keep it aligned with the project's latest methods. Verify guidance against the current source files.

## Purpose

This method organizes the Environmental Control (EC) Chamber model and provides reusable editors and live views for developing it. It builds on the Sierra method rather than replacing it: Sierra supplies general systems-engineering concepts and patterns; this project adds EC Chamber-specific concepts, rules, and a connection-flow workflow.

The guidance below describes the current repository. Where editor usage cannot be verified from the model files, that limitation is stated rather than inferred.

## Minimum OML Background

OML separates a methodology from a model:

- **Vocabularies** define concepts, relations, properties, and rules. They form the method's domain language.
- **Descriptions** use vocabulary terms to define a particular system and its assertions and instances.
- **Bundles** collect vocabularies or descriptions into an importable and reason-able set.
- **Markdown templates** provide reusable Compose content. Notebook blocks can query the loaded model and display live results; editor blocks describe structured authoring surfaces.

Sierra is the upstream method at `https://www.modelware.io/sierra/...`. Its terms keep their Sierra IRIs when used by this project. This repository's project vocabularies have their own IRIs under `http://www.example.com/...`; they extend Sierra where appropriate. Including a custom vocabulary in a bundle makes it available, but does not automatically type any model instance with its custom concepts.

## What This Project Adds

The project method source lives under `src/method/oml/www.example.com/`:

- `component.oml` extends Sierra's component vocabulary and adds `dependsOnPower` plus the `PowerDependency` rule.
- `stakeholder.oml` extends Sierra's stakeholder vocabulary and adds `CriticalRequirement`, a requirement restricted to High priority.
- `environment.oml` introduces chamber, environmental-condition, sensor, actuator, and air-filter concepts; their properties and relations; and `TemperatureMaintenance`.
- `bundle.oml` includes the project vocabularies and the Sierra vocabularies they depend on.

Sierra's upstream general-purpose vocabularies are under `src/method/oml/www.modelware.io/sierra/`; most Sierra Markdown templates are under `src/method/md/www.modelware.io/sierra/`. One important provenance exception is the local Verification editor template at `src/method/md/www.modelware.io/sierra/operational-analysis/verification.md`: its `VerificationCaseShape` and editor were created by the project author for Assignment 4 and were not originally part of Sierra. The file is stored in the Sierra-style path and uses Sierra's verification vocabulary, but its editor/shape is a local method contribution. The project should add to upstream vocabularies rather than copy them. Add a custom vocabulary when the project needs new domain meaning; keep project-specific facts in model descriptions.

The active model bundle, `src/model/oml/www.example.com/ec-chamber/bundle.oml`, uses the project method bundle and includes the EC Chamber descriptions.

## Description Patterns and Layout

The model is organized by engineering concern so each description has a clear subject and can be composed with related descriptions:

| Pattern | Description location | Reusable editor | Purpose |
|---|---|---|---|
| Stakeholders and context | `src/model/oml/www.example.com/ec-chamber/context-analysis/` | Sierra stakeholder, concern, mission, and objective templates | Capture who has an interest in the system, why it matters, and the objectives that guide it. |
| System decomposition | `src/model/oml/www.example.com/ec-chamber/system-analysis/components.oml` | Sierra Components tree editor | Define system parts and their containment hierarchy. |
| Item flows | `src/model/oml/www.example.com/ec-chamber/system-analysis/connections.oml` | Project Item Flows table editor | Define each connection as one item flow between a source port and a receiving port. |
| Requirements and verification | `src/model/oml/www.example.com/ec-chamber/operational-analysis/requirements.oml` and `verifications.oml` | Sierra Requirements editor; locally authored Assignment 4 Verification editor | State what the system must do, then connect verification cases to requirements and evidence. |

Other model descriptions are grouped similarly: `operational-analysis/` contains capabilities, entities, items, requirements, scenarios, state machines, and verifications; `system-analysis/` contains components, connections, and masses. The model bundle collects these descriptions.

The structure separates reusable semantics from project assertions. For example, the component vocabulary defines what a `Component`, `Port`, and `Connection` mean; `components.oml` and `connections.oml` describe the parts and flows of this chamber.

## Reusable Editors and Notebooks

Compose templates live under `src/method/md/`. Sierra templates provide reusable editors for the inherited patterns, with the local Verification editor described above as an exception. The project-specific templates are under `src/method/md/www.example.com/method/system-analysis/`:

- `item-flows.md` is an editable table over Sierra `Connection` relation instances. Each row has exactly one source port, one receiving port, and one transferred item. The row represents one connection, not a multi-hop route.
- `connections.md` is a live Python notebook view. It queries connections, resolves each endpoint port's owning component, and displays source part, source port, connection, item, receiving port, and receiving part. It shows all item kinds represented by the model, including signals, energy, and material. An endpoint without an owning component remains visible as unassigned rather than being dropped.

The EC Chamber Markdown page `src/model/md/www.example.com/EC Chamber/System Analysis/Connections.md` composes the project flow editor, Sierra's connection diagram, and the live pathway notebook. This is the example of the intended authoring-and-review loop: author one flow in the editor, then inspect the model-driven table and diagram.

The project's Markdown pages are under `src/model/md/www.example.com/EC Chamber/`; corresponding OML descriptions are under `src/model/oml/www.example.com/ec-chamber/`. Markdown provides navigable Compose experiences and notebook views; OML remains the source of model facts.

### Verification and Connection Editor Patterns

The two editor-backed patterns most directly added to or demonstrated by this project are:

| Editor | Modeling pattern | Shape and authored facts | Reuse source |
|---|---|---|---|
| Verification table | A verification case is an entity that verifies one requirement and records its method, status, optional description and pass/fail criteria, and evidence. A Passed or Failed case must have evidence. | The locally authored Assignment 4 `VerificationCaseShape` targets Sierra's `verification:VerificationCase`; its property columns are derived from `verifies`, `method`, `status`, `hasEvidence`, and inherited base properties. Its SPARQL constraint flags Passed or Failed cases without evidence. | The project reuses its Assignment 4-authored `verification.md` Compose template, stored under the Sierra-style path; the verification vocabulary itself is Sierra's. |
| Item-flow table | A connection is one directed flow of one item between two ports. The parts at either end are derived from port ownership; they are not separately asserted in each row. | The project `ConnectionShape` targets Sierra `component:Connection` relation instances. It requires exactly one `oml:hasSource` port, one `oml:hasTarget` port, and one `component:transfers` item. | The project-specific `item-flows.md` Compose template defines the editor; `connections.md` supplies the live pathway view. |

These patterns preserve the distinction between authored facts and computed presentation. The verification editor authors verification cases; the connection editor authors endpoint and flow assertions. The live pathway notebook joins those connection assertions to port ownership to display source part, source port, connection, item, receiving port, and receiving part.

## Business Rules

The project method currently defines two inference rules:

1. **Power dependency** (`PowerDependency`, in `component.oml`): when a connection carries an item typed as Sierra `Energy`, infer that the component owning the receiving port depends on the component owning the source port. The assertion direction follows the physical flow: receiver depends on provider. The current model represents electrical power from `PowerSupply` to powered equipment, not from the controller.
2. **Temperature maintenance** (`TemperatureMaintenance`, in `environment.oml`): when a chamber has a sensor that gauges a temperature condition, infer that the chamber maintains that condition.

These are entailment rules: they derive relationships from asserted facts. They do not replace the editor's responsibility to capture the ports, connection, transferred item, sensor, gauge, and chamber relationships that support the inference.

## Current Dogfooding Evidence

The following 11 model instances are the editor dogfooding set, created through the corresponding reusable Compose tables:

| Editor | Instances | Count |
|---|---|---:|
| Assignment 4 Verification table (local template stored under Sierra path) | `VC1`, `VC2`, `VC3`, `VC4`, `VC5` | 5 |
| Project Item Flows table | `Conn_TempSensor_to_Controller`, `Conn_HumiditySensor_to_Controller`, `Conn_AirQuality_to_Controller`, `Conn_PowerSupply_to_Heater`, `Conn_PowerSupply_to_AHU`, `Conn_AirFilter_outflow` | 6 |
| **Total** |  | **11** |

This meets the criterion of at least ten total instances created through reusable editors. The verification statuses are intentionally varied to demonstrate status handling: VC1 is In Progress, VC2 and VC5 are Passed, and VC3 and VC4 are Not Started. Evidence is present on both Passed cases as required; other cases may include evidence before completion, but any case set to Passed or Failed must have evidence. No inferred relationship is counted as an authored instance.

## Validation and Known Limits

Use these commands from the repository root:

- `oml lint` checks OML source syntax and workspace consistency.
- `oml validate` runs SHACL validation for the Markdown editor shapes against the current model.
- `oml render -m src/model/md -b <output-folder> -c src/model/oml/www.example.com/ec-chamber/bundle.oml` checks that project Markdown and Compose templates render.
- `oml reason` checks ontology consistency and inference roots.

At the time this guide was written, Markdown rendering succeeds. `oml validate` reports two mass-rollup violations: `EnvironmentalChamber` and `AirHandlingUnit` have asserted masses despite containing children. Those are existing model-data issues, not failures reported against the Item Flows editor. Keep this distinction when reporting validation status.

Static rendering confirms that the Markdown and Compose templates resolve, but it does not exercise the interactive editor or verify that a UI edit reaches the live query. To check that workflow, open the EC Chamber System Analysis Connections page in the OML Markdown Preview, change a source port, receiving port, or flow item in the Item Flows table, confirm the assertion is saved, and verify that the Live Pathways row reflects the change. Restore the intended value afterward if this is only a test. This is the remaining manual acceptance check; it is different from creating the 11 dogfooding instances already listed above.

The project environmental concepts and `CriticalRequirement` are available from the method bundle but are not yet used as types by the current EC Chamber model instances. The product components and requirements currently use Sierra's general types. Using the specialized project concepts in model descriptions is a separate modeling step, not automatic bundle behavior.

## Reuse Guidance

To reuse the method for another project:

1. Include the Sierra vocabularies required by the model and the project extensions that apply.
2. Keep project extensions in their own namespace and extend Sierra concepts instead of copying their definitions.
3. Create project descriptions under a project-specific model namespace, arranged by the same concerns where useful.
4. Compose reusable editor templates against the appropriate description or bundle context.
5. Use live notebook queries for derived summaries and pathways; edit the underlying asserted facts through an editor.
6. Add domain rules only when the antecedent facts and inferred conclusion have a clear, testable meaning.
7. Validate the OML, editor shapes, and rendered Compose pages, and record which model examples were actually entered through editors when demonstrating dogfooding.
