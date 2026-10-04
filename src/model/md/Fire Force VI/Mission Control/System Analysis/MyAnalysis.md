---
ontology: https://fireforce6.github.io/mission-control/system-analysis/connections
---

## 1. Subsystem Interface Coverage

Shows direct connection counts between subsystems. Notice that 0s highlight interface gaps where no physical/logical channels are established.

```matrix
---
rowColumnLabel: Subsystem (Source / Target)
stylesheet:
  - selector: cell [Number(value) > 0]
    style:
      background-color: #d4edda
  - selector: cell [Number(value) == 0]
    style:
      background-color: #f8d7da
---
# NBPM: query 1/5 (Coverage)
# NBPM: view type matrix 1/3 (matrix)

PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  {
    SELECT DISTINCT ?rowUri ?colUri
    WHERE {
      ?c1 a component:Component ; (component:hasPort|^component:portOf) ?p1 .
      ?c2 a component:Component ; (component:hasPort|^component:portOf) ?p2 .
      BIND(?c1 AS ?rowUri)
      BIND(?c2 AS ?colUri)
    }
  }
  OPTIONAL {
    SELECT ?rowUri ?colUri (COUNT(DISTINCT ?conn) AS ?n)
    WHERE {
      ?conn a component:Connection ;
            oml:hasSource ?srcPort ;
            oml:hasTarget ?tgtPort .
      ?srcPort (component:portOf|^component:hasPort) ?rowUri .
      ?tgtPort (component:portOf|^component:hasPort) ?colUri .
    }
    GROUP BY ?rowUri ?colUri
  }
  BIND(REPLACE(STR(?rowUri), "^.*#", "") AS ?row)
  BIND(REPLACE(STR(?colUri), "^.*#", "") AS ?column)
}
ORDER BY ?row ?column
```


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


## 3. Interface Flow and Rule Conformance

Evaluates all connections against method rules:
- Connection must flow from an `Out` port to an `In` port (we did this previously in SHACL, but we can also do it with SPARQL!).
- Connection must transfer a designated item (e.g., Signal, Energy, Material).

```table
---
stylesheet:
  - selector: cell[col === "Status" && value === "FAIL"]
    style:
      background-color: #ffcccc
      font-weight: bold
  - selector: cell[col === "Status" && value === "PASS"]
    style:
      background-color: #d4edda
---
# NBPM: query 3/5 (Conformance)
# NBPM: view type table 3/3 (table) -- we're done with this assignment requirement! W00t!

PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT ?Connection ?SourcePort ?SourceDir ?TargetPort ?TargetDir ?TransferredItem ?Status
WHERE {
  ?conn a component:Connection ;
        oml:hasSource ?srcPort ;
        oml:hasTarget ?tgtPort .
  OPTIONAL { ?srcPort component:direction ?srcDir }
  OPTIONAL { ?tgtPort component:direction ?tgtDir }
  OPTIONAL { ?conn component:transfers ?item }
  BIND(REPLACE(STR(?conn), "^.*#", "") AS ?Connection)
  BIND(REPLACE(STR(?srcPort), "^.*#", "") AS ?SourcePort)
  BIND(COALESCE(?srcDir, "Missing") AS ?SourceDir)
  BIND(REPLACE(STR(?tgtPort), "^.*#", "") AS ?TargetPort)
  BIND(COALESCE(?tgtDir, "Missing") AS ?TargetDir)
  BIND(IF(BOUND(?item), REPLACE(STR(?item), "^.*#", ""), "None") AS ?TransferredItem)
  BIND(IF(?SourceDir = "Out" && ?TargetDir = "In" && BOUND(?item), "PASS", "FAIL") AS ?Status)
}
ORDER BY ?Status ?Connection
```
