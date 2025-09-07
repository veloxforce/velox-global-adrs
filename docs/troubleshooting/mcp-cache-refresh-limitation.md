# MCP Server - YAML Frontmatter Not Updating After GitHub Push

**Symptoms:**
- MCP `adr://list` resource shows "No description available" for files that have YAML frontmatter on GitHub
- GitHub raw content confirms YAML frontmatter is present: `---\ndescription: "..."\n---`
- In-session MCP reconnection (`/mcp` command) does not refresh cached metadata
- Files with existing descriptions continue to work normally

**Root Cause:**
- MCP server implements session-level caching of GitHub content metadata
- YAML frontmatter parsing occurs during initial GitHub fetch, then cached
- Cache persists across in-session reconnections but not across fresh Claude Code sessions
- Exact caching mechanism and TTL duration unknown (server-side implementation)

**Solution:**

1. **Exit current Claude Code session completely**
2. **Start fresh Claude Code session** 
3. **Verify updated descriptions:**
   ```
   Use ReadMcpResourceTool with server: velox-adrs, uri: adr://list
   Check that previously missing descriptions now appear
   ```

**Verification:**
- MCP list resource should show meaningful descriptions instead of "No description available"
- Files with newly added YAML frontmatter should display parsed descriptions
- Expected result: All ADRs with valid YAML frontmatter show descriptive text

**Related Issues:**
- Affects all YAML frontmatter updates pushed to GitHub after MCP session starts
- Does not affect files that already had YAML frontmatter before session began
- Workaround required for development workflow when adding/updating ADR metadata

**Prevention:**
- Plan YAML frontmatter additions in batches to minimize session restart frequency
- Consider this limitation when updating ADR descriptions during development

**Technical Notes:**
- Cache behavior is server-side, not client-side
- GitHub content accessibility remains constant throughout (not a GitHub sync issue)
- Fresh session forces complete GitHub re-fetch and YAML re-parsing