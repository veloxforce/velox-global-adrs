# 003: Adopt Three Systematic Thinking Commands for AI-Assisted Development

## Status
**Superseded by ADR 003-2** | Date: 2025-09-01

## Context
- Teams repeatedly encounter three distinct cognitive needs during development
- **Cognitive fatigue from repeating complex prompts degrades output quality over time**
- **Same thinking patterns needed across diverse projects but manually recreated each time**
- Current ad-hoc prompting lacks systematic transparency at critical decision points
- Success of `/first_principles` command demonstrates value of structured thinking tools
- **Context Verification Specialist protocol needs systematic tools to bridge planning modes**
- Need for clear separation between investigation, analysis, and implementation thinking
- Scientific method principles underutilized in AI-assisted development workflow

## Decision
We will adopt three systematic thinking commands as cognitive tools for AI-assisted development.

**Command Toolkit:**
```
┌─────────────────────────────────────────────────────────┐
│                 COGNITIVE TOOLKIT                        │
│         (use in any order based on need)                 │
├─────────────────────────────────────────────────────────┤
│ /first_principles  →  Investigation Plan                 │
│ When: Facing uncertainty, need to identify unknowns      │
│ Output: Assumptions vs uncertainties → what to explore   │
│ Bridge: To Context Verification Specialist's             │
│         Investigation Mode                               │
├─────────────────────────────────────────────────────────┤
│ /analyze          →  Empirical Analysis                  │
│ When: Have results/data, need understanding              │
│ Output: What happened and why (no prescriptions)         │
│ IMPORTANT: Pure observation - no code changes,           │
│            no investigations, no executions              │
│ Bridge: From Investigation Findings to next mode         │
├─────────────────────────────────────────────────────────┤
│ /paint_me_a_picture →  Implementation Vision             │
│ When: Ready to build, need transparent approach          │
│ Output: How to implement (YAGNI/KISS focused)           │
│ Bridge: To Context Verification Specialist's             │
│         Implementation Mode                               │
└─────────────────────────────────────────────────────────┘
```

**Integration with Context Verification Specialist:**
- Commands provide the "missing piece" for systematic planning transitions
- `/first_principles` creates investigation plans the specialist can execute
- `/paint_me_a_picture` creates implementation visions the specialist can realize
- `/analyze` provides empirical grounding between modes

**Key Benefits:**
- **No repetition**: Commands encapsulate complex prompts - just add context
- **Project-agnostic**: Works across any domain or technology stack
- **Cognitive offload**: Human focuses on reviewing outputs, not crafting prompts
- **Protocol alignment**: Perfectly complements Context Verification Specialist workflow

**Implementation Approach:** Follow ADR 002's staged methodology (Stage 1 MVP → Stage 4 Production)

## Consequences

### ✅ Positive
- **Eliminates repetitive prompt crafting** - commands work with minimal context
- **Consistent high-quality outputs** even when human is fatigued
- **Universal applicability** across different project types and domains
- **Reduced cognitive load** - review outputs instead of writing prompts
- **Completes Context Verification Specialist toolkit** - provides systematic planning tools
- Systematic transparency at each cognitive stage
- Supports scientific method in development

### ❌ Negative
- Requires learning when to use each command
- Initial versions may need refinement through usage

### ⚪ Neutral
- Creates new slash command conventions to maintain
- Shifts from ad-hoc to systematic prompting
- Changes role from prompt writer to output reviewer
- Integrates with existing protocol infrastructure

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Keep explaining prompts manually** | Cognitive fatigue leads to degraded prompts and outputs over time |
| **Project-specific templates** | Requires constant recreation for new domains |
| **Single universal command** | Different cognitive modes need different structures |
| **Ad-hoc prompting** | Lacks repeatability and systematic improvement |
| **Extend existing protocol only** | Protocol needs systematic tools, not more complexity |