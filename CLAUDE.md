# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository stores **global Architecture Decision Records (ADRs)** that become **MCP Resources** - discoverable knowledge for AI agents across organizational projects. Think of it as an architectural brain accessible to AI via the MCP protocol.

## ADR Format & Structure

### File Naming
- Sequential numbering: `001-descriptive-title.md`, `002-next-decision.md`
- Use hyphens, keep titles concise but descriptive

### Required Structure
```markdown
---
description: "Brief description for MCP metadata"
---

# ADR-NNN: Title

## Status
[Proposed|Accepted|Deprecated|Superseded]

## Context
[Why this decision is needed]

## Decision
[What we decided to do]

## Consequences
[Positive and negative outcomes]
```

## MCP Resource System

### Resource Registration
ADRs are automatically exposed via MCP through `.mcp/resources.json` which maps ADRs to discoverable resources with:
- URI scheme: `git://velox-global-adrs/NNN-title.md`
- Priority levels (0.0-1.0)
- Audience tags (typically `["assistant"]`)
- Searchable tags

### Resource Metadata
Include YAML frontmatter in ADRs for MCP discovery:
```yaml
---
description: "Brief description for resource catalogs"
mcp:
  priority: 0.8
  audience: ["assistant"] 
  tags: ["relevant", "keywords"]
---
```

## Common Workflows

### Adding New ADRs
1. Find next sequential number by checking existing files
2. Create `NNN-descriptive-title.md` following ADR structure
3. Include MCP metadata in frontmatter
4. Update `.mcp/resources.json` if implementing MCP server
5. Commit with clear message describing the architectural decision

### Git Workflow
- This is version-controlled wisdom - commit history shows decision evolution
- Standard Git workflow: branch, commit, push, PR if collaborative
- Keep commits focused on single decisions

## Architecture Notes

- **Read-only knowledge base**: AI agents consume this as context, don't modify during tasks
- **Cross-project patterns**: Decisions here transcend individual codebases
- **MCP Integration**: Designed for AI discovery via Model Context Protocol
- **Version control**: Git provides decision history and ensures current state

## Key Principles from ADR-001
The repository follows "Universal Plant Engineering" - organic growth based on actual needs rather than predicted requirements. ADRs should emerge from real architectural decisions, not theoretical planning.