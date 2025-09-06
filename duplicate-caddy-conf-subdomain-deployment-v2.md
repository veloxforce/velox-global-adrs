# 001: Implement Caddy conf.d Pattern for Subdomain Service Deployment on Staging Server

## Status

**Accepted** | Date: 2025-07-24

## Context

- Multi-service architecture requires isolated subdomain access for each service
- Wildcard DNS (*.iitr-cloud.de) already configured to enable any subdomain routing
- Need operational simplicity for adding new services without complex configuration management
- SSL certificates must be automatically provisioned and renewed for all subdomains
- Team requires clear, repeatable process for service deployment that works 30 months from now
- End-of-day deployment deadlines demand fastest, most reliable configuration approach
- Services include RAG environments (dev/staging/prod) and monitoring tools (OpenLit)

## Decision

We will use Caddy's conf.d/*.conf pattern for subdomain-based service deployment with standardized file naming and operational workflow.

**Architecture Pattern:**

```
Internet → Caddy (Port 80/443) → Service Containers
         ├── service.iitr-cloud.de → localhost:PORT
         ├── openlit.iitr-cloud.de → localhost:3000
         └── rag-staging.iitr-cloud.de → localhost:8082

```

**Configuration Structure:**

```
/etc/caddy/
├── Caddyfile                    # import conf.d/*.conf
└── conf.d/
    ├── service-name.conf        # Active configurations
    └── service-name.disabled-YYYY-MM-DD.conf  # Disabled backups

```

**Completion Criteria:**

- conf.d directory structure established
- Main Caddyfile configured with import statement
- Service configuration template documented
- Operational workflow validated with OpenLit deployment

## Process

### One-Time Infrastructure Setup (Already Done - Never Touch Again)

**DNS Configuration:**

- ✅ Wildcard DNS record: `.iitr-cloud.de` → Server IP (136.243.71.58)
- ✅ Main domain: `iitr-cloud.de` → Server IP

**Caddy Installation:**

- ✅ Caddy installed as systemd service
- ✅ Service running: `systemctl status caddy`

**Directory Structure:**

- ✅ `/etc/caddy/conf.d/` directory created
- ✅ Main Caddyfile configured with `import conf.d/*.conf`

### Per-Service Deployment Process (Repeatable Steps)

**Step 1: Create Service Configuration**

```bash
sudo nano /etc/caddy/conf.d/SERVICE-NAME.conf

```

**Step 2: Add Configuration Content**

```
SERVICE-NAME.iitr-cloud.de {
    reverse_proxy localhost:PORT
}

```

**Step 3: Validate Configuration**

```bash
sudo caddy validate --config /etc/caddy/Caddyfile

```

**Step 4: Reload Caddy (Graceful, No Downtime)**

```bash
sudo systemctl reload caddy

```

**Step 5: Verify Deployment**

```bash
# Check service status
systemctl status caddy

# Test HTTP access (will redirect to HTTPS)
curl -I <http://SERVICE-NAME.iitr-cloud.de>

# Test HTTPS access (SSL auto-generated)
curl -I <https://SERVICE-NAME.iitr-cloud.de>

```

**Step 6: Monitor SSL Certificate Generation**

```bash
# View Caddy logs for certificate provisioning
journalctl -u caddy --no-pager | grep -i certificate

```

### Service Management Operations

**Disable Service (Preserve Configuration):**

```bash
sudo mv /etc/caddy/conf.d/SERVICE-NAME.conf /etc/caddy/conf.d/SERVICE-NAME.disabled-$(date +%Y-%m-%d).conf
sudo systemctl reload caddy

```

**Re-enable Service:**

```bash
sudo mv /etc/caddy/conf.d/SERVICE-NAME.disabled-YYYY-MM-DD.conf /etc/caddy/conf.d/SERVICE-NAME.conf
sudo systemctl reload caddy

```

**List Active Configurations:**

```bash
ls -la /etc/caddy/conf.d/*.conf

```

## Consequences

### ✅ Positive

- **Operational Simplicity**: Single file per service, clear enable/disable workflow
- **Automatic SSL**: Let's Encrypt certificates provisioned and renewed automatically
- **Zero Downtime**: Configuration reloads without service interruption
- **Industry Standard**: Modern approach used by nginx and other web servers
- **Scalable**: Easy to add unlimited services without architectural changes
- **Future-Proof**: Clear documentation enables team members to add services months later

### ❌ Negative

- **Configuration Loss Risk**: Disabled services lose configuration if not properly renamed
- **No Central Repository**: Can't easily see all possible service configurations
- **File Management**: Must remember to use descriptive names for disabled configurations

### ⚪ Neutral

- **Learning Curve**: Team must learn conf.d pattern instead of sites-available/sites-enabled
- **Backup Strategy**: Need consistent naming convention for disabled configurations

## Alternatives Considered

| Alternative | Rejection Reason |
| --- | --- |
| **sites-available/sites-enabled** | Symlink complexity and broken link risks outweigh benefits for small-scale deployment |
| **Single Caddyfile** | Becomes unwieldy with multiple services, harder to manage individual service configs |
| **Docker Labels** | Requires container orchestration complexity not needed for current scale |