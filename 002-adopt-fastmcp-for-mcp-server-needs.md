---
mcp:
  priority: 0.8
  audience: ["assistant"]
  tags: ["mcp", "framework", "tooling", "servers"]
---

# 002: adopt fastmcp for mcp server needs

## Status
**Accepted** | Date: 2025-01-03

## Context
- Need to create MCP servers that expose organizational resources (ADRs, documentation, tools)
- MCP protocol implementation involves significant boilerplate (server setup, protocol handlers, error management)
- Team requires rapid prototyping capability for MCP resource exposure
- Python is the primary scripting language in the organization
- Direct protocol implementation would duplicate effort across every server
- Documentation and resources need to be accessible to AI agents via standard protocol

## Decision
We will use FastMCP as the standard framework for all MCP server implementations.

**Core Pattern:**
```python
from fastmcp import FastMCP

mcp = FastMCP("Server Name")

@mcp.resource("scheme://{parameter}")
def resource_function(parameter: str) -> str:
    """Resource documentation"""
    return content

@mcp.tool
def tool_function(arg: type) -> result:
    """Tool documentation"""
    return result
```

**Reference Documentation:** [gofastmcp.com](https://gofastmcp.com) (avoid duplication)

**Key Resources:**
- [Installation & Quickstart](https://gofastmcp.com/getting-started/quickstart)
- [Resources & Templates](https://gofastmcp.com/servers/resources)
- [GitHub Repository](https://github.com/jlowin/fastmcp)

**Completion Criteria:** 
- All MCP servers use FastMCP framework
- Resources exposed via decorator pattern
- Servers runnable with `fastmcp run server.py`

## Consequences

### ✅ Positive
- 10-line minimum viable servers with decorator syntax
- Automatic JSON-RPC protocol handling without manual implementation
- Built-in transport options (STDIO, HTTP, SSE) without extra code
- Type hints automatically generate validation schemas

### ❌ Negative
- Python-only solution limits language diversity
- Dependency on external framework (pip install fastmcp)
- Framework abstractions may hide protocol details from developers

### ⚪ Neutral
- Requires pip or uv for dependency management
- Documentation at gofastmcp.com becomes critical reference
- Decorator pattern becomes standard across all implementations

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Direct MCP SDK** | Requires extensive boilerplate for each server implementation |
| **TypeScript FastMCP** | Team has stronger Python expertise and existing toolchain |
| **Custom Framework** | Would duplicate FastMCP's proven functionality |