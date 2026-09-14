# Env Control Chamber

The Env Control Chamber ontologies.

This is an [OML](https://www.modelware.io/) project.

The Environmental Control (EC) chamber is a chamber designed with hobbyist 3D printers in mind. For optimal 3D printing, environemnt variables such as temperture, humidity, and air quality need to be controlled. In most situations, this is extremely difficult without a small and controlled space in which the 3D printer operates. Therefore, the EC chamber will house most commercially available 3D printers within it and maintain appropriate temperature and humidity levels. For user safety, a HEPA filter cleans the exhaust air, ensuring microplastics and harmful gases are mostly cleansed before exiting the chamber.

## Structure
This repo borrows heavily from the Sierra method of model organization. The vocabulary and definition are split into two folders, `oml\www.example.com\method`and `oml\www.example.com\model`, respectively. Within `method`, the Sierra vocabularies have been slightly modified to accomodate the EC chamber and a new file was added: `environment.oml`. Similarly, within `model`, the Sierra organizational method was used to delineate spaces for capabilities, stakeholders, requirements, components, mass, and more. No custom files were needed for the EC chamber in the model folder.

## Getting Started

1. Open this folder in VS Code with the **OML Code** extension installed.
2. Run `oml lint` to check the model for errors.
3. Edit the OML files under `src/oml/` to build out your model.

## Commands

- `oml lint` — lint OML files and report any problems
- `oml start` — start the OML server
- `oml export -o build/owl` — export OWL files
