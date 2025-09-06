# 002: Establish Shared Project Directory Structure for Multi-Developer Environment

## Status

**Accepted** | Date: 2025-07-25

## Context

- Development team expanding from single developer to multiple team members working on same server
- Current ad-hoc project organization in individual user directories creates access barriers
- Need for consistent project location across all development work
- Multiple developers require read/write access to shared codebases and configurations
- Project handoffs and collaboration hindered by scattered directory structures
- No established standard for where to place development projects on shared servers
- Risk of permission conflicts and access issues with current approach

## Decision

We will establish `/home/shared/projects/` as the standard location for all development projects with proper group-based permission structure.

**Directory Structure:**

```
/home/shared/
├── projects/
│   ├── project-a/
│   ├── project-b/
│   └── [future-projects]/

```

**Permission Architecture:**

```
Group: dev-team
├── All developers added to dev-team group
├── Directory ownership: root:dev-team
├── Permissions: 2775 (group sticky bit)
└── New files inherit group ownership automatically

```

**Implementation Commands:**

```bash
# Create shared structure
sudo mkdir -p /home/shared/projects
sudo groupadd dev-team
sudo chown root:dev-team /home/shared/projects
sudo chmod 2775 /home/shared/projects

# Add developers to group
sudo usermod -a -G dev-team [developer-username]

```

**Completion Criteria:**

- `/home/shared/projects/` directory exists with correct permissions
- All current and future developers added to `dev-team` group
- Test project created successfully by any team member
- Documentation updated to reference standard location

## Consequences

### ✅ Positive

- **Consistent project location** - all developers know where to find projects
- **Seamless collaboration** - multiple developers can work on same codebase simultaneously
- **Simplified onboarding** - new developers automatically get access to all projects
- **Easier project handoffs** - no need to transfer ownership or relocate files
- **Permission inheritance** - new files automatically have correct group permissions
- **Set-once-forget-forever** - eliminates repeated permission configuration

### ❌ Negative

- **Initial setup overhead** - requires one-time configuration of groups and permissions
- **Group management responsibility** - need to remember to add new developers to group
- **Potential security consideration** - developers have access to all projects (mitigated by team trust)

### ⚪ Neutral

- **Migration required** - existing projects need to be moved to new location
- **Documentation updates** - deployment scripts and README files need path updates
- **Habit adjustment** - developers need to adapt to new standard location

## Alternatives Considered

| Alternative | Rejection Reason |
| --- | --- |
| **`/opt/projects/`** | FHS defines /opt for vendor software packages, not development projects |
| **`/srv/projects/`** | FHS defines /srv for served data (web content), not source code |
| **Individual user directories** | Creates access barriers and complicates collaboration |
| **`/var/projects/`** | FHS defines /var for variable data (logs, cache), not development code |
| **Root directory projects** | Security risk and not aligned with filesystem hierarchy standards |