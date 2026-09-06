---
id: http://www.example.org/mia/cells/cell-21
title: "Affiliations"
type: cell-databook
version: 1.2.0
created: 2026-07-10
description: >
  Cell DataBook for folder "Affiliations" (cell:category: cat:Affiliations). It is a
  one-member cell with one member entry about :Self — a minimal stub,
  since "Affiliations" is a purely organizational category node with no
  content or relationship of its own beyond Alice's required membership.
  Also carries an empty topic graph, required now that cat:Affiliations's own
  TemplateCell is isTopicCell: true (Check 31) — the real topic content
  lives in this category's own leaf cell (Boston Hub Society) instead.
mia:
  category: "cat:Affiliations"
  creator: ":Self"
  owner: ":Self"
  member:
    id: "http://www.example.org/mia/graphs/graph-39"
    claimant: ":Self"
    subject: ":Self"
    template: "pshapes:ContactInfoShape"
  topic:
    id: "http://www.example.org/mia/graphs/graph-93"
    claimant: ":Self"
    subject: ":Self"
---

## Graphs

<a id="graph-39"></a>
### Graph 39

#### Overview

This graph is the cell's one required `member` entry — a cell with a single `member` entry in the user's own category-cell tree always has `:Self` as that member (see Check 21), regardless of what the cell's `subject` is — here, Alice herself. The "Affiliations" cell is a purely organizational category node (`cell:category: cat:Affiliations`) with no relationship or subject of its own beyond Alice's required membership, so this stub carries no further claims. Alice is both the claimant and the subject. It carries her given name, satisfying the `ContactInfoShape` `ctpl:AffiliationsTemplateCell` sets as `cell:memberGraphShape` — no longer deliberately empty.

#### Graph

```turtle
<!-- databook:id: alice-affiliations-member-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-39#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix cco: <https://w3id.org/cco-domains/cco/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix persona: <http://mee.foundation/ontologies/persona#> .

:Self rdf:type owl:NamedIndividual ,
               persona:Person .

:Self <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → GivenName (ContactInfoShape)
        rdf:type cco:ent00000002 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "Alice"
    ] .
```

<a id="graph-93"></a>
### Graph 93

#### Overview

This graph is the cell's required `topic` — required now that `cat:Affiliations`'s own `TemplateCell` is `isTopicCell: true` (Check 31), even though "Affiliations" is a purely organizational scaffold cell with no topic content of its own (the real content lives in this category's own leaf cell, Boston Hub Society, instead). Deliberately empty — no triples at all, per Check 32's own allowance. Alice is both the claimant and the subject.

#### Graph

```turtle
<!-- databook:id: alice-affiliations-topic-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-93#graph -->
```
