---
id: http://www.example.org/mia/cells/cell-35
title: "Acme"
type: cell-databook
version: 1.2.0
created: 2026-07-10
description: >
  Cell DataBook for folder "Acme" (cell:category: cat:Organization). It is a
  one-member cell with one member entry about :Self, plus one topic graph about
  :Acme itself — Alice's employer's own o:Organization profile, required now that
  cat:Organization's own TemplateCell is isTopicCell: true (Check 31).
mia:
  category: "cat:Organization"
  creator: ":Self"
  owner: ":Self"
  member:
    id: "http://www.example.org/mia/graphs/graph-53"
    claimant: ":Self"
    subject: ":Self"
    template: "pshapes:ContactInfoShape"
  topic:
    id: "http://www.example.org/mia/graphs/graph-94"
    claimant: ":Self"
    subject: ":Acme"
    template: "oshapes:OrganizationShape"
---

## Graphs

<a id="graph-53"></a>
### Graph 53

#### Overview

This graph is the cell's one required `member` entry — a cell with a single `member` entry in the user's own category-cell tree always has `:Self` as that member (see Check 21), regardless of what the cell's `subject` is — here, Alice herself. The "Acme" cell's own subject is Alice's employer, carried by its `cell:topic` (graph 94) rather than by this member stub. Alice is both the claimant and the subject. It carries her given name, satisfying the `ContactInfoShape` `ctpl:OrganizationTemplateCell` sets as `cell:memberGraphShape` — no longer deliberately empty.

#### Graph

```turtle
<!-- databook:id: alice-acme-member-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-53#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix cco: <https://w3id.org/cco-domains/cco/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix persona: <http://mee.foundation/ontologies/persona#> .

:Self rdf:type owl:NamedIndividual ,
               persona:Person .

:Self <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → GivenName
        rdf:type cco:ent00000002 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "Alice"
    ] .
```

<a id="graph-94"></a>
### Graph 94

#### Overview

This graph captures Acme, Alice's employer, as an `o:Organization` in its own right — the cell's
`cell:topic`, and what its derived subject resolves to (see integrity.md's Check 18). Alice self-enters
this record: Acme is not a PDN-interoperable node, so she is the claimant even though the graph is
about Acme.

#### Graph

```turtle
<!-- databook:id: alice-acme-org-profile-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-94#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix o: <http://mee.foundation/ontologies/organization#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

:Acme rdf:type owl:NamedIndividual ,
               o:Organization ;
    rdfs:label "Acme"@en ;

    o:numMembers 2400 ;
    o:hasWebsite "https://acme.example.com"^^xsd:anyURI .
```
