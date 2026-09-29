---
template:
  id: http://www.example.com/method/system-analysis/power
  name: "Power Demand"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Power Demand

Mark each powered leaf component as an electrical load and record its nominal operating demand in watts. Leave composites alone: their totals are derived from their descendants, so a stored total would be a second source of truth that goes stale.

Record a passive part (a filter, a shell) as an explicit `0` when it is known to draw nothing. Leaving it blank means "not yet estimated".

For each load, list the operating states it runs in (**Active In**). The Power Budget analysis sums each state's loads to find the peak demand.

```tree-editor
---
columns: { this: { label: "Component" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix component: <https://www.modelware.io/sierra/component#> .
@prefix eccomponent: <http://www.example.com/component#> .
@prefix state: <https://www.modelware.io/sierra/state#> .

# Read-only context: the decomposition and the derived total.
eccomponent:PowerHierarchyShape
    a sh:NodeShape ;
    sh:targetClass component:Component ;
    dash:readOnly true ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX base: <https://www.modelware.io/sierra/base#>
            PREFIX component: <https://www.modelware.io/sierra/component#>
            PREFIX eccomponent: <http://www.example.com/component#>
            CONSTRUCT {
                $this eccomponent:totalPower ?formatted
            } WHERE {
                SELECT $this (CONCAT(STR(SUM(?w)), " W") AS ?formatted)
                WHERE {
                    $this a component:Component ;
                        base:contains* ?load .
                    ?load eccomponent:powerDemand ?w .
                }
                GROUP BY $this
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "A component that contains other components must not assert its own power demand. Record demand on its leaf loads; the total rolls up." ;
        sh:select """
            PREFIX base: <https://www.modelware.io/sierra/base#>
            PREFIX eccomponent: <http://www.example.com/component#>
            SELECT $this WHERE {
                ?child base:isContainedBy $this .
                $this eccomponent:powerDemand ?w .
            }
        """ ;
    ] ;
    sh:property [
        sh:path base:isContainedBy ;
        sh:name "Container" ;
        sh:class component:Component ;
        dash:composite true ;
    ] ;
    sh:property [
        sh:path eccomponent:totalPower ;
        sh:name "Total Demand" ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    .

# Editable work: one demand value per load.
eccomponent:ElectricalLoadShape
    a sh:NodeShape ;
    sh:targetClass eccomponent:ElectricalLoad ;
    sh:property [
        sh:path eccomponent:powerDemand ;
        sh:name "Demand (W)" ;
        sh:datatype xsd:decimal ;
        sh:minInclusive 0 ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "Record one non-negative nominal demand in watts. Use 0 for a load known to draw nothing." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path eccomponent:activeIn ;
        sh:name "Active In" ;
        sh:class state:State ;
        sh:order 4 ;
    ] ;
    .

# Editable work: the capacity the loads are budgeted against.
eccomponent:PowerSourceShape
    a sh:NodeShape ;
    sh:targetClass eccomponent:PowerSource ;
    sh:property [
        sh:path eccomponent:powerCapacity ;
        sh:name "Capacity (W)" ;
        sh:datatype xsd:decimal ;
        sh:minExclusive 0 ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "Record one positive continuous capacity in watts." ;
        sh:order 2 ;
    ] ;
    .
```
