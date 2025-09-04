# 008: Distinguish Workflows, Protocols, and Agents in AI System Architecture

## Status
**Accepted** | Date: 2025-08-05

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
│   PROTOCOL LAYER            │ ← Behavioral patterns (WHEN/WHY)
├─────────────────────────────┤
│   AGENT LAYER               │ ← Autonomous system (DECIDES)
├─────────────────────────────┤
│   WORKFLOW LAYER            │ ← Sequential steps (HOW)
├─────────────────────────────┤
│   TOOL LAYER               │ ← Capabilities (WHAT)
└─────────────────────────────┘
```

**Recognition Framework:**

| Pattern | Definition | Key Phrase | Example |
|---------|------------|------------|---------|
| **Workflow** | Sequential HOW-TO instructions | "Do this, then this, then this" | Binary assignment → Add due dates → Check dependencies |
| **Protocol** | Behavioral WHEN/WHY patterns | "When X happens, apply principle Y" | When facing major changes, seek approval |
| **Agent** | Autonomous planning system | "Achieve this goal using available tools" | Debug this issue and create a fix |

**The Trust Test:** 
> "Can I trust it to figure out the steps?"
> - YES → Need Agent + Protocol
> - NO → Need Workflow

**Implementation Specifics:**
- Workflows define HOW with predefined data requirements
- Protocols define WHEN/WHY expecting agents to determine HOW
- Agents provide autonomous decision-making and planning
- Protocols without agents are employee handbooks without employees

**Completion Criteria:**
- Development teams can correctly classify requirements within 5 minutes
- No attempts to "protocolize" sequential workflows
- Clear documentation of which existing systems provide agent capabilities
- Architectural decisions explicitly state layer usage

## Consequences

### ✅ Positive
- Clear architectural decisions reduce wasted development effort
- Teams can quickly identify appropriate patterns for use cases
- Prevents attempting to create autonomy through instructions alone
- Enables proper layering of behavioral guidance on autonomous systems
- Reduces confusion about why certain approaches succeed/fail

### ❌ Negative
- Requires access to existing agent platforms for protocol implementation
- Teams must accept that some processes are inherently sequential
- May limit flexibility for teams wanting everything to be "intelligent"

### ⚪ Neutral
- Existing "protocol" implementations may need reclassification
- Documentation must clearly distinguish between pattern types
- Training required to understand architectural distinctions

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Everything as Agents** | Most tasks don't require autonomous planning; overengineering simple processes |
| **Everything as Workflows** | Loses adaptability and judgment for dynamic situations |
| **Protocols as Standalone** | Protocols without execution layer are just documentation |
| **No Clear Distinctions** | Current state - leads to architectural mismatches and failed implementations |