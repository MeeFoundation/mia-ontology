# Address History Tracking - Simple Pattern

## Overview

Track N addresses over a person's lifetime with temporal bounds (start date, optional end date).

**Use case:** "Alice moved to her postal address in September 2025, and still lives there at the present time"

## Pattern

Each address in a person's history is modeled as an **AddressDesignation** with:
- Person (who lived there)
- Address (where they lived)
- Time interval (when they lived there)
  - Start date (required)
  - End date (optional - absence indicates current address)

## Example

```turtle
# Alice lived in Boston from 2020-2025
:Boston_Residence rdf:type postal:AddressDesignation ;
    BFO_0000139 :Alice_Walker ;        # has participant (person)
    ont00001879 :Address_Boston ;      # designated by (address)
    BFO_0000153 :Interval_2020_2025 .  # occupies temporal region

:Interval_2020_2025 rdf:type BFO:TemporalInterval ;
    postal:hasStartDate "2020-01-01"^^xsd:date ;
    postal:hasEndDate "2025-08-31"^^xsd:date .

# Alice lives in Concord from 2025-present
:Concord_Residence rdf:type postal:AddressDesignation ;
    BFO_0000139 :Alice_Walker ;
    ont00001879 :Address_Concord ;
    BFO_0000153 :Interval_2025_Present .

:Interval_2025_Present rdf:type BFO:TemporalInterval ;
    postal:hasStartDate "2025-09-01"^^xsd:date .
    # No end date = current address
```

## Classes Used

### From PostalAddressOntology (v1.1.0)
- **postal:AddressDesignation** - Process of a person being associated with an address
  - Subclass of: BFO Process
  - Links person, address, and time interval

### From BFO
- **BFO_0000038** (TemporalInterval) - One-dimensional temporal region with duration
- **BFO_0000139** (has participant) - Links AddressDesignation to Person
- **BFO_0000153** (occupies temporal region) - Links AddressDesignation to TemporalInterval

### Properties
- **postal:hasStartDate** - Start date of temporal interval (xsd:date, required)
- **postal:hasEndDate** - End date of temporal interval (xsd:date, optional)

## Key Features

✅ **Simple** - One class, three properties, minimal complexity  
✅ **Address history** - Track N addresses over lifetime  
✅ **Open-ended intervals** - No end date = current address  
✅ **BFO conformant** - Uses standard BFO temporal modeling  
✅ **Extensible** - Can add address types/roles later if needed  

## Example Queries

**Where did Alice live in 2023?**
- Find AddressDesignation where:
  - has participant = Alice
  - interval overlaps 2023

**Where does Alice currently live?**
- Find AddressDesignation where:
  - has participant = Alice
  - no end date (open-ended interval)

**When did Alice live in Boston?**
- Find AddressDesignation where:
  - designated by = Boston address
  - return start/end dates

**List all of Alice's addresses chronologically**
- Find all AddressDesignations where:
  - has participant = Alice
  - order by start date

## Future Extensions

If needed later, this pattern can be extended to support:

### Multiple Simultaneous Addresses
```turtle
# Add type/role property
:designation postal:addressType "primary" .
:designation postal:addressType "mailing" .
```

### Move Events
```turtle
# Add physical relocation events if needed
:move rdf:type ont00000201 .  # Act of Change of Residence
```

### Address Roles
```turtle
# Distinguish mailing vs. billing vs. legal
:designation postal:hasRole postal:MailingRole .
```

## Testing

1. Load example.ttl in Protégé
2. Verify classes imported correctly:
   - postal:AddressDesignation
   - BFO_0000038 (TemporalInterval)
3. Verify instances created:
   - :Boston_Residence (2020-2025)
   - :Concord_Residence (2025-present)
4. Verify temporal intervals:
   - Boston has start AND end date
   - Concord has start date ONLY (current)

## SHACL Validation (Optional)

If you want to enforce this pattern:

```turtle
persona:AddressDesignationShape a sh:NodeShape ;
    sh:targetClass postal:AddressDesignation ;
    
    # Must have exactly one person
    sh:property [
        sh:path BFO_0000139 ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:class ont00001262  # Person
    ] ;
    
    # Must have exactly one address
    sh:property [
        sh:path ont00001879 ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:class postal:USPostalAddress
    ] ;
    
    # Must have exactly one temporal interval
    sh:property [
        sh:path BFO_0000153 ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:class BFO_0000038  # TemporalInterval
    ] .

persona:TemporalIntervalShape a sh:NodeShape ;
    sh:targetClass BFO_0000038 ;
    
    # Must have start date
    sh:property [
        sh:path postal:hasStartDate ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:date
    ] ;
    
    # End date is optional
    sh:property [
        sh:path postal:hasEndDate ;
        sh:maxCount 1 ;
        sh:datatype xsd:date
    ] .
```

## Summary

**Simplest possible model for address history:**
- Person lived at Address during Interval
- Interval has start date (required) and end date (optional)
- No end date = current address
- Extensible for future needs

This keeps the model simple while meeting the core requirement: track where a person lived over time with temporal bounds.
