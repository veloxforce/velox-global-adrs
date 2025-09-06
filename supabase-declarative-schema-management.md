---
description: "Declarative schema management where developers define desired state, letting CLI generate migrations with tooling-enforced isolation"
status: "proposed"
date: "2025-08-31"
---

# Supabase Declarative Schema Management with Schema-Isolated Migrations

## Status

**Proposed** | Date: 2025-08-31

## Context

- **Single developer** managing multiple projects - need maximum simplicity without coordination overhead
- **Cost constraint** requires multiple projects sharing one database instance
- **Imperative migrations** scatter schema state across hundreds of files, obscuring current structure
- **Project isolation** critical - changes to one project must not affect others
- **Simplicity priority** - willing to trade control for reduced complexity
- **State visibility** needed - must see complete schema without reconstructing from migrations

## Decision

We will use declarative schema management where developers define only desired state, letting the CLI generate migrations, with tooling enforcing isolation.

**Implementation Stack:**

```
[Developer] → [/migrate] → [Schema File] → [migrate.sh] → [CLI --schema] → [Database]
     ↑                                                                           ↓
     └─────────────────── Review & Commit ←────────────────────────────────────┘
```

**Tooling Pattern:**
```bash
# Developer workflow
./migrate.sh project_name migration_description

# Generates isolation-enforced migration
supabase gen migration migration_name --schema=schema_project_name

# Applies to isolated schema
supabase db push --schema=schema_project_name
```

**Schema Isolation Strategy:**
- Each project gets dedicated schema: `schema_project_name`
- Migrations auto-prefixed with schema context
- CLI tooling prevents cross-schema interference
- Shared database instance with logical separation

## Consequences

### ✅ Positive
- **Simplified Mental Model**: Define desired state, not migration steps
- **Project Isolation**: Schema separation prevents cross-project impact
- **State Visibility**: Single file shows complete current schema structure
- **Tool-Enforced Safety**: CLI prevents accidental cross-schema modifications
- **Cost Efficiency**: Multiple projects share single database instance

### ❌ Negative
- **Migration Control Loss**: Less control over specific migration implementation
- **CLI Dependency**: Relies on Supabase CLI for migration generation
- **Schema Proliferation**: Multiple schemas in single database instance
- **Rollback Complexity**: Declarative approach may complicate rollback scenarios

### ⚪ Neutral
- Development workflow changes from imperative migration writing to declarative schema definition
- Database administration requires understanding multiple schema contexts within single instance