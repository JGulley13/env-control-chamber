---
template:
  id: http://www.example.com/method/analysis/verification-coverage
  name: "Verification Coverage"
  rank: 0
  expose:
    - kind: compose
---
## Verification Coverage

**Question.** Is every requirement on its way to being verified?

**Gap check (orphans):** requirements with no verification case. The method expects every requirement to have one, High priority first.

```table
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?Requirement ?Priority
WHERE {
    ?Requirement a stakeholder:Requirement .
    OPTIONAL { ?Requirement base:priority ?Priority }
    FILTER NOT EXISTS { ?case verification:verifies ?Requirement }
}
ORDER BY ?Priority ?Requirement
```

**Near miss:** requirements that have a verification case, but none has passed yet. One passing result closes each.

```table
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT DISTINCT ?Requirement ?Case ?Status
WHERE {
    ?Case verification:verifies ?Requirement ;
        verification:status ?Status .
    FILTER NOT EXISTS {
        ?passed verification:verifies ?Requirement ;
            verification:status "Passed" .
    }
}
ORDER BY ?Requirement
```

**Coverage:** every requirement against every status. A `0` means no case is recorded in that combination. A row of all zeros is a requirement with no case at all.

```matrix
---
rowColumnLabel: "Requirement / Status"
stylesheet:
  - selector: cell[Number(value) > 0]
    style:
      background-color: "#D1FAE5"
---
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?row ?column (COUNT(?case) AS ?value)
WHERE {
    ?row a stakeholder:Requirement .
    VALUES ?column { "Not Started" "In Progress" "Passed" "Failed" }
    OPTIONAL {
        ?case verification:verifies ?row ;
            verification:status ?column .
    }
}
GROUP BY ?row ?column
ORDER BY ?row
```
