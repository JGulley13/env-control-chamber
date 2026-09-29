---
template:
  id: http://www.example.com/method/analysis/mass-rollup
  name: "Mass Rollup"
  rank: 0
  expose:
    - kind: compose
---
## Mass Rollup

**Question.** What is the total mass of the system, and does it exceed the structure's capacity?

```tree
---
columns: { this: { label: "Component" }, totalMass: { label: "Total Mass (kg)" } }
containment: [ base:contains ]
---
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX component: <https://www.modelware.io/sierra/component#>
PREFIX analysis: <http://www.example.com/method/analysis#>

CONSTRUCT {
    ?component base:contains ?child .
    ?component analysis:totalMass ?totalMass .
}
WHERE {
    {
        # value x unit multiplier = kg, summed over the component and everything it contains
        SELECT ?component (SUM(xsd:decimal(?value) * xsd:decimal(?multiplier)) AS ?totalMass)
        WHERE {
            ?component base:contains* ?part .
            ?part component:mass ?quantity .
            ?quantity oml:value ?value ;
                oml:unit/oml:multiplier ?multiplier .
        }
        GROUP BY ?component
    }
    OPTIONAL { ?component base:contains ?child }
}
```

**Capacity check.** The top-level system's total mass against its recorded `structuralCapacity`, in kg.

```table
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX component: <https://www.modelware.io/sierra/component#>
PREFIX eccomponent: <http://www.example.com/component#>

SELECT ?System ?TotalMass_kg ?Capacity_kg ?WithinCapacity
WHERE {
    {
        SELECT ?System (SUM(xsd:decimal(?value) * xsd:decimal(?multiplier)) AS ?TotalMass_kg)
        WHERE {
            ?System base:contains* ?part .
            FILTER NOT EXISTS { ?System base:isContainedBy ?container }
            ?part component:mass ?quantity .
            ?quantity oml:value ?value ;
                oml:unit/oml:multiplier ?multiplier .
        }
        GROUP BY ?System
    }
    OPTIONAL { ?System eccomponent:structuralCapacity ?Capacity_kg }
    BIND(IF(!BOUND(?Capacity_kg), "Not recorded",
         IF(?TotalMass_kg <= ?Capacity_kg, "Yes", "No")) AS ?WithinCapacity)
}
```

**Interpretation.** Totals are derived from leaf masses each time the page renders. No composite stores its own total. "Not recorded" means the question cannot be answered until a capacity is entered.
