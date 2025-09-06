---
description: "Caddy conf.d pattern for subdomain-based service deployment with wildcard DNS and automatic SSL"
---

# 001: caddy-confd-pattern-container-web-access

## Status

**Accepted** | Date: 2025-07-24

## Context

- Multi-service architecture requires isolated subdomain access for each service
- Wildcard DNS (*.{YOUR_DOMAIN}) configured to enable any subdomain routing
- Need operational simplicity for adding new services without complex configuration management
- SSL certificates must be automatically provisioned and renewed for all subdomains
- Team requires clear, repeatable process for service deployment that works 30 months from now
- End-of-day deployment deadlines demand fastest, most reliable configuration approach
- Services include various environments (dev/staging/prod) and monitoring tools

## Decision

We will use Caddy's conf.d/*.conf pattern for subdomain-based service deployment with standardized file naming and operational workflow.

**Architecture Pattern:**

```
Internet → Caddy (Port 80/443) → Service Containers
         ├── service.{YOUR_DOMAIN} → localhost:{SERVICE_PORT}
         ├── monitoring.{YOUR_DOMAIN} → localhost:3000
         └── app-staging.{YOUR_DOMAIN} → localhost:8082

```

**Service Organization Principle:**

**One Docker Compose Service = One Caddy Configuration File**

Each conf.d file corresponds to a complete Docker Compose service stack, handling all containers within that service through multiple routing rules.

**Example - Multi-Container Service:**
```caddyfile
# /etc/caddy/conf.d/myapp.conf
# Handles entire Docker Compose service with frontend + backend

myapp.{YOUR_DOMAIN} {
    # Frontend (default route)
    reverse_proxy localhost:{FRONTEND_PORT}
    
    # Backend API routes  
    handle_path /api/* {
        reverse_proxy localhost:{BACKEND_PORT}
    }
    
    # WebSocket routes (if applicable)
    handle_path /ws/* {
        reverse_proxy localhost:{WEBSOCKET_PORT}
    }
}
```

**File Organization:**
- ✅ `myapp.conf` → Complete service (frontend + backend + websockets)
- ❌ `myapp-frontend.conf` + `myapp-backend.conf` → Fragmented configuration

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

### One-Time Infrastructure Setup (Per Server)

**DNS Configuration:**

- ✅ Wildcard DNS record: `.{YOUR_DOMAIN}` → Server IP ({SERVER_IP})
- ✅ Main domain: `{YOUR_DOMAIN}` → Server IP

**Caddy Installation:**

- ✅ Caddy installed as systemd service
- ✅ Service running: `systemctl status caddy`

**Directory Structure:**

- ✅ `/etc/caddy/conf.d/` directory created
- ✅ Main Caddyfile configured with `import conf.d/*.conf`

### Per-Service Deployment Process (Repeatable Steps)

**Step 1: Create Service Configuration**

```bash
sudo nano /etc/caddy/conf.d/{SERVICE_NAME}.conf

```

**Step 2: Add Configuration Content**

```caddyfile
{SERVICE_NAME}.{YOUR_DOMAIN} {
    # Default route (typically frontend)
    reverse_proxy localhost:{MAIN_PORT}
    
    # API routes (if backend container exists)
    handle_path /api/* {
        reverse_proxy localhost:{API_PORT}
    }
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
curl -I http://{SERVICE_NAME}.{YOUR_DOMAIN}

# Test HTTPS access (SSL auto-generated)
curl -I https://{SERVICE_NAME}.{YOUR_DOMAIN}

```

**Step 6: Monitor SSL Certificate Generation**

```bash
# View Caddy logs for certificate provisioning
journalctl -u caddy --no-pager | grep -i certificate

```

### Service Management Operations

**Disable Service (Preserve Configuration):**

```bash
sudo mv /etc/caddy/conf.d/{SERVICE_NAME}.conf /etc/caddy/conf.d/{SERVICE_NAME}.disabled-$(date +%Y-%m-%d).conf
sudo systemctl reload caddy

```

**Re-enable Service:**

```bash
sudo mv /etc/caddy/conf.d/{SERVICE_NAME}.disabled-YYYY-MM-DD.conf /etc/caddy/conf.d/{SERVICE_NAME}.conf
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

<if_caddy_not_installed ask_for_user_confirmation=true>  
Caddy Web Server: https://caddyserver.com/docs/install - If Caddy is not installed, Ubuntu installation instructions are in the Debian/Ubuntu section
</if_caddy_not_installed>