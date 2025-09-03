---
mcp:
  priority: 0.9
  audience: ["assistant"]
  tags: ["data-engineering", "business-intelligence", "analytics"]
---

# ADR-005: Use Hierarchical Field Selection for Business Intelligence Data Model

## Status
Accepted

## Context
Business analysis requires data from multiple sources (Timing API for time tracking, Invoice CSV for revenue). With 100+ available fields across systems, analysis sessions were taking 3+ hours due to:
- No clear field selection criteria
- Entity name mismatches between systems
- Mixed units in same fields
- Analysis paralysis from too many options

## Decision
Implement a 15-field hierarchical data model with three tiers of importance and standardized entity mapping.

### Field Hierarchy

| Tier | Purpose | Timing API Fields | Invoice CSV Fields |
|------|---------|-------------------|-------------------|
| **Core** (6 fields) | Minimum viable analysis | • project.title<br>• duration<br>• start_date | • clientName<br>• amount<br>• headerCustomFields.Invoice Period |
| **Context** (5 fields) | Pattern understanding | • notes<br>• title | • rate<br>• quantity<br>• status |
| **Diagnostic** (4 fields) | Deep investigation | - | • dueDate<br>• paidAmount<br>• dueAmount<br>• lineItem |

### Entity Mapping Table

| Invoice Client Name | Timing Project Name | Standard Identifier |
|-------------------|-------------------|-------------------|
| Roman Rolnik | 002-Client-IITR | IITR |
| U. Wilsch GmbH & Co. KG | 001-Project-Invoice_Agent | Invoice_Agent |
| FemRide UG | 003-Client-FemRide | FemRide |
| ARCHIBUS Solution Center | 006-Client-Archibus | Archibus |

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

## Implementation Notes

### Quick Extraction Commands
```bash
# Timing API - Include project data to avoid second call
GET /api/v1/time-entries?start_date_min=2025-07-01&include_project_data=1

# Invoice CSV - Extract only needed columns
csvcut -c clientName,amount,quantity,rate,status,"headerCustomFields.Invoice Period"
```

### Critical Validations
1. **Period alignment**: Always filter both sources to same date range
2. **Unit checking**: Verify if quantity represents hours (rate × quantity ≈ amount)
3. **Payment reality**: Check status and dueAmount for actual cash flow

## References
- Timing API Documentation: https://web.timingapp.com/docs/
- CSV Processing: Use Python pandas or csvkit for extraction

## Decision Drivers
- Session analysis showed 90% of fields were never used
- Entity mismatch caused repeated manual correlation
- Notes field analysis revealed work complexity patterns
- Payment status fields revealed cash flow vs revenue disconnect

---

*Last Updated: 2025-09-03*