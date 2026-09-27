---
# NBPM: "Package one analysis or view as a reusable compose template" -- modular template for graph!
template:
  id: https://www.modelware.io/sierra/system-analysis/connectivity-graph
  name: "Subsystem Interface Graph"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---

## 2. System Interface Graph

Let's now visualize the active subsystem connections -- this time in a graph!

```graph
---
layout:
  mode: force
  running: true
  fit: true
  padding: 24
  force:
    repulsion: 3500
    linkDistance: 120
    springStrength: 0.008
    gravity: 0.0015
    damping: 0.90
stylesheet:
  - selector: node
    style:
      fill: #3498db
      stroke: #2c3e50
      stroke-width: 1
      color: white
  - selector: node [value.includes("FireSat")]
    style:
      fill: #9b59b6
  - selector: node [value.includes("Platform")]
    style:
      fill: #e67e22
  - selector: node [value.includes("Payload")]
    style:
      fill: #1abc9c
  - selector: node [value.includes("Propulsion")]
    style:
      fill: #e74c3c
---
# NBPM: query 2/5 (View graph)
# NBPM: view type graph 2/3 (graph)

PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

CONSTRUCT {
  ?srcComp component:connectedTo ?tgtComp .
  ?srcComp a component:Component .
  ?tgtComp a component:Component .
}
WHERE {
  ?conn a component:Connection ;
        oml:hasSource ?srcPort ;
        oml:hasTarget ?tgtPort .
  ?srcPort (component:portOf|^component:hasPort) ?srcComp .
  ?tgtPort (component:portOf|^component:hasPort) ?tgtComp .
}
```
