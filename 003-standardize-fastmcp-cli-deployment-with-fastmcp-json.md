---
description: "Standardized deployment pattern for FastMCP servers using CLI and centralized configuration"
---

# 003: standardize fastmcp cli deployment with fastmcp.json configuration

## Status
**Accepted** | Date: 2025-09-05

## Context
- ADR-002 established FastMCP as framework standard but left deployment patterns undefined
- Direct Python execution (`python server.py`) requires manual dependency and environment management
- MetaMCP deployment (ADR-010) requires STDIO transport and specific execution patterns
- Environment variables scattered across .env files, shell exports, and documentation
- pyproject.toml dependency management creates additional configuration layer
- Team needs consistent deployment pattern across local development and production environments

## Decision
We will standardize all FastMCP server deployments using FastMCP CLI with fastmcp.json configuration.

**Core Configuration Pattern:**
```json
{
  "$schema": "https://gofastmcp.com/public/schemas/fastmcp.json/v1.json",
  "source": {
    "path": "server.py",
    "entrypoint": "mcp"
  },
  "environment": {
    "dependencies": ["fastmcp", "requests"] # Always just without version numbers
  },
  "env": {
    "API_KEY": "${API_KEY}",
    "API_URL": "${API_URL}"
  }
}
```

**Execution Pattern:**
```bash
# Via uvx (no global install) - recommended
uvx fastmcp run /path/to/fastmcp.json  # Production deployment
uvx fastmcp run /path/to/server.py     # Direct execution

# Via global FastMCP CLI
fastmcp run /path/to/fastmcp.json     # With configuration
fastmcp run /path/to/server.py        # Direct execution
```

**Container Deployment Patterns:**

For Docker/MetaMCP environments, **always use absolute paths** in fastmcp.json to avoid working directory issues:

```json
{
  "$schema": "https://gofastmcp.com/public/schemas/fastmcp.json/v1.json",
  "source": {
    "path": "/projects/mcp-servers/server-name/server.py",  // Absolute path required
    "entrypoint": "mcp"
  },
  "environment": {
    "dependencies": ["requests", "python-frontmatter"]
  }
}
```

```bash
# Container execution - references fastmcp.json with absolute paths
uvx fastmcp run /projects/mcp-servers/server-name/fastmcp.json

# Local development - relative paths work fine
uvx fastmcp run ./fastmcp.json
uvx fastmcp run ./server.py
```

**Path Resolution Rules:**
- **Production/Container:** Use absolute paths in `fastmcp.json` "source.path"
- **Local Development:** Relative paths acceptable for direct CLI execution
- **Why:** Container working directories differ from file locations, causing path resolution failures

**Environment Variable Extrapolation:**
- Verified working: `${VAR}` syntax in fastmcp.json reads from .env files
- Tested with FastMCP 2.12.2: environment variables successfully resolved at runtime
- Pattern: `.env` → `fastmcp.json` with `${VAR}` → server receives resolved values

**Reference Documentation:**
- [FastMCP JSON Schema](https://gofastmcp.com/public/schemas/fastmcp.json/v1.json)
- [FastMCP CLI Commands](https://gofastmcp.com/deployment/server-configuration)
- [Server Configuration Guide](https://gofastmcp.com/deployment/server-configuration)
- [Running Servers Documentation](https://gofastmcp.com/deployment/running-server)

**Completion Criteria:**
- All MCP servers include fastmcp.json configuration with proper `source` object structure
- Environment variables use `${VAR}` extrapolation pattern
- Servers run via `uvx fastmcp run /path/to/fastmcp.json` for production
- Container deployments use absolute paths in fastmcp.json "source.path"
- pyproject.toml eliminated for MCP server projects

## Consequences

### ✅ Positive
- **Single configuration source**: Dependencies, environment, and entrypoint in one file
- **Environment extrapolation**: `${VAR}` syntax enables .env file integration (verified working)
- **Zero Python setup**: uvx execution eliminates virtual environment management
- **Schema validation**: JSON schema ensures configuration correctness

### ❌ Negative
- **CLI dependency**: Requires FastMCP CLI installation (uvx mitigates globally)
- **Migration effort**: Existing servers need fastmcp.json creation
- **Pattern change**: Developers must adapt from `python server.py` to `fastmcp run`

### ⚪ Neutral
- Replaces pyproject.toml for dependency management in MCP projects
- .env files remain for sensitive credential storage

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Direct Python execution** | Requires manual dependency installation and environment setup |
| **Docker containers** | Adds complexity without solving configuration centralization |
| **pyproject.toml + uv** | Two configuration files vs one; no environment extrapolation |
| **Shell scripts** | Platform-specific; doesn't standardize configuration format |

## Implementation Notes

**Migration from pyproject.toml:**
```python
# From pyproject.toml dependencies
dependencies = [
    "fastmcp>=2.8.0",
    "python-dotenv>=1.1.0",  # No longer needed
    "requests>=2.32.4"
]

# To fastmcp.json environment.dependencies
"dependencies": ["fastmcp", "requests"]
```

**Transport Configuration:**
```python
# For MetaMCP deployment (STDIO)
mcp.run()  # Default STDIO transport

# Previous HTTP transport (deprecated)
mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)
```

**Troubleshooting Common Issues:**

| Error Pattern | Root Cause | Solution |
|---------------|------------|----------|
| `source Field required [type=missing]` | Missing `source` object in fastmcp.json | Add proper `source: {"path": "...", "entrypoint": "mcp"}` structure |
| `File not found: /app/apps/backend/server.py` | Path resolution in wrong directory | Use absolute paths in container fastmcp.json |
| `spawn fastmcp EACCES` | Permission/PATH issues in container | Use `uvx fastmcp run` instead of direct `fastmcp` |
| `No module named 'frontmatter'` | Dependencies not installed from config | Ensure `environment.dependencies` array is properly configured |

## References

- **Builds on**: ADR-002 (FastMCP framework adoption)
- **Enables**: ADR-010 (MetaMCP deployment requirements)
- **Testing evidence**: 
  - Bruce BEM server migration (2025-09-04) proved environment extrapolation
  - Velox ADR Server deployment (2025-09-05) confirmed container path resolution requirements
- **Schema reference**: https://gofastmcp.com/public/schemas/fastmcp.json/v1.json
- **CLI version tested**: FastMCP 2.12.2 with MCP SDK 1.13.1

---

*Last Updated: 2025-09-05*