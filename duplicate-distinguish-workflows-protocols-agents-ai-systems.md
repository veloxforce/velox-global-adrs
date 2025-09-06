---
description: "DUPLICATE: Architectural distinctions between workflows, protocols, and agents in AI system design - REVIEW NEEDED"
status: "accepted"
date: "2025-08-05"
---

# DUPLICATE: Distinguish Workflows, Protocols, and Agents in AI System Architecture

## Status
**Accepted** | Date: 2025-08-05

**⚠️ DUPLICATE NOTICE:** This ADR appears to duplicate existing content in `distinguish-workflows-protocols-agents-ai-systems.md`. Review needed to determine if there are differences or if this can be removed.

## Context
- AI development teams frequently conflate workflows, protocols, and agents, leading to architectural mismatches
- Failed attempts to "protocolize" sequential workflows waste significant development time
- Success with one approach (protocol on agent) creates false pattern matching for other use cases
- Teams struggle to determine whether to build behavioral guidelines or step-by-step instructions
- Confusion about what creates autonomous behavior vs automated execution
- No clear framework exists for choosing between architectural patterns
- Attempting to create agency through behavioral instructions alone consistently fails

## Decision
We will maintain strict architectural distinctions between workflows, protocols, and agents.

**Architectural Hierarchy:**
```
┌─────────────────────────────┐
│         AGENTS              │  ← Autonomous decision-making entities
│                             │
├─────────────────────────────┤
│        PROTOCOLS            │  ← Communication standards & interfaces  
│                             │
├─────────────────────────────┤
│        WORKFLOWS            │  ← Sequential step-by-step processes
└─────────────────────────────┘
```

**Pattern Selection Framework:**

| Use Case | Pattern | Implementation |
|----------|---------|----------------|
| **Autonomous Decisions** | Agent | Entity that evaluates context and chooses actions |
| **Communication Standards** | Protocol | Interface definitions and message formats |
| **Sequential Steps** | Workflow | Ordered process with conditional branches |

**Anti-Patterns to Avoid:**
- ❌ **Workflow as Protocol**: Don't protocol-ize sequential steps
- ❌ **Protocol as Agent**: Communication standards don't make decisions  
- ❌ **Agent as Workflow**: Don't script autonomous entities

## Consequences

### ✅ Positive
- **Clear Architecture**: Teams understand when to use each pattern
- **Reduced Waste**: Prevents failed attempts to force inappropriate patterns
- **Scalable Design**: Each pattern serves its intended purpose effectively
- **Autonomous Behavior**: Agents can actually make decisions rather than follow scripts

### ❌ Negative
- **Learning Curve**: Teams must understand three distinct architectural concepts
- **Pattern Selection**: Requires architectural judgment to choose correctly
- **Integration Complexity**: Combining patterns requires careful interface design

### ⚪ Neutral
- Existing systems may need refactoring to align with clear pattern distinctions
- Documentation must clearly specify which pattern each component implements