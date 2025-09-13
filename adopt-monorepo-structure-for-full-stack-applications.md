# Adopt Monorepo Structure for Full-Stack Applications

## Status
**Accepted** | Date: 2025-09-02

## Context
- Frontend and backend in separate repositories cause deployment synchronization failures when version dependencies exist
- Code reviews require multiple PRs across repositories, fragmenting context and increasing review complexity
- Local development requires managing multiple git clones, terminal sessions, and port configurations
- Deployment rollbacks become complex when frontend and backend changes are interdependent
- Onboarding new developers requires documenting and maintaining multiple repository setup procedures

## Decision
We will consolidate full-stack applications into a monorepo structure with standardized directory layout.

**Repository Structure:**
```
project-name/
├── frontend/          # Vercel-deployed React/Vue/Next application
├── backend/           # VPS-deployed API service (FastAPI/Node/etc)
├── deployment/        # Docker Compose and environment configs
├── docs/              # ADRs and project documentation
└── Makefile           # Standardized operational commands
```

**Deployment Flow:**
```
Git Push to Main
    ├── Vercel (automatic)
    │   └── Builds frontend/ → CDN deployment
    └── GitHub Actions
        └── SSH to VPS → docker-compose up (backend/)
```

**Completion Criteria:**
- Single git repository containing all application components
- Docker Compose references local `../backend` path instead of git URLs
- Vercel connected to monorepo with base directory set to `frontend/`
- CI/CD pipeline triggers both deployment targets from single commit

**Implementation Specifics:**
- Configure Vercel with `"rootDirectory": "frontend"` in settings
- Docker builds use `context: ../backend` for local paths

## Consequences

### ✅ Positive
- Atomic deployments ensure frontend and backend remain synchronized
- Single PR encompasses entire feature across full stack
- Simplified local development with single clone and unified scripts
- Consistent versioning across all application components

### ❌ Negative
- Repository size increases 3x affecting clone and fetch times
- Merge conflicts potentially span multiple application layers
- All developers have visibility to entire codebase regardless of focus area
- CI/CD runs all checks even for component-specific changes

### ⚪ Neutral
- IDE requires monorepo-aware plugins for optimal performance
- Git history combines all component changes in single timeline
- Deployment scripts must handle partial updates efficiently

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Multi-repository** | Deployment coordination failures, version mismatch issues, complex rollback procedures |
| **Git submodules** | Steep learning curve, complex update procedures, frequent detached HEAD states |
| **Microservices** | Premature optimization, operational overhead exceeds current requirements |

---

*This ADR establishes monorepo as the standard structure for full-stack applications, eliminating repository fragmentation and deployment coordination issues.*