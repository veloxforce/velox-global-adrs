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

## Decision
Use manual data extraction with strict period alignment and monthly aggregation as the standard analysis approach.

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
   
   # Invoice: Manual export then filter
   csvcut -c clientName,amount,period,status invoice_export.csv | \
     csvgrep -c period -m "July 2025" -m "August 2025"
   ```

2. **Transform Phase**
   - Filter both sources to same period FIRST
   - Apply entity mapping (see ADR-005)
   - Separate hourly from fixed billing items
   - Check payment status for cash flow reality

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

### Validation Checklist
- [ ] Both data sources filtered to identical date range?
- [ ] Entity names mapped to standard identifiers?
- [ ] Units verified (hours vs packages)?
- [ ] Payment status checked (DRAFT vs PAID)?
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
- **Entity Mapping**: See ADR-005 for client name standardization

## Decision Drivers
- Invoice system lacks API access (vendor limitation)
- Monthly patterns emerged as natural business rhythm
- Payment timing analysis revealed cash flow != revenue
- 3-hour exploration reduced to 15-minute execution with proper sequence

---

*Last Updated: 2025-09-03*  
*Next Review: When invoice API becomes available*