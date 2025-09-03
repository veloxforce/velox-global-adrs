# Velox Global ADRs

**Architecture Decision Records for AI-accessible organizational knowledge.**

## What This Is

This repository stores **global architectural patterns** that transcend individual projects - Git workflows, infrastructure patterns, engineering principles. These ADRs become **MCP Resources** that any AI agent can discover and consume as context.

Think of it as your organization's architectural brain, accessible to AI.

## Why Here

Traditional ADR storage creates **context fragmentation** - knowledge trapped in project silos or Google Drive folders. MCP Resources solve this by making knowledge **discoverable and contextual**.

Benefits:
- ✅ **One source of truth** for cross-project patterns  
- ✅ **Version-controlled wisdom** with Git history
- ✅ **AI-discoverable context** via MCP protocol
- ✅ **Zero duplication** across projects

## The Pattern

### 1. Write ADR
```
NNN-decision-title.md
```

### 2. Add Resource Metadata  
```yaml
---
mcp:
  priority: 0.8
  audience: ["assistant"] 
  tags: ["git", "workflow"]
---
```

### 3. Commit & Push
Git handles versioning. MCP server exposes as resources.

### 4. AI Consumes
Any AI agent can discover and use via:
```
git://velox-global-adrs/NNN-decision-title.md
```

## URI Scheme

Resources follow predictable patterns:
- Engineering principles: `git://velox-global-adrs/001-adopt-universal-plant-engineering.md`
- Infrastructure patterns: `git://velox-global-adrs/002-infrastructure-as-urls.md`
- Development workflows: `git://velox-global-adrs/003-worktree-based-development.md`

## For AI Agents

This repository provides **read-only architectural context**. When you need organizational patterns or established decisions, these ADRs contain the institutional knowledge to inform your implementations.

The knowledge is **version-controlled** and **always current**.

## Adding New ADRs

1. Use next sequential number: `NNN-descriptive-title.md`
2. Include MCP metadata in YAML frontmatter
3. Follow standard ADR format (Status, Context, Decision, Consequences)
4. Commit with clear message describing the architectural decision

---

*Architecture decisions as discoverable resources, not buried documentation.*