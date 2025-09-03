---
description: "Eliminate maintenance burden through GitHub API discovery with YAML frontmatter metadata parsing"
---

# Use Dynamic Discovery for ADR Listings

## Status
**Proposed** | Date: 2025-09-03

## Context
- MCP server needs to discover available ADRs without requiring server code changes when new ADRs are added
- Hardcoded ADR lists create maintenance burden - every new ADR requires server code updates
- AI agents need descriptive context to select relevant ADRs without fetching all content
- Content creators shouldn't need programming knowledge to add new architectural decisions
- The "Infrastructure as URLs" vision requires content to appear automatically when published

## Decision
We will fetch ADR lists dynamically from GitHub API and parse YAML frontmatter for descriptions, eliminating all hardcoded ADR maintenance in the server.

**Implementation Architecture:**
```
GitHub API (/repos/{owner}/{repo}/contents)
    ↓ (List .md files)
Filter (exclude README.md)
    ↓ (For each ADR)
Fetch raw content from GitHub
    ↓ (Parse YAML frontmatter)
Extract description metadata
    ↓ (Return enriched listing)
AI sees: {"filename": "...", "description": "..."}
```

**Key Design Choices:**
- Dynamic file discovery via GitHub API
- YAML frontmatter parsing with python-frontmatter
- Description-only metadata (YAGNI principle)
- Filter README.md from ADR listings

## Consequences

### ✅ Positive
- **Zero Maintenance Burden** - Add ADR to GitHub → automatically appears in listings
- **Intelligent AI Selection** - Descriptions enable informed resource selection without blind fetching
- **Content Creator Friendly** - No server code changes needed to add ADRs
- **True Single Source** - GitHub repository is the only place to manage ADRs
- **Consistent with Stateless Architecture** - No cached lists to invalidate

### ❌ Negative
- **Performance Cost** - N+1 API calls (list files + fetch each for metadata)
- **GitHub API Rate Limits** - Each listing consumes multiple API calls from 60/hour quota
- **Network Latency** - Multiple round-trips for complete listing (~500ms-1s total)

### ⚪ Neutral
- Trade-off accepted: slower listing for zero maintenance
- Metadata parsing happens on every list request
- Performance acceptable for ADR access patterns (infrequent operations)

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Hardcoded KNOWN_ADRS Array** | Requires server code updates for every new ADR; maintenance burden |
| **Static resources.json Metadata** | Duplicates information; drift between content and metadata |
| **Cache Parsed Metadata** | Adds cache invalidation complexity; violates stateless principle |
| **Complex Metadata (tags, priority)** | YAGNI - description sufficient for AI selection |
| **Include All .md Files** | README.md not an ADR; would pollute listings |

## Implementation Notes

**Evolution Journey:**
1. Started with hardcoded `KNOWN_ADRS = ["001-...", "002-..."]`
2. Realized maintenance burden with every ADR addition
3. Implemented GitHub API listing for dynamic discovery
4. Added YAML frontmatter parsing for descriptions
5. Simplified to description-only metadata (removed tags, priority)

**Key Insight:** The performance cost (N+1 API calls) is acceptable because it completely eliminates maintenance burden. For ADR use cases, maintenance elimination > performance optimization.

**Dependencies:**
- `python-frontmatter` for YAML parsing
- GitHub API for file listing
- `requests` library for HTTP fetching