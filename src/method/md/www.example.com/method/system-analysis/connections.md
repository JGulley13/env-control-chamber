---
template:
  id: http://www.example.com/method/system-analysis/connections
  name: "Connections"
  rank: 0
  expose:
    - kind: compose
---

## Live Pathways
Each row shows one connection and the components that own its endpoint ports.

```table
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT DISTINCT
	?SourcePart ?SourcePort ?Connection ?FlowItem ?ReceivingPort ?ReceivingPart
WHERE {
	?Connection a component:Connection ;
		oml:hasSource ?SourcePort ;
		oml:hasTarget ?ReceivingPort ;
		component:transfers ?FlowItem .
	OPTIONAL { ?SourcePart component:hasPort ?SourcePort . }
	OPTIONAL { ?ReceivingPart component:hasPort ?ReceivingPort . }
}
ORDER BY ?SourcePart ?SourcePort ?Connection
```
