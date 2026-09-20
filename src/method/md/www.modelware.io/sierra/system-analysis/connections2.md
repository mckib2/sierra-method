### Assignment 4, Step 2: focal type, structure, expected content, and rules

I'd like to split this notebook into two parts: A) the ports and B) the connections.

#### 1. Part A: Component Ports (component:Port)

- Focal Type: component:Port
- Properties:
    • component:portOf: Parent component (component:Component, maxCount 1)
    • component:direction: "In" or "Out" (component:Direction, maxCount 1)
    • base:description: Description / notes (dash:TextAreaEditor, maxCount 1)
- Editor: table-editor

Let's make some authoring rules (that may or may not be stricter than what is specified by the TBox specs):
- port must have no more than 1 parent component
- port must have no more than 1 direction, one of "In" or "Out"
- port must have a single description (this is more strict -- for fun!)

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
- connection must have a single description (again, more strict for fun!)


#### 1. component:PortShape

 ```shacl
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
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    .
```

#### 2. component:ConnectionShape

```shacl
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
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    .
```
