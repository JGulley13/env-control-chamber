---
template:
  id: http://www.example.com/method/analysis/power-budget
  name: "Power Budget"
  rank: 0
  expose:
    - kind: compose
---
## Power Budget

**Question.** What is the total power demand in each operating state, and is the power supply sufficient for all of them?

A load counts toward a state's total when it is recorded as `activeIn` that state in the Power Demand editor.

```chart
---
type: bar
data:
  labels: state
  datasets:
    - label: Demand (W)
      data: watts
options:
  plugins:
    title:
      display: true
      text: Power demand by operating state
    legend:
      display: false
  scales:
    y:
      beginAtZero: true
---
PREFIX eccomponent: <http://www.example.com/component#>

SELECT ?state (SUM(?demand) AS ?watts)
WHERE {
    ?load eccomponent:activeIn ?stateIri ;
        eccomponent:powerDemand ?demand .
    BIND(REPLACE(STR(?stateIri), "^.*#", "") AS ?state)
}
GROUP BY ?state
ORDER BY DESC(?watts)
```

The script compares each state's total with the supply capacity and reports the worst case.

```python
# Get each state's total demand and the supply capacity.
result = await query("""
PREFIX eccomponent: <http://www.example.com/component#>
SELECT ?state ?demand ?capacity
WHERE {
    { SELECT ?state (SUM(?watts) AS ?demand)
      WHERE { ?load eccomponent:activeIn ?state ; eccomponent:powerDemand ?watts }
      GROUP BY ?state }
    ?supply eccomponent:powerCapacity ?capacity .
}
""")

rows = result["rows"]
if not rows:
    display("<p>No state demands or supply capacity are recorded, so sufficiency cannot be checked.</p>")
else:
    # The state that draws the most power decides whether the supply is sufficient.
    worst = max(rows, key=lambda row: float(row["demand"]))
    state = worst["state"].split("#")[-1]
    demand = float(worst["demand"])
    capacity = float(worst["capacity"])
    margin = capacity - demand
    verdict = "sufficient" if margin >= 0 else "NOT sufficient"
    display(f"<p>Peak demand is <b>{demand:.1f} W</b> in <b>{state}</b>. Supply capacity is {capacity:.1f} W, "
            f"leaving a margin of <b>{margin:.1f} W</b>. The supply is <b>{verdict}</b>.</p>")
```

**Gap check: loads with demand but no operating state.** These are never counted in any state total.

```table
PREFIX eccomponent: <http://www.example.com/component#>

SELECT ?Load ?Demand_W
WHERE {
    ?Load eccomponent:powerDemand ?Demand_W .
    FILTER(?Demand_W > 0)
    FILTER NOT EXISTS { ?Load eccomponent:activeIn ?state }
}
```

**Gap check: loads with demand but no power feed.** No energy connection reaches the load or anything that contains it.

```table
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX component: <https://www.modelware.io/sierra/component#>
PREFIX eccomponent: <http://www.example.com/component#>

SELECT ?Load ?Demand_W
WHERE {
    ?Load eccomponent:powerDemand ?Demand_W .
    FILTER(?Demand_W > 0)
    FILTER NOT EXISTS {
        ?feeder base:contains* ?Load ;
            component:hasPort ?port .
        ?connection oml:hasTarget ?port ;
            component:transfers/a component:Energy .
    }
}
ORDER BY DESC(?Demand_W)
```

**Interpretation.** Sufficiency is judged at nominal steady-state demand. Inrush current and conversion losses are not modeled. An empty gap table means no such load is recorded.
