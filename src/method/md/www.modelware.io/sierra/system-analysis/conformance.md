---
# NBPM: "Package one analysis or view as a reusable compose template" -- modular template for conformance!
template:
  id: https://www.modelware.io/sierra/system-analysis/conformance
  name: "Interface Flow and Rule Conformance"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---

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
