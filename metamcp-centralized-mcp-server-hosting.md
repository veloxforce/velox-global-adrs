---
description: "Centralized MCP server hosting to reduce local setup friction and unify access across environments"
---

# ADR-009: Adopt MetaMCP for Centralized MCP Server Hosting and Management

## Status

**Accepted** | Date: 2025-09-04

## Context

- Multiple MCP servers require individual local setup, dependency management, and client configuration
- AI development projects need unified access to organizational MCP server capabilities
- Local MCP server management creates friction for team members and AI agents across different environments
- No centralized platform exists for orchestrating, inspecting, and managing multiple MCP servers
- MCP client applications require separate configuration for each individual server connection

## Decision

We will adopt MetaMCP platform as our centralized MCP server hosting and management infrastructure, leveraging its four core concepts for unified MCP orchestration.

**MetaMCP Core Concepts:**

### MCP Servers
- **Purpose**: Configure and manage individual MCP server instances
- **Capability**: Support for STDIO, SSE, and Streamable HTTP server types with automatic dependency management
- **Documentation**: [MCP Servers](https://docs.metamcp.com/en/concepts/mcp-servers)

### Namespaces  
- **Purpose**: Group related MCP servers with shared middleware and access policies
- **Capability**: Enable/disable servers at namespace level and apply transformations to requests/responses
- **Documentation**: [Namespaces](https://docs.metamcp.com/en/concepts/namespaces)

### Endpoints
- **Purpose**: Create public access points for namespaces with configurable transport protocols
- **Capability**: Expose unified MCP interface via SSE, Streamable HTTP, or OpenAPI endpoints
- **Documentation**: [Endpoints](https://docs.metamcp.com/en/concepts/endpoints)

### Inspector
- **Purpose**: Debug and test MCP servers with saved configuration persistence
- **Capability**: Built-in testing interface with immediate access to configured servers and endpoints
- **Documentation**: [Inspector](https://docs.metamcp.com/en/concepts/inspector)

## Consequences

### ✅ Positive

- **Unified Management**: Single interface for configuring, monitoring, and accessing multiple MCP servers
- **Universal Client Access**: Any MCP-compatible client can access all organizational servers through standardized endpoints
- **Operational Simplicity**: Eliminates individual MCP server setup and dependency management per team member
- **Built-in Testing**: Inspector provides immediate debugging capabilities without external tooling
- **Namespace Organization**: Logical grouping of related servers with shared policies and middleware
- **Transport Flexibility**: Multiple connection methods (SSE, HTTP, OpenAPI) for diverse client requirements

### ❌ Negative

- **Infrastructure Dependency**: Requires dedicated hosting infrastructure and maintenance
- **Single Point of Management**: Centralized architecture creates potential bottleneck for MCP access
- **Platform Learning Curve**: Team must understand MetaMCP concepts beyond individual MCP server operation

### ⚪ Neutral

- **Hosting Requirements**: Needs 2GB-4GB memory instance for production deployment
- **Container Architecture**: Docker-based deployment aligns with existing infrastructure patterns

## Implementation Notes

**Deployment Pattern:**
- Docker Compose deployment with PostgreSQL backend
- Web-based configuration through dashboard interface
- Reverse proxy integration for domain-based access

**Migration Strategy:**
- Existing MCP servers can be imported as server configurations
- Gradual transition from local to centralized server access
- Namespace design should reflect organizational project structure

## References

- **MetaMCP Documentation**: [https://docs.metamcp.com/](https://docs.metamcp.com/)
- **GitHub Repository**: [https://github.com/metatool-ai/metamcp](https://github.com/metatool-ai/metamcp)
- **Core Concepts Guide**: [https://docs.metamcp.com/en/concepts](https://docs.metamcp.com/en/concepts)
- **Integration Examples**: [https://docs.metamcp.com/en/integrations](https://docs.metamcp.com/en/integrations)

## Decision Drivers

- Need for centralized MCP server management across organizational projects
- Elimination of local setup friction for AI development team members
- Built-in debugging and testing capabilities through inspector interface
- Universal MCP client compatibility without per-server configuration

---

*Last Updated: 2025-09-04*