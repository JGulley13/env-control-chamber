---
template:
  id: http://www.example.com/method/system-analysis/connections
  name: "Connections"
  rank: 0
  expose:
    - kind: compose
---

## Live Pathways

The table is ordered by source part. It displays one row per connection and includes the part owning each endpoint.

```python
import html

result = await query("""
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX component: <https://www.modelware.io/sierra/component#>
SELECT DISTINCT
	?sourcePart ?sourcePartLabel ?sourcePort ?sourcePortLabel
	?connection ?connectionLabel ?item ?itemLabel
	?receivingPort ?receivingPortLabel ?receivingPart ?receivingPartLabel
WHERE {
	?connection a component:Connection ;
		oml:hasSource ?sourcePort ;
		oml:hasTarget ?receivingPort ;
		component:transfers ?item .
	OPTIONAL {
		?sourcePart a component:Component ; component:hasPort ?sourcePort .
		OPTIONAL { ?sourcePart rdfs:label ?sourcePartLabel }
	}
	OPTIONAL {
		?receivingPart a component:Component ; component:hasPort ?receivingPort .
		OPTIONAL { ?receivingPart rdfs:label ?receivingPartLabel }
	}
	OPTIONAL { ?sourcePort rdfs:label ?sourcePortLabel }
	OPTIONAL { ?connection rdfs:label ?connectionLabel }
	OPTIONAL { ?item rdfs:label ?itemLabel }
	OPTIONAL { ?receivingPort rdfs:label ?receivingPortLabel }
}
ORDER BY ?sourcePartLabel ?sourcePart ?sourcePortLabel ?connection
""")

rows = result["rows"]

def pathway_label(label, iri):
	value = label or (str(iri).rsplit("#", 1)[-1].rsplit("/", 1)[-1] if iri else "Unassigned part")
	return html.escape(str(value))

if not rows:
	display("<p><em>No complete item flows are defined.</em></p>")
else:
	headers = ["Source Part", "Source Port", "Connection", "Flow Item", "Receiving Port", "Receiving Part"]
	body = "".join(
		"<tr>" + "".join(
			f"<td>{pathway_label(row.get(label_key), row.get(iri_key, ''))}</td>"
			for label_key, iri_key in [
				("sourcePartLabel", "sourcePart"),
				("sourcePortLabel", "sourcePort"),
				("connectionLabel", "connection"),
				("itemLabel", "item"),
				("receivingPortLabel", "receivingPort"),
				("receivingPartLabel", "receivingPart"),
			]
		) + "</tr>"
		for row in rows
	)
	header = "".join(f"<th>{html.escape(name)}</th>" for name in headers)
	display(
		'<table class="oml-md-table">'
		f"<thead><tr>{header}</tr></thead><tbody>{body}</tbody></table>"
	)
```
