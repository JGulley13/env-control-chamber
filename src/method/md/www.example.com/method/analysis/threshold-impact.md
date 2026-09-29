---
template:
  id: http://www.example.com/method/analysis/threshold-impact
  name: "Threshold Impact"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: condition
      type: iri
      required: true
    - id: increase
      type: number
      required: true
---
## Threshold Impact

**Question.** If the upper limit of [[${condition}]] is raised by ${increase}, which verification activities must be re-run?

A verification case is affected when its evidence is a sensor that gauges this condition. It must be re-run when it has already produced results against the old limit: Passed, Failed, or In Progress.

```table
---
stylesheet:
  - selector: cell[col === "MustRerun" && value === "Yes"]
    target: value
    style:
      color: "#DC2626"
      font-weight: 600
---
PREFIX env: <http://www.example.com/environment#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?Case ?Status ?MustRerun ?Requirement ?Sensor ?CurrentLimit ?NewLimit ?CriticalThreshold
WHERE {
    VALUES ?condition { <${condition}> }
    ?Sensor env:gauges ?condition .
    ?Case verification:hasEvidence ?Sensor ;
        verification:verifies ?Requirement ;
        verification:status ?Status .
    OPTIONAL { ?condition env:upperLimit ?CurrentLimit }
    OPTIONAL { ?condition env:criticalThreshold ?CriticalThreshold }
    BIND(?CurrentLimit + ${increase} AS ?NewLimit)
    BIND(IF(?Status IN ("Passed", "Failed", "In Progress"), "Yes", "No") AS ?MustRerun)
}
ORDER BY ?Case
```

**Interpretation.** Cases marked No have not run yet; they only need the new limit in their pass/fail criteria. Also compare the new limit with the critical threshold: a raised limit at or above it leaves no warning band. A case that depends on the limit but cites no sensor as evidence does not appear here.
