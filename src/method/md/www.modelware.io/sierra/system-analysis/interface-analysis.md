---
# NBPM: "Package one analysis or view as a reusable compose template" -- we did it!
template:
  id: https://www.modelware.io/sierra/system-analysis/interface-analysis
  name: "System Interface Connectivity Analysis"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---

# System Interface Connectivity Analysis

This is a "Small Analysis Layer Over Fire Force" that evaluates subsystem interconnections, flow conformance w.r.t. direction, interface coverage, and any orphan interfaces we can find.

## 1. Subsystem Interface Coverage

```compose
template: https://www.modelware.io/sierra/system-analysis/coverage
```

## 2. System Interface Graph

```compose
template: https://www.modelware.io/sierra/system-analysis/connectivity-graph
```

## 3. Interface Flow and Rule Conformance

```compose
template: https://www.modelware.io/sierra/system-analysis/conformance
```

## 4. Orphan Interface Detection

```compose
template: https://www.modelware.io/sierra/system-analysis/orphans
```
