---
description: "Enable local FastMCP development through Docker build context and volume mounts for container access"
---

# ADR-010: Enable Local FastMCP Development through Docker Build Context and Volume Mounts

## Status
Accepted

## Context

MetaMCP deployment faced two critical limitations:
- **No FastMCP capabilities**: External Docker image `ghcr.io/metatool-ai/metamcp:latest` lacked FastMCP CLI tools
- **No MCP server access**: Container isolation prevented access to host-based MCP servers in `/home/shared/projects/`
- **Limited customization**: External image dependency blocked authentication improvements (OAuth integration issues)

Error symptoms:
- `"spawn uvx ENOENT"` - FastMCP CLI missing
- `"File not found: /app/apps/backend/server.py"` - No volume access
- Unable to extend MetaMCP for organizational requirements

## Decision

### 1. Fork and Local Build Context
- Fork MetaMCP to `veloxforce/metamcp` for organizational control
- Switch docker-compose.yml from external image to local build:
  ```yaml
  # FROM: image: ghcr.io/metatool-ai/metamcp:latest  
  # TO:   build: .
  ```

### 2. FastMCP Integration
- Add FastMCP CLI installation to Dockerfile:
  ```dockerfile
  USER root
  RUN apt-get update && apt-get install -y python3-pip
  RUN uv tool install fastmcp
  USER nextjs
  ```

### 3. Volume Mount Strategy  
- Mount projects directory with read-only protection:
  ```yaml
  volumes:
    - /home/shared/projects:/projects:ro
  ```
- MCP servers accessible at `/projects/[project-name]/server.py`

## Alternatives Considered

| Alternative | Pros | Cons | Decision |
|-------------|------|------|----------|
| External image + individual server copies | Simple deployment | File duplication, sync overhead | ❌ Rejected |
| Mount only `/mcp-servers` directory | Better security isolation | Limited to single server type | ⚪ Future option |
| Full containerization (copy servers) | Pure container approach | Build complexity, no live updates | ❌ Rejected |
| **Fork + local build + volume mount** | **Full control, live updates, security** | **Fork maintenance** | ✅ **Selected** |

## Consequences

### ✅ Positive
- **Development foundation**: Full control for authentication/OAuth improvements
- **Zero file duplication**: Direct host access eliminates sync overhead  
- **Live development**: Changes reflected without container rebuilds
- **Scalable server access**: Any project can be MCP-enabled
- **Security balance**: Read-only mount prevents container file modification

### ❌ Negative  
- **Fork maintenance overhead**: Must track upstream MetaMCP updates
- **Host filesystem dependency**: Reduces pure containerization benefits
- **Broad project visibility**: Container sees entire `/home/shared/projects/`

### ⚪ Neutral
- **Security trade-off**: Functionality vs isolation - acceptable for development server
- **Build time increase**: Local compilation vs image pull - one-time cost

## Implementation Details

**Container paths mapping**:
- Host: `/home/shared/projects/mcp-servers/adr-server/server.py`  
- Container: `/projects/mcp-servers/adr-server/server.py`

**MetaMCP server configuration**:
- Command: `uvx`
- Args: `fastmcp run /projects/[project]/server.py`

## Related Decisions
- Builds on ADR-009: MetaMCP adoption for organizational MCP hosting
- Enables future authentication system improvements
- Foundation for organizational MCP server standardization

## References
- [FastMCP Documentation](https://gofastmcp.com)
- [Docker Volume Documentation](https://docs.docker.com/storage/volumes/)
- [UV Tool Management](https://docs.astral.sh/uv/tools/)