---
template:
  id: http://www.example.com/method/analysis/sensor-failure
  name: "Sensor Failure"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: sensor
      type: iri
      required: true
---
## Sensor Failure

**Question.** If [[${sensor}]] fails, which transitions are blocked, and where can the chamber still go?

**Blocked:** transitions guarded by a condition that requires this sensor.

```table
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX state: <https://www.modelware.io/sierra/state#>
PREFIX env: <http://www.example.com/environment#>

SELECT ?Transition ?Source ?Target ?Trigger ?Guard
WHERE {
    ?Guard env:requiresSensor <${sensor}> ;
        state:guards ?Transition .
    ?Transition oml:hasSource ?Source ;
        oml:hasTarget ?Target .
    OPTIONAL { ?Transition state:isTriggeredBy ?Trigger }
}
ORDER BY ?Source
```

**Fault responses:** transitions triggered by a fault. The Transition Guards rule forbids sensor guards on these, so a sensor failure cannot block them.

```table
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX state: <https://www.modelware.io/sierra/state#>

SELECT ?Transition ?Source ?Target ?Trigger
WHERE {
    ?Transition state:isTriggeredBy ?Trigger ;
        oml:hasSource ?Source ;
        oml:hasTarget ?Target .
    ?Trigger base:category "Fault" .
}
ORDER BY ?Source
```

The blocked transitions as a graph: the sensor points to each transition it blocks, and each transition points to the states it would connect.

```graph
---
layout:
  mode: dag
  fit: true
  dag:
    rankDir: LR
---
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX state: <https://www.modelware.io/sierra/state#>
PREFIX env: <http://www.example.com/environment#>
PREFIX analysis: <http://www.example.com/method/analysis#>

CONSTRUCT {
    <${sensor}> analysis:blocks ?transition .
    ?transition analysis:from ?source ;
        analysis:to ?target .
}
WHERE {
    ?guard env:requiresSensor <${sensor}> ;
        state:guards ?transition .
    ?transition oml:hasSource ?source ;
        oml:hasTarget ?target .
}
```

**Interpretation.** Only modeled guards appear. A guard written as text without its required sensors is invisible here. Every state that heats or dehumidifies should have a fault response.
