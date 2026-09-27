# Env Control Chamber

The Env Control Chamber ontologies.

This is an [OML](https://www.modelware.io/) project.

The Environmental Control (EC) Chamber is designed for hobbyist 3D printers. It controls temperature, humidity, and air quality in a small enclosure, and filters exhaust air to reduce particulates and contaminants.

## Repository Navigation

OML source is organized into two layers under `src/`:

- `src/method/` defines reusable vocabularies and their documentation.
- `src/model/` describes the EC Chamber project using those vocabularies.

### Method: Vocabularies

The Sierra method vocabularies are in [`src/method/oml/www.modelware.io/sierra/`](src/method/oml/www.modelware.io/sierra/). Each `.oml` file defines a vocabulary, for example [`base.oml`](src/method/oml/www.modelware.io/sierra/base.oml), [`component.oml`](src/method/oml/www.modelware.io/sierra/component.oml), and [`stakeholder.oml`](src/method/oml/www.modelware.io/sierra/stakeholder.oml). Sierra's [`bundle.oml`](src/method/oml/www.modelware.io/sierra/bundle.oml) groups its standard vocabularies.

Project-specific vocabulary extensions and additions are in [`src/method/oml/www.example.com/`](src/method/oml/www.example.com/):

- [`component.oml`](src/method/oml/www.example.com/component.oml) extends Sierra's component vocabulary with the project power-dependency relation and rule.
- [`stakeholder.oml`](src/method/oml/www.example.com/stakeholder.oml) extends Sierra's stakeholder vocabulary with `CriticalRequirement`.
- [`environment.oml`](src/method/oml/www.example.com/environment.oml) defines EC Chamber concepts such as chambers, sensors, actuators, and environmental conditions.
- [`bundle.oml`](src/method/oml/www.example.com/bundle.oml) is the project method bundle. It includes these project vocabularies and the Sierra vocabularies they build on.

Add reusable project concepts and relations to these method vocabularies. Sierra-defined terms retain their Sierra IRIs; terms introduced by the project extensions use the IRIs declared in the `www.example.com` vocabulary files.

### Model: EC Chamber Descriptions

The project descriptions are in [`src/model/oml/www.example.com/ec-chamber/`](src/model/oml/www.example.com/ec-chamber/). The root [`bundle.oml`](src/model/oml/www.example.com/ec-chamber/bundle.oml) uses the project method bundle and includes the project description files.

- `context-analysis/` contains `stakeholders.oml`, `concerns.oml`, `missions.oml`, and `objectives.oml`.
- `operational-analysis/` contains `capabilities.oml`, `entities.oml`, `items.oml`, and `requirements.oml`, plus the `scenarios/` and `statemachines/` folders.
- `system-analysis/` contains `components.oml`, `connections.oml`, and `masses.oml`.

Add project instances and assertions to these model descriptions. Use Sierra IRIs when describing Sierra concepts, and use the project vocabulary IRIs when assigning project-defined types or asserting project-defined relations. Including a vocabulary in a bundle makes it part of the model's available ontology set; it does not automatically assign its custom types to model instances.

### Markdown Views

Markdown templates and documentation for the Sierra method are in [`src/method/md/www.modelware.io/sierra/`](src/method/md/www.modelware.io/sierra/). EC Chamber project pages are in [`src/model/md/www.example.com/EC Chamber/`](src/model/md/www.example.com/EC%20Chamber/), organized into Context Analysis, Operational Analysis, and System Analysis. These Markdown files provide browsable views and templates; the OML files above define the vocabularies and project descriptions.

## Getting Started

1. Open this folder in VS Code with the **OML Code** extension installed.
2. Run `oml lint` to check the model for errors.
3. Start with the project method bundle and model bundle, then edit the relevant `.oml` file under `src/method/oml/` or `src/model/oml/`.

## Commands

- `oml lint` — lint OML files and report any problems
- `oml start` — start the OML server
- `oml export -o build/owl` — export OWL files
