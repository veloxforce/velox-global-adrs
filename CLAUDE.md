# CLAUDE.md

This file helps Claude Code navigate the ADR repository structure.

## 🧭 Navigation Strategy
**Philosophy**: Point to the right ADR for the right context. Never duplicate content.

---

## 📚 Documentation Types

**🔵 Project ADRs**: How this MCP server system works
- Located in `/ADR/` folder
- Defines architecture, backend, and discovery mechanisms
- Read these to understand how this repository serves ADRs to AI agents

**🟢 Global Content ADRs**: Organizational knowledge served to AI agents
- Located in repository root
- Engineering principles and architectural patterns  
- These are the actual content distributed via MCP servers

---

## 🛠️ System Architecture

**How this repository works as MCP backend:**
- **MCP Framework Choice** → `ADR/002-adopt-fastmcp-for-mcp-server-needs.md`
- **Distribution Model** → `ADR/003-use-mcp-protocol-for-adr-distribution.md`
- **Backend Architecture** → `ADR/004-use-github-as-stateless-adr-backend.md`
- **Discovery Mechanism** → `ADR/005-use-dynamic-discovery-for-adr-listings.md`

**Key Insight**: This repository serves as content backend for MCP servers, not for direct consumption.

---

## 📂 Repository Structure

```
/
├── ADR/                → Project-specific decisions (how this system works)
├── *.md (root level)   → Global content ADRs (what gets served to AI agents)
└── docs/work-log/      → Historical implementation context
```

**Current Global Content ADRs:**
- `001-adopt-universal-plant-engineering.md` - Core engineering principles
- `005-use-hierarchical-field-selection-for-business-intelligence.md` - BI patterns
- `006-use-manual-data-extraction-with-period-aligned-analysis.md` - Analysis methods
- `008-distinguish-workflows-protocols-agents-in-ai-systems.md` - AI system design
- `caddy-confd-pattern-container-web-access.md` - Infrastructure patterns

---

## ➕ Adding New ADRs

**Global Content ADRs** (organizational knowledge):
1. Add to root: `NNN-descriptive-title.md`
2. Include YAML frontmatter with description
3. Push to GitHub - automatically discovered by MCP servers

**Project ADRs** (about this system):
1. Add to `/ADR/`: `NNN-title.md`  
2. Document decisions about MCP server operation
3. Update this router if navigation needs change

**ADR Structure:**
```markdown
---
description: "Brief description for MCP metadata"
---

# ADR-NNN: Title

## Status
[Proposed|Accepted|Deprecated|Superseded]

## Context, Decision, Consequences...
```

---

## 🔄 For Future Claude Instances

- This repository serves as MCP server content backend
- ADRs are fetched dynamically from GitHub (no local storage)  
- The system is stateless - everything flows from GitHub
- Never duplicate ADR content - always point to source files