# Persona Ontology

The persona ontology is an application profile over existing ontologies. It models the domain of natural people in the Mee Identity Agent (MIA). It documents in SHACL which classes and properties from other ontologies are used to describe a person.

## Purpose

It defines a formal, machine-readable model of a real-world person's identity data — names, addresses, phone numbers, SSNs, physical characteristics, family relationships, social connections, and more — by reusing and constraining existing well-known ontologies rather than inventing new ones.

## Ontological Foundation

Built on **BFO** (Basic Formal Ontology) and **CCO** (Common Core Ontologies), specifically:
- **PersonOntology** — person, name types, family relationships
- **AddressOntology** — postal address structure
- **StagingOntology** — temporal modeling (address history)
- **AgentOntology** — physical qualities, social networks, interpersonal relationships

## Key Components

- **`persona.ttl`** — The application ontology. Imports the domain ontologies above and documents which classes and properties Mee uses (required vs. optional). Also defines Mee-specific extension properties such as `persona:hasSocialNetwork`.

- **`persona-shacl.ttl`** — SHACL constraint rules defining how instance data must be structured. Validates:
  - Person naming: FullName OR (GivenName + FamilyName) required; AlternateName (maiden name, preferred name) optional
  - Identifiers: SSN format (`NNN-NN-NNNN`), email format, phone (E.164)
  - Address: required street, city, state (USPS 2-letter), ZIP; optional country
  - Physical characteristics: hair color (text value), eye color
  - Family relationships: `is mother of` / `has mother` range must be a Person
  - Social network: optional (0..1), members must be Persons

- **`example.ttl`** — A concrete RDF instance for "Margery Alice Walker" (goes by "Alice Walker") demonstrating the ontology in practice:
  - Names: legal name, middle name, preferred name, maiden name (Margery Alice Arnold)
  - Identifiers: SSN, email, phone
  - Addresses: current (Paradise, CA, 2025–present) and previous (Boston, MA, 2020–2025) with temporal intervals
  - Physical characteristics: height (68 in), eye color (blue), hair color (grey)
  - Family: son Paul Walker, linked via CCO `is mother of` / `has mother`
  - Social network: partitioned into Family, Colleague, and Other sub-groups (CCO Social Networks linked via BFO `has part`), with Paul Walker in both Family and Colleague to demonstrate intersection

## Architecture

Names follow a **peer pattern** — all name types (FullName, GivenName, FamilyName, AlternateName) connect directly to Person via `designated by` (`ont00001879`). They are siblings, not nested under a PersonName parent.

Address history is tracked with **temporal intervals** — each address designation has a start date and an optional end date; absence of an end date indicates the current address.

## Validation

```bash
shaclvalidate -datafile example.ttl -shapesfile persona-shacl.ttl
```
