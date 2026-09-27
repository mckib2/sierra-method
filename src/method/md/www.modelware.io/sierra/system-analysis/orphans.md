---
# NBPM: "Package one analysis or view as a reusable compose template" -- modular template for orphans!
template:
  id: https://www.modelware.io/sierra/system-analysis/orphans
  name: "Orphan Interface Detection"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---

## 4. Orphan Interface Detection

Let's try to detect ports that are declared on subsystems but which have no incoming or outgoing connections (i.e., `FILTER NOT EXISTS`).

```table
---
stylesheet:
  - selector: cell[col === "Impact"]
    style:
      color: #c0392b
      font-weight: bold
---
# NBPM: query 4/5 (Orphan)

PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT ?OrphanPort ?Component ?Direction ?Impact
WHERE {
  ?p a component:Port .
  OPTIONAL { ?p component:direction ?dir }
  OPTIONAL { ?p (component:portOf|^component:hasPort) ?comp }
  FILTER NOT EXISTS {
    ?conn a component:Connection .
    { ?conn oml:hasSource ?p } UNION { ?conn oml:hasTarget ?p }
  }
  BIND(REPLACE(STR(?p), "^.*#", "") AS ?OrphanPort)
  BIND(IF(BOUND(?comp), REPLACE(STR(?comp), "^.*#", ""), "Unassigned") AS ?Component)
  BIND(COALESCE(?dir, "Unspecified") AS ?Direction)
  BIND(
    IF(?Direction = "In", "Unrouted Command/Power Input (Subsystem Inoperable)", "Unrouted Telemetry/Signal Output (Unmonitored Subsystem)")
    AS ?Impact
  )
}
ORDER BY ?Component ?OrphanPort
```
