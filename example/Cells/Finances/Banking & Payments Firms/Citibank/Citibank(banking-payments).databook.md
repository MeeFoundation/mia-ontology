---
id: http://www.example.org/mia/cells/cell-04
title: "Citibank"
type: cell-databook
version: 2.0.0
created: 2026-07-10
description: >
  Cell DataBook for folder "Citibank" (cell:category: cat:BankingPayments). It is a two-member
  cell (member entries about :Citibank and :Self) with two topic graphs about :Self — one Alice's
  own self-asserted service-account username/password, the other Citibank's own claimed record
  (debit card, checking account, online service account).
mia:
  category: "cat:BankingPayments"
  creator: ":Self"
  owner: ":Self"
  member:
    - id: "http://www.example.org/mia/graphs/graph-27"
      claimant: ":Self"
      subject: ":Citibank"
      template: "pshapes:ContactInfoShape"
    - id: "http://www.example.org/mia/graphs/graph-77"
      claimant: ":Self"
      subject: ":Self"
      template: "pshapes:ContactInfoShape"
  topic:
    - id: "http://www.example.org/mia/graphs/graph-75"
      claimant: ":Self"
      subject: ":Self"
      template: "sashapes:ServiceAccountShape"
    - id: "http://www.example.org/mia/graphs/graph-76"
      claimant: ":Citibank"
      subject: ":Self"
      template:
        - "sashapes:ServiceAccountShape"
        - "bankingshapes:DebitCardShape"
        - "bankingshapes:CheckingAccountShape"
---

## Graphs

<a id="graph-27"></a>
### Graph 27

#### Overview

This graph captures Alice Walker's own self-claimed notes about Citibank as an institution — her own record of the organization, distinct from Citibank's own claimed record about her (graph 76). Alice is the claimant.

#### Graph

```turtle
<!-- databook:id: alice-citibank-org-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-27#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix o: <http://mee.foundation/ontologies/organization#> .
@prefix cco: <https://w3id.org/cco-domains/cco/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

:Citibank rdf:type owl:NamedIndividual ,
                   o:Organization ;
    rdfs:label "Citibank"@en ;
    o:hasWebsite "https://citibank.com"^^xsd:anyURI ;

    <https://w3id.org/cco-domains/cco/ont00001917> [  # described by → Organization Note
        rdf:type cco:ent00000048 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "My primary checking account is here — used for rent and bill-pay autopay. Opened 2019."
    ] .
```

<a id="graph-77"></a>
### Graph 77

#### Overview

This graph is one of the cell's two required `member` entries, claimed by and about `:Self` — a minimal given-name stub, so `:Self` is a genuine member of this cell (see Check 21) alongside `:Citibank` (graph 27), matching the diagram's two member circles. Note: this makes the cell's own `member` subjects (`:Self`, `:Citibank`) overlap with its `topic` subject (`:Self`, from graphs 75/76) — a known, deliberately-deferred Check 18 tension.

#### Graph

```turtle
<!-- databook:id: alice-citibank-member-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-77#graph -->
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
    ] ,
    [  # designated by → OrganizationName (ContactInfoShape)
        rdf:type cco:ent00000047 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "Acme"
    ] ,
    [  # designated by → EmailAddress (ContactInfoShape)
        rdf:type cco:ent00000024 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "alice@acme.com"
    ] .
```

<a id="graph-75"></a>
### Graph 75

#### Overview

This graph is Alice's own self-asserted claim about her Citibank online service account — just its username and password, as she herself knows them — distinct from Citibank's own claimed record of the same account (graph 76). Alice is the claimant.

#### Graph

```turtle
<!-- databook:id: alice-citibank-self-asserted-account-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-75#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix persona: <http://mee.foundation/ontologies/persona#> .
@prefix cco: <https://w3id.org/cco-domains/cco/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix serviceaccounts: <http://mee.foundation/ontologies/service-accounts#> .

:Self rdf:type owl:NamedIndividual ,
               persona:Person .

:Alice_Citibank_Online rdf:type owl:NamedIndividual ,
                     serviceaccounts:ServiceAccount ,
                     cco:ent00000033 ;  # Online Service Account
    cco:ent00000035 "awalker@gmail.com" ;           # has user handle (username), self-known
    serviceaccounts:hasPassword "C1t1b@nk#2024!" .          # has password, self-known

:Self <https://w3id.org/cco-domains/cco/ent00000045> :Alice_Citibank_Online .  # holds user account
```

<a id="graph-76"></a>
### Graph 76

#### Overview

This graph captures Alice Walker's financial relationship with Citibank. Citibank is a PDN Organization node which directly claims the information about Alice in this graph. The information in this graph has been transmitted from the Citibank PDN node to Alice's own instance of the app. It includes a VISA debit card linked to a checking account, plus an online service account for online.citi.com. Citibank is the claimant.

#### Graph

```turtle
<!-- databook:id: citibank-graph -->
<!-- databook:graph: http://www.example.org/mia/graphs/graph-76#graph -->
@prefix : <http://www.example.org/mia#> .
@prefix persona: <http://mee.foundation/ontologies/persona#> .
@prefix o: <http://mee.foundation/ontologies/organization#> .
@prefix cco: <https://w3id.org/cco-domains/cco/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix serviceaccounts: <http://mee.foundation/ontologies/service-accounts#> .
@prefix banking: <http://mee.foundation/ontologies/banking#> .

:Self rdf:type owl:NamedIndividual ,
               persona:Person .

:Citibank rdf:type owl:NamedIndividual ,
                   o:Organization ;
    rdfs:label "Citibank"@en .

:Self rdfs:comment "Alice Walker regarding her Citibank relationship."@en ;
    cco:ent00000073 :Alice_Debit_Card ;             # has payment card
    persona:hasBankAccount :Alice_Checking_Account ;
    cco:ent00000045 :Alice_Citibank_Online .  # holds user account

:Alice_Debit_Card rdf:type owl:NamedIndividual ,
                           banking:DebitCard ,
                           cco:ent00000051 ;  # Debit Card
    rdfs:label "Alice Walker's VISA Debit Card"@en ;
    <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → Card Number (PAN)
        rdf:type cco:ent00000052 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "4111-1111-1111-1111"
    ] ;
    <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → CVV
        rdf:type cco:ent00000053 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "123"
    ] ;
    cco:ent00000070 [  # has expiration date → Calendar Date Identifier
        rdf:type cco:ont00001340 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "12/28"
    ] ;
    banking:accessesBankAccount :Alice_Checking_Account .

:Alice_Checking_Account rdf:type owl:NamedIndividual ,
                                 banking:CheckingAccount ;
    rdfs:label "Alice Walker's Citibank Checking Account"@en ;
    <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → Checking Account Number
        rdf:type cco:ent00000071 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "9876543210"
    ] ;
    <https://w3id.org/cco-domains/cco/ont00001879> [  # designated by → Routing Number
        rdf:type cco:ent00000072 ;
        <https://w3id.org/cco-domains/cco/ont00001765> "021000089"
    ] .

:Alice_Citibank_Online rdf:type owl:NamedIndividual ,
                                serviceaccounts:ServiceAccount ,
                                cco:ent00000033 ;  # Online Service Account
    rdfs:label "Alice Walker's Citibank Online Account"@en ;
    cco:ent00000034 "Citibank" ;                   # has service name
    cco:ent00000035 "awalker@gmail.com" ;           # has user handle (username)
    cco:ent00000036 "https://online.citi.com"^^xsd:anyURI ;  # has service URI
    serviceaccounts:hasPassword "C1t1b@nk#2024!" .         # has password
```
