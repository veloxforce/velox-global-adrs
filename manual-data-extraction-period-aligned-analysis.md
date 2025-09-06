---
mcp:
  priority: 0.9
  audience: ["assistant"]
  tags: ["data-architecture", "business-intelligence", "timing-api", "analytics"]
---

# ADR-006: Use Manual Data Extraction with Period-Aligned Analysis

## Status
Accepted

## Context
Business intelligence analysis requires combining time tracking data (Timing API) with revenue data (Invoice system). Initial analysis sessions revealed critical issues:
- Period mismatches led to dividing 9 months of revenue by 2 months of time
- Mixed units in quantity fields (hours vs packages vs licenses)
- No automated invoice API access available
- Entity names don't match across systems

## Field Discovery First
Before any field extraction, agents must discover the actual data structure to avoid assumptions:

```python
# ALWAYS run this discovery command first
import csv
with open('Invoice_Data_Refrens_*.csv', 'r') as f:
    fieldnames = list(csv.DictReader(f).fieldnames)
    print(f'Total CSV fields: {len(fieldnames)}')
    # Expected: 104 fields
    # Use exact field names from discovery, not assumptions
```

## Decision
Use manual data extraction with strict period alignment and monthly aggregation as the standard analysis approach.

### Why Manual Extraction (Strategic Choice)

**Pattern Discovery Phase:**
- Still learning which fields matter for different business decisions
- Client relationship dynamics evolving (FemRide negotiations, IITR analysis patterns)
- Payment term variations requiring flexible investigation

**Avoiding Premature Optimization:**
- No script maintenance overhead during exploration
- Quick adaptation to new clients or data structure changes
- Easy debugging when data patterns look unexpected
- Immediate pivot capability for new analysis requirements

**Future Automation Path:**
Manual Discovery → Validated Patterns → Reusable Scripts → Full Automation
*(We're intentionally at step 1 to build solid foundations)*

### Extraction Strategy

| Data Source | Access Method | Extraction Approach | Key Parameters |
|------------|---------------|-------------------|----------------|
| **Timing API** | REST API with Bearer token | Direct API calls | • `start_date_min`/`max`<br>• `include_project_data=1`<br>• JSON response |
| **Invoice System** | Manual CSV export | Download full export, filter locally | • 90+ columns available<br>• Extract 10-12 relevant<br>• Filter by period field |

### Processing Sequence

1. **Extract Phase**
   ```bash
   # Timing: Direct API call
   curl -H "Authorization: Bearer $TOKEN" \
     "https://web.timingapp.com/api/v1/time-entries?start_date_min=2025-07-01&start_date_max=2025-08-31&include_project_data=1"
   
   # Invoice: Manual export then filter using voucherDate (more reliable)
   # Use voucherDate for filtering - period field may be empty
   python -c "
   import csv
   with open('invoice_export.csv') as f:
       reader = csv.DictReader(f)
       for row in reader:
           if '-07-2025' in row.get('voucherDate', '') or '-08-2025' in row.get('voucherDate', ''):
               print(','.join([row.get('clientName', ''), row.get('amount', ''), row.get('status', ''), row.get('voucherDate', '')]))
   "
   ```

2. **Transform Phase**
   - Filter both sources to same period FIRST
   - Apply entity mapping (see ADR-005 for field significance)
   - Calculate payment terms: (dueDate - voucherDate).days
   - Check overdue status: compare dueDate to today's date
   - Separate hourly from fixed billing items using quantity field validation

3. **Analyze Phase**
   - Aggregate to monthly grain
   - Calculate metrics only on aligned data
   - Validate units before calculations

## Consequences

### Positive
- ✅ **Timing API simplicity**: Well-structured REST API with good documentation
- ✅ **Period alignment enforcement**: Prevents invalid comparisons
- ✅ **15-minute execution**: When following the defined sequence
- ✅ **Payment visibility**: CSV includes status, dueDate, paidAmount fields

### Negative
- ❌ **Manual invoice export**: No API means manual download step
- ❌ **Entity mapping overhead**: Must apply mapping table each time
- ❌ **No real-time data**: Invoice data is point-in-time export

### Neutral
- ⚪ **Monthly aggregation standard**: Loses daily detail but matches business cycles
- ⚪ **Python/bash scripting**: Simple scripts sufficient, no complex tooling needed

## Critical Lessons Learned

### What Went Wrong
- **Period mismatch**: Compared Jan-Sep revenue with Jul-Aug time (900% error)
- **Unit confusion**: Treated package quantities as hours
- **Payment assumptions**: Counted unpaid invoices as revenue

### Critical Calculations
These calculations must be implemented for complete business analysis:

```python
# Payment Terms Analysis
from datetime import datetime

vouch_date = datetime.strptime(row['voucherDate'], '%d-%m-%Y')
due_date = datetime.strptime(row['dueDate'], '%d-%m-%Y')
payment_terms = (due_date - vouch_date).days

# Overdue Status Check
today = datetime.now()
if row['status'] != 'PAID' and due_date < today:
    days_overdue = (today - due_date).days
    status = f'OVERDUE by {days_overdue} days'
else:
    days_until_due = (due_date - today).days
    status = f'Due in {days_until_due} days'
```

### Validation Checklist
**Step 0: Field Discovery** - Always run field discovery command before extraction
- [ ] Both data sources filtered to identical date range using voucherDate?
- [ ] Entity names mapped to standard identifiers (see ADR-005)?
- [ ] Payment terms calculated from voucherDate to dueDate?
- [ ] Overdue status checked against today's date, not just UNPAID?
- [ ] Units verified (hours vs packages) using rate × quantity ≈ amount?
- [ ] Monthly aggregation applied consistently?

## Implementation Notes

### Common Pitfalls to Avoid
1. **Never compare different periods** - Filter first, analyze second
2. **Check invoice status** - DRAFT ≠ revenue
3. **Verify quantity units** - Not everything is hours
4. **Include `include_project_data=1`** - Avoids second API call

### Quick Reference
- **Timing API Docs**: https://web.timingapp.com/docs/
- **Bearer Token Location**: `.env` file as `TIMING_API_TOKEN`
- **Field Selection Logic**: See ADR-005 for hierarchical field significance
- **Date Filtering**: Use voucherDate (col 5) when period field (col 69) is empty

## Decision Drivers
- Invoice system lacks API access (vendor limitation)
- Monthly patterns emerged as natural business rhythm
- Payment timing analysis revealed cash flow != revenue
- 3-hour exploration reduced to 15-minute execution with proper sequence

---

*Last Updated: 2025-09-03*  
*Next Review: When invoice API becomes available*