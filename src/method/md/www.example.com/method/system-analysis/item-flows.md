---
template:
  id: http://www.example.com/method/system-analysis/item-flows
  name: "Item Flows"
  rank: 0
  expose:
    - kind: compose
---
# Item Flows

Define each connection as one item flow between two component ports. The source and receiving parts are determined by the components that own those ports.

```table-editor
---
columns: { this: { label: "Connection" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix component: <https://www.modelware.io/sierra/component#> .

component:ConnectionShape
    a sh:NodeShape ;
    sh:targetClass component:Connection ;
    sh:property [
        sh:path oml:hasSource ;
        sh:name "Source Port" ;
        sh:class component:Port ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path oml:hasTarget ;
        sh:name "Receiving Port" ;
        sh:class component:Port ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path component:transfers ;
        sh:name "Flow Item" ;
        sh:class component:Item ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    .
```