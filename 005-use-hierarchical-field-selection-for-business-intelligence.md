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

## Field Discovery First
Before any field extraction, agents must discover the actual data structure to avoid assumptions:

```python
# ALWAYS run this discovery command first
import csv
with open('Invoice_Data_Refrens_*.csv', 'r') as f:
    fieldnames = list(csv.DictReader(f).fieldnames)
    print(f'Total CSV fields: {len(fieldnames)}')
    # Expected: 104 fields
    # Use field positions, not field names in analysis
```

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

## Critical Calculations
These calculations must be implemented for complete business analysis:

### Payment Terms Analysis
```python
# Calculate payment terms (days between invoice and due date)
from datetime import datetime

vouch_date = datetime.strptime(row['voucherDate'], '%d-%m-%Y')
due_date = datetime.strptime(row['dueDate'], '%d-%m-%Y')
payment_terms = (due_date - vouch_date).days
```

### Overdue Status Check
```python
# Determine if invoice is overdue vs within terms
today = datetime.now()
if row['status'] != 'PAID' and due_date < today:
    days_overdue = (today - due_date).days
    status = f'OVERDUE by {days_overdue} days'
else:
    days_until_due = (due_date - today).days
    status = f'Due in {days_until_due} days'
```

### Date-Based Filtering (Primary Method)
```python
# Use voucherDate for period filtering, not period field
if '-07-2025' in row['voucherDate'] or '-08-2025' in row['voucherDate']:
    # Process July-August invoices
    # Period field (col 69) may be empty - don't rely on it
```

## Implementation Notes

### Quick Extraction Commands
```bash
# Timing API - Include project data to avoid second call
GET /api/v1/time-entries?start_date_min=2025-07-01&include_project_data=1

# Invoice CSV - Use column numbers for reliability
python -c "import csv; print(list(csv.DictReader(open('file.csv')).fieldnames))"
# Then extract specific columns by name, not position
```

### Critical Validations
**Step 0: Field Discovery** - Always run field discovery command before extraction
1. **Date filtering**: Use voucherDate (col 5) not period field (col 69) - period may be empty
2. **Period alignment**: Filter both Timing and Invoice to identical date ranges
3. **Payment terms**: Calculate from voucherDate to dueDate - reveals cash flow timing
4. **Overdue analysis**: Check dueDate against today, not just UNPAID status
5. **Unit checking**: Verify if quantity (col 78) represents hours (rate × quantity ≈ amount)

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