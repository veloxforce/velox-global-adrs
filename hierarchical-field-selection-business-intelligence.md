---
mcp:
  priority: 0.9
  audience: ["assistant"]
  tags: ["data-engineering", "business-intelligence", "analytics"]
---

# ADR-005: Use Hierarchical Field Selection for Business Intelligence Data Model

## Status
Proposed | 2025-09-04

## Context
Business analysis requires data from multiple sources (Timing API for time tracking, Invoice CSV for revenue). With 100+ available fields across systems, analysis sessions were taking 3+ hours due to:
- No clear field selection criteria
- Entity name mismatches between systems
- Mixed units in same fields
- Analysis paralysis from too many options

## Decision
Implement a 15-field hierarchical data model with three tiers of importance and standardized entity mapping.

### Field Hierarchy

| Tier | Purpose | Timing API Fields | Invoice CSV Fields (Column Numbers) |
|------|---------|-------------------|------------------------------------|
| **Core** (7 fields) | Minimum viable analysis | • project.title<br>• duration<br>• start_date | • clientName (col 10)<br>• amount (col 83)<br>• voucherDate (col 5)<br>• dueDate (col 6) |
| **Context** (5 fields) | Pattern understanding | • notes<br>• title | • rate (col 82)<br>• quantity (col 78)<br>• status (col 38)<br>• description (col 76) |
| **Diagnostic** (3 fields) | Deep investigation | - | • paidAmount (col 71)<br>• dueAmount (col 72)<br>• headerCustomFields.Invoice Period (col 69) |

### Entity Mapping Table

| Invoice Client Name | Timing Project Name | Standard Identifier |
|-------------------|-------------------|-------------------|
| Roman Rolnik | 002-Client-IITR | IITR |
| U. Wilsch GmbH & Co. KG | 001-Project-Invoice_Agent | Invoice_Agent |
| FemRide UG | 003-Client-FemRide | FemRide |
| ARCHIBUS Solution Center Hosting Services OÜ | 006-Client-Archibus | Archibus |

**Note**: Entity relationships are discovered from data, not hardcoded. Use this mapping as a starting point, but verify client names exist in current dataset.

## Consequences

### Positive
- ✅ **85% complexity reduction**: 15 fields instead of 100+
- ✅ **Analysis speed**: 15-minute execution vs 3-hour exploration
- ✅ **Clear selection logic**: Purpose-driven field tiers
- ✅ **Consistent joins**: Entity mapping enables cross-system analysis
- ✅ **Revealed insights**: Notes field showed IITR complexity, payment timing showed cash flow reality

### Negative
- ❌ **Manual mapping maintenance**: Entity resolution table needs updates for new clients
- ❌ **Lost granularity**: 85 ignored fields may contain edge case insights

### Neutral
- ⚪ **Monthly aggregation standard**: Optimal grain for business analysis
- ⚪ **Unit disambiguation required**: Quantity field means hours OR packages OR licenses

### Why These 15 Fields

**Core Fields (Data Engineering Perspective):**
- **clientName (col 10)**: Cross-system join key enabling revenue-to-time correlation
- **amount (col 83)**: Revenue potential (not actual cash until status verified)
- **voucherDate (col 5)**: Invoice creation timestamp for period filtering
- **dueDate (col 6)**: Payment expectation date revealing cash flow timing

**Context Fields (Pattern Recognition):**
- **rate (col 82) + quantity (col 78)**: Unit validation (hourly vs fixed billing detection)
- **status (col 38)**: Payment reality check (DRAFT ≠ revenue, UNPAID ≠ overdue)
- **description (col 76)**: Work complexity indicators for resource planning

**Diagnostic Fields (Deep Investigation):**
- **paidAmount/dueAmount (cols 71,72)**: Cash flow reality vs invoice amounts
- **headerCustomFields.Invoice Period (col 69)**: Intended period (may be empty, use voucherDate fallback)

**Why 15 Fields Work:**
- 85% complexity reduction from 104 total fields
- Each tier serves specific decision-making purpose
- Hierarchy prevents analysis paralysis while maintaining investigative depth

## Implementation Notes

### Extraction Process
For detailed extraction methodology, discovery commands, and validation workflow, see **ADR-006: Use Manual Data Extraction with Period-Aligned Analysis**.

### Data Quality Requirements
1. **Period alignment**: Both data sources must cover identical time ranges
2. **Entity mapping**: Apply standardized client identifiers for cross-system joins
3. **Unit validation**: Confirm quantity field interpretation (hours vs packages vs licenses)
4. **Payment status awareness**: Distinguish between revenue potential and cash reality

## References
- Timing API Documentation: https://web.timingapp.com/docs/
- CSV Processing: Use Python pandas or csvkit for extraction

## Decision Drivers
- Session analysis showed 90% of fields were never used
- Entity mismatch caused repeated manual correlation
- Notes field analysis revealed work complexity patterns
- Payment status fields revealed cash flow vs revenue disconnect