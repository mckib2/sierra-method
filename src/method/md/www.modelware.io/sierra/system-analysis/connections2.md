---
template:
  id: https://www.modelware.io/sierra/system-analysis/connections2
  name: "Ports 'n Connections"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---


### Assignment 4, Step 2: focal type, structure, expected content, and rules

I'd like to split this notebook into two parts: A) the ports and B) the connections.

#### 1. Part A: Component Ports (component:Port)

- Focal Type: component:Port
- Properties:
    • component:portOf: Parent component (component:Component, maxCount 1)
    • component:direction: "In" or "Out" (component:Direction, maxCount 1)
    • base:description: Description / notes (dash:TextAreaEditor, maxCount 1)
- Editor: table-editor

Let's make some authoring rules:
- port must have no more than 1 parent component
- port must have no more than 1 direction, one of "In" or "Out"
- port must have no more than 1 description

#### 2. Part B: Component Connections (component:Connection)

- Focal Type: component:Connection (relation entity)
- Properties:
    - oml:hasSource: Source port (component:Port, minCount 1, maxCount 1)
    - oml:hasTarget: Target port (component:Port, minCount 1, maxCount 1)
    - component:transfers: Transferred item (base:Item, e.g. Signal, Energy, Material)
    - base:description: Textual description (dash:TextAreaEditor, maxCount 1)
- Editor: table-editor

Similarly, let's impose some construction rules:
- connection must have a single source port
- connection must have a single target port
- connection can specify what is transfered
- connection must have no more than 1 description


#### 1. component:Port Shape

```table-editor
---
columns: { this: { label: "Port" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix component: <https://www.modelware.io/sierra/component#> .

component:PortShape
    a sh:NodeShape ;
    sh:targetClass component:Port ;
    sh:property [
        sh:path component:portOf ;
        sh:name "Component" ;
        sh:class component:Component ;
        sh:maxCount 1 ;
        sh:order 1 ;  # NBPM: put in orders for fun and to be explicit
    ] ;
    sh:property [
        sh:path component:direction ;
        sh:name "Direction" ;
        sh:in ( "In" "Out" ) ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;  # NBPM: dash!
        #sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    .
```

#### 2. component:Connection Shape

```table-editor
---
columns: { this: { label: "Connection" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
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
        sh:name "Target Port" ;
        sh:class component:Port ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path component:transfers ;
        sh:name "Transferred Item" ;
        sh:class base:Item ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        #sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;

    sh:sparql [
        sh:message "Hey, bud!  A connection must flow from an 'Out' port to an 'In' port." ;
        sh:select """
            PREFIX oml: <http://opencaesar.io/oml#>
            PREFIX component: <https://www.modelware.io/sierra/component#>
            SELECT $this WHERE {
                $this oml:hasSource ?src ;
                      oml:hasTarget ?tgt .
                ?src component:direction ?srcDir .
                ?tgt component:direction ?tgtDir .
                FILTER (?srcDir != "Out" || ?tgtDir != "In")
            }
        """ ;
    ] ;

    # NBPM: I was unhappy with the above validation we were achieving with SHACL, so I imagined
    #       a more interesting thing we could check: make sure we have no short-circuit conditions!
    sh:sparql [
    sh:message "Short-circuit violation: Cannot connect two ports belonging to the same component." ;
    sh:select """
        PREFIX oml: <http://opencaesar.io/oml#>
        PREFIX component: <https://www.modelware.io/sierra/component#>
        SELECT $this WHERE {
            $this oml:hasSource ?src ;
                    oml:hasTarget ?tgt .
            ?src component:portOf ?comp .
            ?tgt component:portOf ?comp .
        }
    """ ;
    ] ;
    .
```

#### 3. Active Transfer Items

We could imagine that someone might want to know about which transfer items have been associated with connections, so let's make a table that shows that.

```table-editor
---
columns: { this: { label: "Transfer Item" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix component: <https://www.modelware.io/sierra/component#> .

component:ActiveItemShape
    a sh:NodeShape ;
    sh:targetClass base:Item ;
    dash:readOnly true ;
    sh:property [
        sh:path component:isTransferedBy ;
        sh:name "Transferred By (Connections)" ;
        sh:class component:Connection ;
        dash:readOnly true ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        dash:readOnly true ;
        sh:order 2 ;
    ] ;
    .
```

