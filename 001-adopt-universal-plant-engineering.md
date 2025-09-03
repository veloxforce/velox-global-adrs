---
mcp:
  priority: 0.9
  audience: ["assistant"]
  tags: ["engineering", "principles", "systems"]
---

# ADR-001: Adopt Universal Plant Engineering Principle

## Status
Accepted

## Context
Traditional software development follows construction metaphors - building systems from blueprints with fixed requirements. This leads to brittle architectures that resist change and require extensive planning before implementation.

Biological systems demonstrate superior adaptability - they grow organically, self-heal, prune unused components, and evolve based on environmental pressures.

## Decision
Apply biological growth patterns to system architecture and development processes.

### Core Principles
1. **Organic Growth** - Systems expand based on actual usage patterns, not predicted requirements
2. **Self-Healing** - Components automatically recover from failures and adapt to problems
3. **Natural Pruning** - Dead or unused code is identified and removed automatically
4. **Environmental Adaptation** - Architecture evolves in response to real-world constraints

## Implementation Guidelines

### Development Process
- Start with minimal viable systems
- Allow natural feature evolution based on user needs
- Implement monitoring that reveals system health and usage patterns
- Build auto-scaling and self-recovery into core components

### Architecture Decisions
- Favor composable, loosely-coupled components
- Implement circuit breakers and graceful degradation
- Use feature flags for natural experimentation
- Design for observability and introspection

## Consequences

### Positive
- Systems become more resilient and adaptable
- Development effort focuses on genuinely needed features
- Reduced over-engineering and premature optimization
- Natural alignment between system capabilities and real requirements

### Negative
- Requires cultural shift from "plan everything" to "grow intelligently"
- Initial uncertainty about final system shape
- Need for robust monitoring and observability infrastructure

### Mitigation Strategies
- Invest heavily in observability tooling
- Establish clear pruning criteria and automated cleanup processes
- Document growth patterns and environmental pressures
- Regular architecture reviews to guide healthy evolution

## Related Decisions
- Infrastructure as URLs (enables organic tool growth)
- Self-healing development environments
- Autonomous AI agent patterns