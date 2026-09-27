---
# NBPM: "Package one analysis or view as a reusable compose template" -- modular template for coverage!
template:
  id: https://www.modelware.io/sierra/system-analysis/coverage
  name: "Subsystem Interface Coverage Matrix"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
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
