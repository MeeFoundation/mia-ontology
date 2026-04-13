# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **RDF/OWL ontology project** — a formal semantic knowledge model for representing natural people's identity data in the Mee Identity Agent (MIA). It is an *application profile* ontology: it reuses and constrains existing well-known ontologies rather than defining new terms.

There are no build, compile, test, or lint commands. The files are Turtle (`.ttl`) loaded into semantic web tools (Protégé).

## Core Files

| File | Purpose |
|------|---------|
| `persona.ttl` | Main application ontology — imports domain ontologies, annotates which classes/properties are required vs. optional for Mee |
| `persona-shacl.ttl` | SHACL validation shapes — constraint rules for valid instance data (e.g., a Person must have FullName OR GivenName+FamilyName) |
| `example.ttl` | Concrete RDF instance data for "Alice Walker" — the canonical usage example |
| `project_files/` | Reference materials: imported domain ontologies (PersonOntology.ttl, AddressOntology.ttl, StagingOntology.ttl), BFO/CCO source files, PDFs, docs |

## Architecture

### Three-Layer Design

```
example.ttl
  └─ imports → persona.ttl (application profile)
               ├─ imports → PersonOntology.ttl
               ├─ imports → AddressOntology.ttl
               └─ imports → StagingOntology.ttl
                             └─ imports → BFO terms

persona-shacl.ttl
  └─ imports → persona.ttl
```

1. **Foundation**: BFO (Basic Formal Ontology) — provides temporal modeling (`TemporalInterval`) and core relations
2. **Domain Ontologies** (in `project_files/`): PersonOntology, AddressOntology, StagingOntology
3. **Application Profile** (`persona.ttl`): aggregates domain ontologies; uses annotation properties (`usesRequiredClass`, `usesOptionalClass`, `usesCCOClass`, `usesCCOProperty`) to document Mee's usage

### Key Architectural Patterns

**Peer name pattern** (not hierarchical): All name types (FullName, GivenName, FamilyName, AlternateName) connect directly to Person via `ont00001879` (designated by). They are siblings, not nested.

**Address history pattern**: `AddressDesignation` links Person → Address → `TemporalInterval`. Open-ended intervals (no `hasEndDate`) indicate current address. See `TEMPORAL_TRACKING_SOLUTION.md` for details.

### Key Identifiers

Classes and properties use numeric IRIs. The most common:

- `ont00001262` = Person
- `ont00001879` = designated by (Person ← name/identifier)
- `ont00001765` = has text value (designator → literal string)
- `ent00000001`–`ent00000006` = name types (FullName, GivenName, AdditionalName, FamilyName, _, AlternateName)
- `ent00000008` = SSN; `ent00000023` = Phone; `ent00000024` = Email
- `ent00000010` = PostalAddress; `ent00000016` = AddressDesignation
- `BFO_0000038` = TemporalInterval; `ent00000017/18` = hasStartDate/hasEndDate

## Versioning

Before committing any change to `example.ttl`, `persona.ttl`, or `persona-shacl.ttl`, increment the **minor version number** in that file's `owl:versionInfo` annotation and update the description to summarise the change. For example:

```
owl:versionInfo "Version 3.0.3 - added social network"@en
```
becomes:
```
owl:versionInfo "Version 3.0.4 - added birth date"@en
```

## Keeping Files in Sync

Whenever changes are made to `example.ttl` or `persona.ttl`, `persona-shacl.ttl` must be updated to match:

- **New property usage in `example.ttl`** (e.g., a new physical characteristic, relationship, or identifier added to a Person instance) → add or extend a SHACL shape to validate that property on the relevant target class.
- **New class or property defined in `persona.ttl`** (e.g., `persona:hasSocialNetwork`) → add a SHACL shape that constrains how instances of the domain class may or must use it.

Always update `persona-shacl.ttl` in the same edit session as the change that triggers it.

## Validation

**SHACL validation** (e.g., using Apache Jena's `shaclvalidate`):
```bash
shaclvalidate -datafile example.ttl -shapesfile persona-shacl.ttl
```

**Protégé**: Load `persona.ttl`; Protégé will import the domain ontologies via IRI resolution. Use the reasoner (HermiT/Pellet) to check consistency.

## Gitignore Notes

`catalog-v001.xml` and `/project_files` are gitignored (Protégé IDE artifacts). The `project_files/` directory exists locally but is not tracked — it contains source domain ontologies and reference documents.
