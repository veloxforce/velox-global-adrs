# 001: Standardize Docker Compose Makefile Targets

## Status

**Accepted** | Date: 2025-09-02

## Context

- Production and staging environments require consistent command interfaces
- Docker Compose commands are verbose and error-prone when typed manually
- Accidental environment deployments occur when default behaviors are unclear
- Teams waste time remembering environment-specific deployment commands
- Resource conflicts (like email inbox polling) require clear environment separation
- Default behaviors should follow principle of least surprise and prevent accidents

## Decision

We will implement a standardized Makefile structure for Docker Compose projects with production-first defaults and explicit staging operations.

**Core Principles:**
- Production is the default environment for all operations
- Staging requires explicit command prefix
- `make` without arguments must show help (never deploy)
- Only two environments: staging and production (development is local)

**Standard Target Structure:**

```makefile
.DEFAULT_GOAL := help  # REQUIRED: Prevent accidental deployments

# Production Operations (Default)
deploy          # Build and deploy production (docker compose up -d --build)
start           # Start production services (docker compose up -d)
restart         # Restart production services (docker compose restart)
stop            # Stop production containers (docker compose down)
logs            # Follow production logs (docker compose logs -f)

# Staging Operations (Explicit)
staging         # Build and deploy staging
staging-start   # Start staging services
staging-restart # Restart staging services
staging-stop    # Stop staging containers
staging-logs    # Follow staging logs

# Utility Operations
help            # Display available commands (DEFAULT)
status          # Show all running containers
clean           # Remove unused Docker resources
```

**Implementation Requirements:**
1. `.DEFAULT_GOAL := help` must be first non-comment line
2. Production compose file: `docker-compose.prod.yml`
3. Staging compose file: `docker-compose.staging.yml`
4. Consistent container naming to prevent conflicts
5. Help text must clearly indicate production vs staging

**Example Makefile Header:**
```makefile
.DEFAULT_GOAL := help

.PHONY: help deploy start restart stop logs staging staging-start staging-restart staging-stop staging-logs status clean

# Production Commands (Default)
deploy: ## Deploy production environment
	@docker compose -f deployment/docker-compose.prod.yml up -d --build

# Staging Commands (Explicit)
staging: ## Deploy staging environment
	@docker compose -f deployment/docker-compose.staging.yml up -d --build
```

## Consequences

### ✅ Positive

- **Safety First**: `make` shows help, preventing accidental deployments
- **Production Focus**: Simple commands for the most critical environment
- **Explicit Staging**: Clear intention required for staging operations
- **Reduced Cognitive Load**: Fewer commands to remember
- **Conflict Prevention**: Clear separation prevents resource conflicts
- **Fast Onboarding**: New developers understand immediately

### ❌ Negative

- **Breaking Change**: Existing muscle memory needs retraining
- **Staging Verbosity**: Staging commands are longer than before
- **Migration Effort**: Existing projects need Makefile updates

### ⚪ Neutral

- Production bias may not suit all projects equally
- Some teams may prefer environment parity in commands
- Requires discipline to maintain standard across projects

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Environment prefix for all** (`prod-deploy`, `staging-deploy`) | Too verbose for common production operations |
| **Environment variable** (`ENV=prod make deploy`) | Hidden state, not discoverable, error-prone |
| **Separate Makefiles** | Increases file count, harder to maintain |
| **No default environment** | Forces choice every time, slows down operations |
| **Dev/Staging/Prod trinity** | Development doesn't need containerization |
