# persona-ontology

The persona ontology is an application profile over existing OWL ontologies. It models the domain of natural people in the Mee Identity Agent. It documents in SHACL which classes and properties from other ontologies are used to describe a person. 

## Purpose

It defines a formal, machine-readable model of a real-world person's identity data — names, addresses, phone numbers, SSNs, height, email, etc. — by reusing and constraining existing well-known ontologies rather than inventing new ones.

## Key components

- **`persona.ttl`** — This application ontology extends IEEE Person Ontology (PO) PersonOntology, PostalAddressOntology, 
                    and StagingOntology for the Mee IdAgent application

- **`persona-shacl.ttl`** — SHACL (Shapes Constraint Language) rules that define *which* classes and properties are valid and *how* data must be structured. For example:
  - A `Person` must have designators (name, SSN, phone, email, address)
  - A `PersonName` must have at least one `GivenName` and exactly one `FamilyName`
  - An `SSN` must match the pattern `\d{3}-\d{2}-\d{4}`
  - A `TemporalRegion` must have a start date

- **`example.ttl`** — A concrete RDF instance of a person ("Margery Alice Walker", who goes by "Alice Walker") showing how the ontology is used in practice, with her name, SSN, phone, email, home address, height, and address tenure.



