---
description: "External shared networks for selective cross-Compose service communication with environment isolation"
---

# DEVELOPMENT-002: Use External Shared Networks for Cross-Compose Service Communication

## Status

**Accepted** | Date: 2025-07-26

## Context

- Multi-environment applications require network isolation between development, staging, and production workloads
- Shared infrastructure services (monitoring, logging, databases) need accessibility from multiple isolated environments
- Container orchestration spans multiple Docker Compose projects for organizational separation
- Service discovery preferred over IP address management for container communication
- Network security requires selective communication rather than full connectivity
- Infrastructure services deployed independently from application environments
- Common pattern needed across microservice architectures with shared operational tooling

## Decision

We will use external shared networks to enable selective communication between isolated Docker Compose projects while maintaining environment boundaries.

**Network Topology Pattern:**

```
Environment A        Shared Infrastructure      Environment B
┌─────────────┐      ┌─────────────────────┐      ┌─────────────┐
│   App Net   │      │  Infrastructure Net │      │   App Net   │
│             │      │    (External)       │      │             │
│ ┌─────────┐ │      │ ┌─────────────────┐ │      │ ┌─────────┐ │
│ │Service A├─┼──────┼─┤Infrastructure   ├─┼──────┼─┤Service B│ │
│ └─────────┘ │      │ │Service          │ │      │ └─────────┘ │
│             │      │ └─────────────────┘ │      │             │
└─────────────┘      └─────────────────────┘      └─────────────┘

```

**Implementation Architecture:**

1. **Create Domain-Specific Networks**: External networks for communication domains (monitoring, logging, data)
2. **Multi-Network Service Membership**: Services join isolated network + required shared networks
3. **Service Discovery**: Reference services by DNS name across shared networks
4. **Selective Access Control**: Only services requiring communication join shared networks

**Configuration Pattern:**

```yaml
# Shared Infrastructure Service
services:
  infrastructure-service:
    networks:
      - infrastructure-default    # Service's own network
      - monitoring-shared         # Cross-environment access

networks:
  infrastructure-default:
    driver: bridge
  monitoring-shared:
    external: true

# Application Environment Service
services:
  application-service:
    networks:
      - application-isolated      # Environment isolation
      - monitoring-shared         # Infrastructure access

networks:
  application-isolated:
    driver: bridge
  monitoring-shared:
    external: true

```

**Completion Criteria:**

- External networks created for each communication domain
- Infrastructure services accessible via DNS from authorized environments
- Environment isolation verified for application-to-application traffic
- Service discovery functional across shared network boundaries

**Implementation Specifics:**

- External network creation: `docker network create <domain>-network`
- Network lifecycle managed outside individual compose projects
- DNS resolution works bidirectionally across shared networks
- Container restart required when adding networks to existing services

## Consequences

### ✅ Positive

- **Environment Isolation Maintained**: Application environments cannot communicate directly
- **Service Discovery Enabled**: DNS-based communication eliminates IP address management
- **Infrastructure Independence**: Shared services deploy independently from applications
- **Granular Access Control**: Networks provide communication boundaries
- **Docker Best Practice Compliance**: Follows documented patterns for multi-compose communication
- **Operational Flexibility**: Infrastructure updates don't impact environment isolation

### ❌ Negative

- **Network Configuration Complexity**: Multiple networks require coordination across compose files
- **Infrastructure Prerequisites**: External networks must exist before dependent services start
- **Cross-Project Dependencies**: Network naming conventions require organizational coordination
- **Troubleshooting Complexity**: Multiple network paths complicate connectivity debugging

### ⚪ Neutral

- **Network Lifecycle Management**: External networks require separate creation and cleanup processes
- **Service Configuration Updates**: Adding network access requires container recreation
- **Documentation Requirements**: Network topology and access patterns need clear documentation

## Alternatives Considered

| Alternative | Rejection Reason |
| --- | --- |
| **Host Network Mode** | Eliminates container network isolation and security boundaries |
| **Static IP Assignment** | Requires IP address coordination and introduces configuration brittleness |
| **Single Shared Network** | Breaks environment isolation by allowing direct cross-environment communication |
| **Port Publishing to Host** | Creates port conflicts and doesn't enable direct service-to-service communication |
| **Service Mesh** | Over-engineered solution for simple cross-compose communication needs |