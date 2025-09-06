# 002: Expose Custom Database Schemas to Supabase REST API

## Status
**Accepted** | Date: 2025-01-04

## Context
- Custom database schemas (non-`public`) are not automatically exposed to Supabase REST API
- Applications using custom schemas encounter 404 errors when accessing tables via Supabase client
- Default Supabase setup only exposes `public` schema to REST/GraphQL endpoints
- Custom schemas require explicit configuration for API accessibility
- Security model relies on RLS policies rather than restrictive database permissions
- Development teams need consistent process for schema exposure to prevent recurring issues

**Error Pattern:**
```
POST https://project.supabase.co/rest/v1/custom_schema.table_name 404 (Not Found)
```

## Decision
We will establish a standardized process for exposing custom database schemas to the Supabase REST API.

**Required Steps for Any Custom Schema:**

### Step 1: Dashboard Configuration
```
Supabase Dashboard → API Settings → Data API Settings
→ Add schema name to "Exposed schemas" list
→ Save configuration
```

### Step 2: Database Permissions (SQL)
```sql
-- Grant schema usage to API roles
GRANT USAGE ON SCHEMA {schema_name} TO anon, authenticated, service_role;

-- Grant table permissions matching Supabase defaults
GRANT ALL ON ALL TABLES IN SCHEMA {schema_name} TO anon, authenticated, service_role;

-- Set default privileges for future objects
ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA {schema_name} 
GRANT ALL ON TABLES TO anon, authenticated, service_role;

ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA {schema_name} 
GRANT ALL ON ROUTINES TO anon, authenticated, service_role;

ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA {schema_name} 
GRANT ALL ON SEQUENCES TO anon, authenticated, service_role;
```

**Completion Criteria:**
- Schema appears in "Exposed schemas" list in Supabase Dashboard
- Database permissions granted to all API roles
- Client can successfully query custom schema tables
- No 404 errors when accessing schema via REST API

## Consequences

### ✅ Positive
- Custom schemas become accessible via Supabase client libraries
- Consistent security model across all schemas (RLS-based)
- Eliminates recurring 404 errors during development
- Maintains Supabase's intended permission structure

### ❌ Negative
- Additional configuration step required for each custom schema
- Broad permissions granted to all roles (security relies on RLS)
- Manual process prone to being forgotten during schema creation

### ⚪ Neutral
- Permission model matches Supabase defaults for `public` schema
- RLS policies remain primary security mechanism
- No impact on existing `public` schema functionality

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Use only `public` schema** | Limits organizational flexibility and namespace management |
| **Restrictive permissions** | Conflicts with Supabase's default security model and RLS approach |
| **Direct SQL queries only** | Bypasses Supabase client benefits and real-time features |

## Implementation Checklist

When creating any new custom schema:

- [ ] Create schema: `CREATE SCHEMA schema_name;`
- [ ] Add to Supabase Dashboard "Exposed schemas"
- [ ] Execute permission grants (SQL above)
- [ ] Test API access with Supabase client
- [ ] Verify no 404 errors in application
- [ ] Document schema purpose and structure

## Validation Commands

```sql
-- Verify schema is accessible
SELECT schema_name FROM information_schema.schemata 
WHERE schema_name = 'your_schema';

-- Check permissions are granted
SELECT grantee, table_schema, privilege_type
FROM information_schema.table_privileges 
WHERE table_schema = 'your_schema'
AND grantee IN ('anon', 'authenticated', 'service_role');
```

```javascript
// Test client access
const { data, error } = await supabase
  .from('your_schema.table_name')
  .select('*')
  .limit(1);
```
