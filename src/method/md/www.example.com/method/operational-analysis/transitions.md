---
template:
  id: http://www.example.com/method/operational-analysis/transitions
  name: "Transition Guards"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Transition Guards

Author states and transitions with the Sierra State Machine editor above. Use this editor to add the guard conditions that must hold for a transition to fire.

Every guard names the sensors whose readings it depends on. That link is what lets the Sensor Failure analysis answer "which transitions stop working when this sensor fails?" A guard written only as text cannot be traced.

```table-editor
---
columns: { this: { label: "Guard" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix state: <https://www.modelware.io/sierra/state#> .
@prefix env: <http://www.example.com/environment#> .

env:TransitionGuardShape
    a sh:NodeShape ;
    sh:targetClass base:Condition ;
    sh:sparql [
        sh:message "This guard depends on sensor data but guards a fault-triggered transition. The failure the transition responds to would also block it. Remove the guard from the fault transition." ;
        sh:select """
            PREFIX base: <https://www.modelware.io/sierra/base#>
            PREFIX state: <https://www.modelware.io/sierra/state#>
            PREFIX env: <http://www.example.com/environment#>
            SELECT $this WHERE {
                $this state:guards ?transition ;
                    env:requiresSensor ?sensor .
                ?transition state:isTriggeredBy ?trigger .
                ?trigger base:category "Fault" .
            }
        """ ;
    ] ;
    sh:property [
        sh:path base:expression ;
        sh:name "Condition" ;
        dash:editor dash:TextAreaEditor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "State the condition in one sentence, for example: the temperature sensor reports a valid reading." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path state:guards ;
        sh:name "Guards Transitions" ;
        sh:class state:Transition ;
        sh:minCount 1 ;
        sh:message "A guard must guard at least one transition." ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path env:requiresSensor ;
        sh:name "Requires Sensors" ;
        sh:class env:Sensor ;
        sh:order 3 ;
    ] ;
    .
```

**Gap check:** guards that name no sensor. The Sensor Failure analysis cannot see them. An empty table means every guard is traceable.

```table
PREFIX state: <https://www.modelware.io/sierra/state#>
PREFIX env: <http://www.example.com/environment#>

SELECT ?Guard ?Transition
WHERE {
    ?Guard state:guards ?Transition .
    FILTER NOT EXISTS { ?Guard env:requiresSensor ?sensor }
}
```
