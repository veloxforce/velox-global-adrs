# Velox Global ADRs

**Architecture Decision Records served via MCP protocol for AI agent consumption.**

## What This Is

This repository serves as the **content backend** for MCP servers that distribute organizational architectural knowledge to AI agents. It contains global patterns, engineering principles, and infrastructure decisions that transcend individual projects.

**Architecture Flow:**
```
GitHub Repository (Source of Truth)
    ↓ (GitHub API + raw URLs)
FastMCP Server (Stateless)
    ↓ (MCP Protocol) 
Any MCP Client (Claude, ChatGPT, etc.)
    ↓
AI Agent with ADR Context
```

## System Architecture

This repository implements a **stateless MCP distribution system**:

- **Dynamic Discovery**: ADRs auto-discovered via GitHub API ([ADR-005](ADR/005-use-dynamic-discovery-for-adr-listings.md))
- **Stateless Backend**: Direct GitHub fetching, no local storage ([ADR-004](ADR/004-use-github-as-stateless-adr-backend.md))
- **MCP Distribution**: Remote server access for universal AI consumption ([ADR-003](ADR/003-use-mcp-protocol-for-adr-distribution.md))
- **FastMCP Framework**: Standardized server implementation ([ADR-002](ADR/002-adopt-fastmcp-for-mcp-server-needs.md))

## Repository Structure

```
/
├── ADR/                → Project decisions (how this system works)
├── *.md (root level)   → Global content ADRs (served to AI agents)
└── docs/work-log/      → Implementation history
```

**Global Content ADRs** (the knowledge served to AI):
- Engineering principles and patterns
- Infrastructure decisions  
- Cross-project architectural guidance

**Project ADRs** (in `/ADR/` folder):
- How this MCP distribution system operates
- Technical implementation decisions for the backend

## For AI Agents

This repository provides **networked architectural context** via MCP servers. ADRs are fetched on-demand from GitHub and served through the MCP protocol - no direct file access required.

Benefits:
- ✅ **Universal Access** - Connect once, access everywhere
- ✅ **Always Current** - Direct GitHub fetching ensures latest content  
- ✅ **Zero Local Setup** - No cloning, no filesystem dependencies
- ✅ **Version Controlled** - Full Git history and collaboration features

## Adding New ADRs

### Global Content ADRs (Organizational Knowledge)
1. Create in root: `NNN-descriptive-title.md`
2. Include YAML frontmatter with description:
   ```yaml
   ---
   description: "Brief description for MCP discovery"
   ---
   ```
3. Follow ADR format: Status, Context, Decision, Consequences
4. Commit and push - **automatically discovered** by MCP servers

### Project ADRs (About This System)
1. Create in `/ADR/`: `NNN-title.md`
2. Document decisions about MCP server operation
3. Update [CLAUDE.md](CLAUDE.md) navigation if needed

## Implementation

MCP servers implementing this backend:
- Fetch ADR lists via GitHub API
- Parse YAML frontmatter for metadata
- Serve content via raw.githubusercontent.com
- Enable AI agents to discover and consume architectural wisdom

---

*Architecture decisions as networked resources, accessible to any AI agent via MCP protocol.*