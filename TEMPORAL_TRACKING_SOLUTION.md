# Temporal Tracking: Address History Solution

## Summary for Paul

You asked: *"We used a TemporalRegion to express that Alice moved to her postal address in September 2025, and still lives there at the present time."*

**Answer:** I've restored this capability with an improved architecture that distinguishes **physical relocation** from **ongoing address designation**.

---

## What Changed

### **Files Updated (2):**

1. **PostalAddressOntology.ttl** - Added 3 new elements:
   - `postal:AddressDesignation` class
   - `postal:hasStartDate` property  
   - `postal:hasEndDate` property

2. **example.ttl** - Added temporal tracking for Alice:
   - Physical move event (Sept 1, 2025)
   - Ongoing residence (Sept 2025 to present)

---

## The Solution: Two-Process Model

Your requirement involves **TWO distinct concepts:**

### **1. Physical Relocation (One-Time Event)**

**What it is:** Alice physically moved to Concord on a specific date.

**Uses CCO:** `ont00000201` (Act of Change of Residence)

**Example:**
```turtle
:Alices_Move_to_Concord rdf:type <https://purl.org/cco/ont00000201> ;  # CCO class
    rdfs:label "Alice's Relocation to Concord"@en ;
    BFO_0000139 :Alice_Walker ;  # has participant
    BFO_0000153 :Sept_1_2025 .   # occupies temporal region (instant)

:Sept_1_2025 rdf:type BFO:TemporalInstant ;
    ont00001765 "2025-09-01"^^xsd:date .
```

---

### **2. Ongoing Address Designation (Continuous Process)**

**What it is:** Alice has been associated with this address from Sept 2025 to present.

**New class:** `postal:AddressDesignation` (added to PostalAddressOntology)

**Example:**
```turtle
:Alices_Residence_at_Concord rdf:type postal:AddressDesignation ;
    rdfs:label "Alice's Residence at Concord Address"@en ;
    BFO_0000139 :Alice_Walker ;  # has participant
    ont00001879 :Home_Address_Concord ;  # designated by (the address)
    BFO_0000153 :Sept_2025_to_Present .  # occupies temporal region (interval)

:Sept_2025_to_Present rdf:type BFO:TemporalInterval ;
    postal:hasStartDate "2025-09-01"^^xsd:date .
    # No postal:hasEndDate = still current!
```

---

## Why Two Separate Concepts?

**Physical move ≠ Address designation**

- **Move:** A one-time event (happened Sept 1, 2025)
- **Residence:** An ongoing state (started Sept 2025, continues today)

**Benefits:**
- ✅ Can track multiple addresses over time (address history)
- ✅ Works for both persons AND organizations
- ✅ Clear semantics (move vs. live-there)
- ✅ Open-ended intervals (no end date = "still current")

---

## Address History Support

**Scenario:** Alice lived in Boston (2020-2025), moved to Concord (2025-present)

```turtle
# Previous address (ended)
:Alices_Boston_Residence rdf:type postal:AddressDesignation ;
    BFO_0000139 :Alice_Walker ;
    ont00001879 :Boston_Address ;
    BFO_0000153 :Jan_2020_to_Aug_2025 .

:Jan_2020_to_Aug_2025 rdf:type BFO:TemporalInterval ;
    postal:hasStartDate "2020-01-01"^^xsd:date ;
    postal:hasEndDate "2025-08-31"^^xsd:date .  # Has end date

# Current address (ongoing)
:Alices_Concord_Residence rdf:type postal:AddressDesignation ;
    BFO_0000139 :Alice_Walker ;
    ont00001879 :Concord_Address ;
    BFO_0000153 :Sept_2025_to_Present .

:Sept_2025_to_Present rdf:type BFO:TemporalInterval ;
    postal:hasStartDate "2025-09-01"^^xsd:date .
    # No end date = still current
```

**Query pattern:** "Find all addresses where Alice lived"
- Answer: Boston (2020-2025), Concord (2025-present)

---

## CCO Conformance

**Used existing CCO classes where available:**
- ✅ `ont00000201` (Act of Change of Residence) - for physical moves
- ✅ `BFO_0000153` (occupies temporal region) - for temporal bounds
- ✅ `BFO_0000148` (TemporalInstant) - for specific dates
- ✅ `BFO_0000038` (TemporalInterval) - for date ranges
- ✅ `BFO_0000139` (has participant) - to connect agent

**Added minimal extension:**
- 🆕 `postal:AddressDesignation` - NEW class (no CCO equivalent found)
- 🆕 `postal:hasStartDate` - Convenience property for xsd:date
- 🆕 `postal:hasEndDate` - Convenience property for xsd:date

---

## Testing in Protégé

**Load order:**
1. Open `PostalAddressOntology.ttl` (imports BFO, CCO)
2. Open `example.ttl` (imports persona.ttl, which imports PostalAddressOntology)

**What to verify:**
- ✅ `postal:AddressDesignation` class is visible
- ✅ `:Alices_Move_to_Concord` instance loads (type: Act of Change of Residence)
- ✅ `:Alices_Residence_at_Concord` instance loads (type: AddressDesignation)
- ✅ Temporal regions have correct dates
- ✅ No reasoner errors

---

## Your Original Pattern (Restored & Improved)

**OLD (what you had):**
```turtle
# Single temporal region
:AddressBearing_2025_Present rdf:type BFO:Process ;
    BFO_0000139 :Alice_Walker ;
    BFO_0000139 :HomeAddress_AW ;
    BFO_0000153 :Interval_2025_Present .
```

**NEW (what you have now):**
```turtle
# Separate move event and ongoing residence
:Alices_Move_to_Concord rdf:type ont00000201 ;  # CCO: Act of Change of Residence
    BFO_0000139 :Alice_Walker ;
    BFO_0000153 :Sept_1_2025 .  # Instant

:Alices_Residence_at_Concord rdf:type postal:AddressDesignation ;
    BFO_0000139 :Alice_Walker ;
    ont00001879 :Home_Address_Concord ;  # designated by
    BFO_0000153 :Sept_2025_to_Present .  # Interval
```

**Improvements:**
- Clearer semantics (move vs. live-there)
- Uses CCO's existing relocation class
- Works for address history (multiple addresses over time)
- Open-ended intervals properly modeled

---

## Future Extensions

**This pattern supports:**

1. **Multiple simultaneous addresses:**
   - Residential address
   - Mailing address  
   - Billing address

2. **Organization addresses:**
   - Company relocations
   - Branch office addresses
   - Historical address tracking

3. **Address roles:**
   - Primary residence
   - Secondary residence
   - Previous addresses

---

## Questions?

**Email me or open an issue in the repo!**

**Key Files:**
- `PostalAddressOntology.ttl` - Contains AddressDesignation class
- `example.ttl` - Shows Alice's move and ongoing residence
- Both ready to test in Protégé

---

**This restores the temporal tracking you requested while maintaining CCO conformance and adding address history support!** ✓
