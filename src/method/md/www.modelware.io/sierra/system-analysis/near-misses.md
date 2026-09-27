---
# NBPM: "Package one analysis or view as a reusable compose template" -- modular template for near misses!
template:
  id: https://www.modelware.io/sierra/system-analysis/near-misses
  name: "Near-Miss Interface Analysis"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---

## Near-Miss Interface Detection

Identifies subsystems that have declared ports and at least one active connection, but still have unconnected interfaces:

```table
---
columns: { component: { label: "Subsystem" }, totalPorts: { label: "Total Ports" }, connectedPorts: { label: "Connected Ports" }, unconnectedPorts: { label: "Unconnected Ports" }, nearMissCondition: { label: "Condition" } }
stylesheet:
  - selector: cell[col === "unconnectedPorts" && Number(value) > 0]
    style:
      background-color: #fff3cd
      color: #856404
      font-weight: bold
---
# NBPM: query 5/5 (Near miss) -- we did it, boys!

PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT ?component 
       (COUNT(DISTINCT ?port) AS ?totalPorts) 
       (COUNT(DISTINCT ?connectedPort) AS ?connectedPorts)
       ((COUNT(DISTINCT ?port) - COUNT(DISTINCT ?connectedPort)) AS ?unconnectedPorts)
       ("Partial Interface Integration" AS ?nearMissCondition)
WHERE {
  ?comp a component:Component ;
        (component:hasPort|^component:portOf) ?port .
  
  OPTIONAL {
    ?conn a component:Connection .
    { ?conn oml:hasSource ?port } UNION { ?conn oml:hasTarget ?port }
    BIND(?port AS ?connectedPort)
  }

  BIND(REPLACE(STR(?comp), "^.*#", "") AS ?component)
}
GROUP BY ?comp ?component
HAVING (COUNT(DISTINCT ?connectedPort) > 0 && COUNT(DISTINCT ?connectedPort) < COUNT(DISTINCT ?port))
ORDER BY DESC(?unconnectedPorts)
```
