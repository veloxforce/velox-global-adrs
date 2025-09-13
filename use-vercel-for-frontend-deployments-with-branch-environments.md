# 011: Use Vercel for Frontend Deployments with Branch Environments

## Status
**Accepted** | Date: 2025-09-02

## Context
- Frontend applications require deployment pipeline with minimal configuration overhead
- Development teams need preview deployments for feature testing before production
- Manual deployment processes create bottlenecks and increase risk of human error
- Branch-based deployment workflows must support main=production and preview environments
- Project-level configuration needed for monorepo subdirectory deployments

## Decision
We will use Vercel for frontend deployments with automatic branch-based environment generation.

**Deployment Flow:**
```
Git Push
    ├── main branch → Production deployment
    └── any other branch → Preview deployment
```

**Setup Requirements:**
```bash
# 1. Configure Git with GitHub-associated email (REQUIRED for deployments)
git config --global user.email "84270258+MariusWilsch@users.noreply.github.com"
git config --global user.name "Marius Santiago Wilsch"

# 2. Verify Vercel CLI presence
command -v vercel >/dev/null 2>&1 || echo "Vercel CLI not found"

# 3. Check project linking status
[ -d "frontend/.vercel" ] || echo "Project not linked"

# 4. Commands from monorepo root use --cwd flag
vercel --cwd frontend/ [command]
```

**Completion Criteria:**
- Git configured with GitHub-associated email for deployment permissions
- Vercel CLI accessible in environment
- Project linked to Vercel (`frontend/.vercel/` directory exists)
- Commands executable from monorepo root using --cwd flag
- Automatic deployments triggered on git push

**Implementation Specifics:**
- Link project from frontend directory: `cd frontend/ && vercel link`
- Use --cwd flag for commands from monorepo root
- Default branch mapping: main=production, others=preview
- Monorepo support through subdirectory targeting

**⚠️ Critical Monorepo Configuration:**
- **Root Directory**: Must be set to `frontend` in Vercel dashboard
  - Location: Project Settings → General → Root Directory
  - Value: `frontend` (not `.`)
  - **Without this**: `sh: line 1: vite: command not found` deployment failure
  - **Why**: Repository has separate frontend/ and backend/ apps, vite dependency lives in frontend/package.json
## Backend Integration Pattern

### Why /api Prefix Convention

When deploying frontend (Vercel) and backend (VPS) separately, the `/api` prefix enables **environment-agnostic frontend code**:

**Without /api prefix:**
```javascript
// Hardcoded backend URLs
const API_BASE_URL = 'https://api.example.com'  // Environment-specific
fetch(`${API_BASE_URL}/users`)
```

**With /api prefix:**
```javascript
// Environment-agnostic
fetch('/api/users')  // Infrastructure handles routing
```

**Core Benefits:**
- **Framework conventions**: Vite, Next.js expect `/api` for proxy configuration
- **Same code, multiple environments**: Dev proxy and production routing use identical frontend calls
- **Infrastructure flexibility**: Easy to switch backends without code changes
- **Development simplicity**: `'/api': 'localhost:8000'` proxy eliminates CORS issues

### Technical Implementation

**Problem**: Frontend calls `/api/internal/generate-csv`, Backend serves `/internal/generate-csv`  
**Solution**: Reverse proxy strips `/api` prefix before forwarding to backend

```nginx
# Caddy configuration (required for separated deployments)
domain.com {
    handle /api/* {
        uri strip_prefix /api
        reverse_proxy localhost:8081
    }
    reverse_proxy localhost:8081  # Frontend static files
}
```

**Environment Routing:**

| Environment | Frontend | API Calls | Backend | Routing |
|-------------|----------|-----------|---------|---------|
| **Development** | `localhost:3000` | `/api/*` → `localhost:8000` | Dev server proxy |
| **Production** | `domain.com` (Vercel) | `/api/*` → `localhost:8081` | Caddy strips `/api` |

### When This Pattern Applies

**✅ Required For:**
- Vercel frontend + VPS backend deployments
- Any separated hosting (frontend CDN + backend server)
- Development environments with proxy needs

**❌ Not Required For:**
- Same-domain deployments (both frontend + backend on VPS)
- Backends that include `/api` in route definitions
- Microservices with dedicated API domains (`api.domain.com`)

## Consequences

### ✅ Positive
- Zero-configuration deployments after initial setup
- Automatic preview URLs for all non-main branches
- Built-in SSL certificates and CDN distribution
- Simple promotion workflow from preview to production

### ❌ Negative
- Platform lock-in to Vercel hosting service
- Limited customization compared to self-hosted solutions
- Dependency on external service availability
- Git commit author must match GitHub account for deployment permissions

### ⚪ Neutral
- CLI dependency for project setup and management
- --cwd flag required for commands from monorepo root
- Branch naming conventions determine environment assignment

## Alternatives Considered

| Alternative | Rejection Reason |
|-------------|------------------|
| **Self-hosted CI/CD** | Configuration complexity outweighs benefits for frontend deployment |
| **Netlify** | Similar feature set without existing organizational familiarity |
| **GitHub Pages** | Limited to static sites, no preview deployment automation |

---

*This ADR establishes Vercel as the standard frontend deployment platform, leveraging automatic branch-based environments with minimal configuration overhead.*