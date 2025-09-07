# Git - Worktree "Missing" After Push/Clone

**Symptoms:**
- Developer creates worktree locally and pushes branch successfully
- On different machine/server: `"git worktree list"` shows no worktrees
- Directory exists but is empty or contains wrong branch
- Error: `"'worktree_branch' could be both a local file and a tracking branch"`
- Code appears in GitHub but not accessible in expected worktree directory

**Root Cause:**
- Git worktrees are **local-only filesystem constructs**, not synchronized git objects
- Only the **branch** gets pushed to remote, never the worktree directory structure
- Each environment must create its own worktrees from existing branches
- Worktrees are personal workspace organization, not shared project structure

**Solution:**

1. **Remove any conflicting directories:**
```bash
# Remove empty/broken worktree directory
rm -rf worktree_branch_name
```

2. **Create worktree from remote branch:**
```bash
# Fetch latest branches
git fetch

# Create worktree from remote branch
git worktree add worktree_branch_name origin/worktree_branch_name
```

3. **Verify worktree structure:**
```bash
# Check worktree is properly configured
git worktree list

# Verify .git file exists (not folder)
ls -la worktree_branch_name/.git
```

**Verification:**
```bash
# Should show your worktree path and branch
git worktree list
# Expected: /path/to/worktree_branch_name [worktree_branch_name]

# Should be a file, not directory
file worktree_branch_name/.git  
# Expected: worktree_branch_name/.git: ASCII text

# Code should be accessible
ls worktree_branch_name/
# Expected: Your project files
```

**Related Issues:**
- **Prevention:** Document that worktrees need recreation on each machine
- **Alternative:** Use regular branch checkout if worktree complexity isn't needed
- **Team workflow:** Share branch names, not worktree commands
- **Cleanup:** `git worktree remove <worktree>` when switching focus

**Related Resources:**
- Git worktree documentation: `git worktree --help`
- Each developer manages their own worktrees independently
- Consider regular branches for simpler team workflows