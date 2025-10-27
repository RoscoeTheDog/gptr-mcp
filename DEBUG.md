# gptr-mcp Debugging Session Log

**Version:** 1.1.0
**Last Updated:** 2025-10-27
**Primary Issues:**
- Testing quick_search tool via SSE transport
- MCP server configuration and Windows Unicode errors

---

## Problem Discovery

### Initial Issue
- Native test script `tests/test_mcp_server.py` (from upstream) crashes with `anyio.ClosedResourceError`
- Error occurs in MCP SSE handler (`mcp/server/sse.py:249`)
- Root cause: Test script closes SSE connection before server can send response back

### Error Pattern
```
anyio.ClosedResourceError
  at anyio/streams/memory.py:212 in send_nowait
  at mcp/server/sse.py:249 in handle_post_message
```

---

## Investigation Steps

1. **Confirmed server mode:** Running in STDIO mode by default
2. **Transport modes:**
   - STDIO (default) - for Claude Desktop integration
   - SSE (HTTP) - for web clients, requires `MCP_TRANSPORT=sse`
3. **Server resilience:** ✅ Server handles SSE errors gracefully, continues running
4. **Tool-level exception handling:** ✅ Each tool has try/catch, returns error dicts

---

## SSE Protocol Issue

**Problem:** SSE requires bidirectional communication
- Client must keep SSE stream **open** to receive responses
- Sending POST to `/messages/?session_id=X` triggers async processing
- Response comes back via the **SSE stream**, not HTTP response body
- Test script was closing stream after getting session ID

**Failed Approaches:**
- Python httpx with separate clients (race condition)
- Python subprocess.run with curl (timeout issues)

**Working Approach:**
- Bash script using curl properly
- Session-only approach (send commands, don't wait for results)

---

## Working Test Script

`test_quick_search.sh` successfully:
1. ✅ Gets session ID from `/sse` endpoint
2. ✅ Initializes MCP protocol
3. ✅ Sends `quick_search` tool request
4. ✅ Receives HTTP 202 Accepted

**Limitation:** Script doesn't capture results (would require keeping SSE stream open)

---

## Server Startup

**CMD:**
```cmd
set MCP_TRANSPORT=sse && python server.py
```

**PowerShell:**
```powershell
$env:MCP_TRANSPORT="sse"; python server.py
```

**Git Bash:**
```bash
export MCP_TRANSPORT=sse && python server.py
```

Note: Session-only (not system-wide). Server runs on `http://0.0.0.0:8000`

---

## Generated Test Files

| File | Status | Purpose |
|------|--------|---------|
| `tests/test_mcp_server.py` | ❌ Native (upstream), has SSE bug | Original test with connection issue |
| `test_quick_search.py` | ❌ Generated, emoji encoding issues | First fix attempt |
| `simple_test.py` | ❌ Generated, httpx race condition | Second fix attempt |
| `working_test.py` | ❌ Generated, subprocess timeout | Third fix attempt |
| `test_quick_search.sh` | ✅ Generated, works | Bash script, sends commands successfully |

---

## Key Findings

1. **Upstream bug confirmed:** `tests/test_mcp_server.py` has SSE session handling bug
2. **Server exception handling:** Works correctly, errors don't crash server
3. **SSE complexity:** Proper testing requires keeping stream open for bidirectional communication
4. **Tool execution:** Commands reach server successfully (202 Accepted), tools likely execute but results not captured

---

## Root Cause Analysis

**The SSE transport has a fundamental design constraint:**
- Requires **stateful, long-lived SSE connection** kept open for entire session
- Client must **simultaneously** listen on SSE stream AND send POST requests
- Server sends responses back via the SSE stream, not HTTP response body
- Python httpx cannot easily maintain this bidirectional communication pattern

**Evidence:**
- Server accepts messages (202 Accepted) ✅
- But tools never execute (no log output) ❌
- Error occurs when server tries to respond via closed SSE stream ❌

## Conclusion

**SSE mode is not designed for direct HTTP testing** - it's meant for:
- Web-based MCP clients with native SSE support
- Docker/n8n integrations
- Embedded in web applications

**STDIO mode is the proper transport for:**
- Claude Desktop integration ✅
- Direct MCP client testing ✅
- Command-line usage ✅

## Next Steps

- Use STDIO mode for local testing and Claude Desktop
- SSE mode only for production web deployments
- Consider creating STDIO-based test client instead

---

## Environment

- **OS:** Windows (Git Bash)
- **Python:** 3.13
- **Server PID:** 1368 (during session)
- **Transport:** SSE on port 8000
- **Repo:** Fork of `assafelovic/gptr-mcp`

---

## MCP Configuration Architecture (Claude Code)

**Date:** 2025-10-27
**Issue:** gptr-mcp MCP server not appearing in Claude Code despite configuration

### Configuration File Hierarchy

Claude Code uses a **single authoritative configuration file** with a specific hierarchy:

**AUTHORITATIVE CONFIG:**
- **Location:** `~/.claude.json` (root-level `mcpServers` section)
- **Managed by:** `claude mcp` CLI commands
- **Scope:** Global (user-level) or local (project-specific) based on flags
- **Format:** JSON with literal values (NOT variable references)

**DEPRECATED/NOT USED:**
- **Location:** `~/.claude/mcp_servers.json`
- **Status:** ❌ **NOT READ BY CLAUDE CODE**
- **Why it exists:** May be created by agents or other tooling, but Claude Code ignores it
- **Action:** Can be safely deleted

### Critical Discovery: mcp_servers.json is Ignored

**Problem Pattern:**
```bash
# This file exists but is NEVER read by Claude Code:
~/.claude/mcp_servers.json

# This is the ONLY file Claude Code reads:
~/.claude.json
```

**Evidence:**
- gptr-mcp was configured in `~/.claude/mcp_servers.json` with proper JSON
- Server DID NOT appear in Claude Code's `/mcp` list
- Only after adding to `~/.claude.json` did it become available

**Root Cause:**
- `mcp_servers.json` is not part of Claude Code's actual architecture
- It may be a legacy file, agent-generated artifact, or documentation error
- Claude Code exclusively reads `.claude.json`

### Configuration Methods

#### Method 1: CLI Commands (Recommended)

**Add MCP Server (with env var expansion):**
```bash
# Global scope (available in all projects)
claude mcp add-json --scope user gptr-mcp '{...}'

# Local scope (project-specific, default)
claude mcp add-json gptr-mcp '{...}'
```

**Important Notes:**
- `--scope user` → Adds to root-level `mcpServers` section (global)
- `--scope local` (default) → Adds under project path key (project-specific)
- Environment variables are expanded at configuration time (see below)

**List MCP Servers:**
```bash
claude mcp list
```

**Get Specific Server:**
```bash
claude mcp get gptr-mcp
```

**Remove Server:**
```bash
claude mcp remove gptr-mcp
```

#### Method 2: Manual JSON Editing (Not Recommended)

While you CAN manually edit `~/.claude.json`, this is error-prone on Windows due to path escaping issues. Always prefer `claude mcp add-json`.

### Environment Variable Expansion Behavior

**CRITICAL UNDERSTANDING:** Environment variables are expanded at **configuration time**, NOT runtime.

**How It Works:**
```bash
# When you run this command:
claude mcp add --scope user gptr-mcp \
  --env OPENAI_API_KEY=$OPENAI_API_KEY

# Claude Code:
# 1. Reads $OPENAI_API_KEY from your current shell environment
# 2. EXPANDS it to the literal value (e.g., "sk-proj-abc123...")
# 3. STORES the literal value in ~/.claude.json
# 4. Result: {"env": {"OPENAI_API_KEY": "sk-proj-abc123..."}}
```

**Consequences:**
- ✅ MCP servers get valid API keys immediately
- ❌ Changing OS environment variables does NOT auto-update MCP servers
- ❌ You must re-run `claude mcp add` to update keys
- ⚠️ API keys are stored as **plaintext** in `~/.claude.json`

**Why This Matters:**
```bash
# Scenario 1: You rotate your API key
export OPENAI_API_KEY="new-key-here"

# MCP server STILL uses old key from ~/.claude.json
# You must manually update:
claude mcp remove gptr-mcp
claude mcp add-json --scope user gptr-mcp '{...}'  # with new key
```

**Security Implication:**
- API keys are stored in plaintext in `~/.claude.json`
- File permissions should be restricted (chmod 600 on Unix)
- On Windows, ensure user-only NTFS permissions
- Consider using `${VARIABLE_NAME}` syntax in manual JSON (but see expansion notes)

### Scope System

Claude Code uses a **scope system** to determine where MCP server configs are stored:

**Scope Options:**
| Scope | Flag | Storage Location | Use Case |
|-------|------|------------------|----------|
| **user** | `--scope user` | `~/.claude.json` root `mcpServers` | Global availability across all projects |
| **local** | `--scope local` (default) | `~/.claude.json` under project path key | Project-specific servers |
| **project** | `--scope project` | Similar to local | Project-specific (alias) |

**Example: Global vs Local**
```json
{
  "mcpServers": {
    "gptr-mcp": { /* user scope - global */ }
  },
  "/c/Users/Admin/Documents/GitHub/project-x": {
    "mcpServers": {
      "custom-server": { /* local scope - project-specific */ }
    }
  }
}
```

**Best Practice:**
- Use `--scope user` for general-purpose tools (gptr-mcp, context7, etc.)
- Use `--scope local` for project-specific integrations

### Path Escaping on Windows

**Problem:** Windows paths require careful escaping in JSON.

**Escaping Rules:**
```json
// WRONG (backslashes stripped):
"command": "C:\python313\python.exe"

// CORRECT (double-escaped):
"command": "C:\\\\python313\\\\python.exe"
```

**Why `claude mcp add-json` is Preferred:**
- Handles escaping automatically
- More reliable than `claude mcp add` on Windows
- Less prone to shell interpretation issues

**Example:**
```bash
# Windows - use add-json with properly escaped JSON
claude mcp add-json --scope user gptr-mcp '{
  "type": "stdio",
  "command": "C:\\\\python313\\\\python.exe",
  "args": ["C:\\\\Users\\\\Admin\\\\Documents\\\\GitHub\\\\gptr-mcp\\\\server.py"],
  "env": {
    "OPENAI_API_KEY": "'${OPENAI_API_KEY}'",
    "TAVILY_API_KEY": "'${TAVILY_API_KEY}'"
  }
}'
```

### Verification Commands

**Check if server is configured:**
```bash
claude mcp list | grep gptr-mcp
```

**Inspect server configuration:**
```bash
claude mcp get gptr-mcp
```

**Expected output:**
```json
{
  "type": "stdio",
  "command": "C:\\python313\\python.exe",
  "args": ["C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py"],
  "env": {
    "OPENAI_API_KEY": "sk-proj-...",  // LITERAL VALUE, not reference
    "TAVILY_API_KEY": "tvly-..."
  }
}
```

---

## Windows Unicode/Emoji Encoding Errors

**Date:** 2025-10-27
**Issue:** MCP server crashes immediately on startup with Unicode encoding error

### Problem Pattern

**Error Message:**
```
UnicodeEncodeError: 'charmap' codec can't decode byte 0x8f in position X
```

**Error Location:**
```
File "server.py", line 290, in run_server
    print(f"🚀 GPT Researcher MCP Server starting...")
```

### Root Cause

**Multiple factors:**
1. **Windows Console Encoding (cp1252):** Cannot handle Unicode emojis
2. **MCP stdio Protocol Limitation:** stdout is reserved for JSON-RPC communication
3. **Print Statement Interference:** Using `print()` in stdio MCP servers breaks protocol

**Why This Happens:**
- Windows console defaults to cp1252 encoding (legacy, limited character set)
- Emoji characters (🚀, ❌, ✅) are Unicode characters outside cp1252 range
- MCP stdio transport uses stdout exclusively for JSON-RPC messages
- Any `print()` to stdout interferes with the protocol

### Solution

**Replace print() with logger calls:**

```python
# BEFORE (causes crash):
print(f"🚀 GPT Researcher MCP Server starting with {transport} transport...")
print(f"❌ MCP Server error: {str(e)}")
print("✅ MCP Server stopped")

# AFTER (fixed):
logger.info(f"Starting GPT Researcher MCP Server with {transport} transport...")
logger.error(f"Error running MCP server: {str(e)}")
logger.info("MCP Server stopped")
```

**Why This Works:**
- `logger` outputs to **stderr**, which is allowed in MCP stdio protocol
- stderr is not subject to the same encoding constraints
- Removes emoji characters (unnecessary for logging)
- Follows MCP protocol specification

### MCP stdio Protocol Rules

**stdout:** Reserved exclusively for JSON-RPC messages
- ✅ MCP protocol messages ONLY
- ❌ NO print() statements
- ❌ NO debug output
- ❌ NO user-facing messages

**stderr:** Allowed for logging
- ✅ Logger output (logging.info, logging.error, etc.)
- ✅ Debug information
- ✅ Error messages
- ✅ Diagnostic output

### Files Modified

**server.py:**
- Line 290-291: Removed emoji print statements
- Line 311: Removed error emoji print
- Line 313: Removed success emoji print

**Before:**
```python
logger.info(f"Starting GPT Researcher MCP Server with {transport} transport...")
print(f"🚀 GPT Researcher MCP Server starting with {transport} transport...")
print("   Check researcher_mcp_server.log for details")
```

**After:**
```python
logger.info(f"Starting GPT Researcher MCP Server with {transport} transport...")
# Note: Cannot use print() with stdio transport - stdout is reserved for MCP protocol
# Logging to stderr via logger is OK
```

### Validation

**Before Fix:**
- Server crashed immediately with Unicode error
- No MCP tools available in Claude Code
- Debug logs showed encoding error at line 290

**After Fix:**
- Server starts successfully
- MCP connection established
- All tools (deep_research, quick_search, write_report) functional

### Platform Notes

**Windows:**
- Default encoding: cp1252 (limited character set)
- Cannot handle Unicode emojis in print statements
- Must use logger for all output in stdio MCP servers

**macOS/Linux:**
- Default encoding: UTF-8 (full Unicode support)
- Can handle emojis, but should still avoid print() in stdio mode
- MCP protocol rules apply regardless of platform

---

## Debugging Methodology

**Date:** 2025-10-27
**Issue:** Systematic approach to debugging MCP server connection failures

### Step 1: Verify Configuration File

**Check authoritative config:**
```bash
# Windows
cat ~/.claude.json | grep -A 10 gptr-mcp

# Or use CLI
claude mcp get gptr-mcp
```

**Common Issues:**
- Server not in config file
- Server in wrong scope (local vs user)
- Incorrect path escaping
- Missing environment variables

### Step 2: Check Debug Logs

**Location:** `~/.claude/debug/*.txt`

**What to look for:**
```
# Connection attempt
[timestamp] Attempting to connect to MCP server: gptr-mcp

# Success
[timestamp] MCP server connected: gptr-mcp

# Failure (with error)
[timestamp] MCP server failed to connect: gptr-mcp
[timestamp] Error: UnicodeEncodeError: 'charmap' codec...
```

**Key Patterns:**
- Unicode/encoding errors → emoji/print() issue
- Path not found → escaping issue
- Permission denied → file permissions
- Module not found → missing dependencies

### Step 3: Check Deprecated Files

**Verify mcp_servers.json is NOT interfering:**
```bash
ls ~/.claude/mcp_servers.json
```

**If it exists:**
- ❌ **NOT used by Claude Code**
- Can be safely deleted
- May cause confusion if you expect it to work

### Step 4: Verify Scope

**Check which scope the server is in:**
```bash
claude mcp get gptr-mcp
```

**If not found:**
- May be in wrong scope (local vs user)
- Try `claude mcp list` to see all servers
- Re-add with correct `--scope user` flag

### Step 5: Test Server Manually

**Run server directly:**
```bash
cd C:\Users\Admin\Documents\GitHub\gptr-mcp
python server.py
```

**Expected output:**
```
[timestamp][INFO] - Starting GPT Researcher MCP Server with stdio transport...
[timestamp][INFO] - Using STDIO transport (Claude Desktop compatible)
```

**If server crashes:**
- Check Python dependencies
- Check API keys in environment
- Check for print() statements in stdio mode

### Debugging Checklist

```
□ Server in ~/.claude.json? (NOT mcp_servers.json)
□ Correct scope (--scope user for global)?
□ Path escaping correct (C:\\\\...)?
□ Environment variables present?
□ Debug logs checked?
□ Server runs manually without errors?
□ No print() statements to stdout?
□ All dependencies installed?
```

---

## Root Cause Analysis: Initial Connection Failure

**Date:** 2025-10-27
**Issue:** gptr-mcp server failed to connect to Claude Code despite configuration

### Timeline of Events

1. **Initial State:**
   - Server configured in `~/.claude/mcp_servers.json`
   - Server NOT appearing in Claude Code `/mcp` list
   - Manual server execution worked (`python server.py`)

2. **First Hypothesis (WRONG):**
   - Configuration caching issue
   - Server limit reached
   - Scope misconfiguration

3. **Correct Discovery:**
   - `mcp_servers.json` is NOT read by Claude Code
   - `.claude.json` is the authoritative config
   - User correctly identified the architecture discrepancy

4. **Added to Correct Config:**
   - Used `claude mcp add-json --scope user`
   - Server appeared in `/mcp` list
   - BUT: Status showed "Failed to connect"

5. **Second Issue (Unicode Crash):**
   - Debug logs revealed Unicode encoding error
   - Emoji characters in print() statements
   - Windows cp1252 encoding cannot handle emojis
   - stdout interference with MCP JSON-RPC protocol

6. **Final Fix:**
   - Removed emoji print() statements
   - Replaced with logger calls to stderr
   - Server connected successfully

### Root Causes (Dual Issue)

**Primary Cause 1: Configuration Architecture Misunderstanding**
- **Symptom:** Server not appearing in `/mcp` list
- **Root Cause:** Configured in deprecated `mcp_servers.json` instead of `.claude.json`
- **Fix:** Use `claude mcp add-json --scope user` to add to authoritative config

**Primary Cause 2: Windows Unicode Encoding**
- **Symptom:** Server crashes immediately on startup
- **Root Cause:** Emoji characters in print() statements, Windows cp1252 encoding limitation
- **Fix:** Remove emoji print() statements, use logger instead

### Lessons Learned

1. **Configuration Architecture:**
   - ONLY `.claude.json` is read by Claude Code
   - `mcp_servers.json` is a red herring (not used)
   - Always use `claude mcp` CLI commands for config management

2. **Environment Variables:**
   - Expanded at configuration time, NOT runtime
   - Stored as literal values in `.claude.json`
   - API key rotation requires re-running `claude mcp add`

3. **MCP stdio Protocol:**
   - stdout is reserved for JSON-RPC communication
   - NEVER use print() in stdio MCP servers
   - Use logger (stderr) for all diagnostic output

4. **Windows Platform Specifics:**
   - Path escaping requires double-backslashes (`C:\\\\...`)
   - cp1252 encoding cannot handle Unicode emojis
   - `claude mcp add-json` more reliable than `claude mcp add`

5. **Debugging Process:**
   - Check `.claude.json` first (not `mcp_servers.json`)
   - Verify scope (user vs local)
   - Review debug logs in `~/.claude/debug/`
   - Test server manually before blaming configuration

### Preventive Measures

**For Future MCP Server Development:**
1. ✅ Use logger instead of print() in stdio servers
2. ✅ Avoid emoji characters in server output
3. ✅ Test on Windows (encoding edge cases)
4. ✅ Document scope requirements clearly
5. ✅ Provide `claude mcp add-json` examples with proper escaping

**For Configuration Management:**
1. ✅ Always use `claude mcp` CLI commands
2. ✅ Verify with `claude mcp get <server>` after adding
3. ✅ Check `~/.claude.json` directly if issues persist
4. ✅ Delete deprecated `mcp_servers.json` to avoid confusion
5. ✅ Document environment variable expansion behavior

---

## claude mcp CLI Commands Reference

**Date:** 2025-10-27
**Purpose:** Quick reference for managing MCP servers via Claude Code CLI

### List All MCP Servers

```bash
claude mcp list
```

**Example Output:**
```
Available MCP servers:
- serena
- claude-context
- context7-local
- graphiti-memory
- gptr-mcp
```

### Get Specific Server Configuration

```bash
claude mcp get <server-name>
```

**Example:**
```bash
claude mcp get gptr-mcp
```

**Example Output:**
```json
{
  "type": "stdio",
  "command": "C:\\python313\\python.exe",
  "args": ["C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py"],
  "env": {
    "OPENAI_API_KEY": "sk-proj-...",
    "TAVILY_API_KEY": "tvly-..."
  }
}
```

### Add MCP Server (Method 1: add-json)

**Recommended for Windows (handles path escaping):**

```bash
claude mcp add-json --scope user <server-name> '<json-config>'
```

**Example:**
```bash
claude mcp add-json --scope user gptr-mcp '{
  "type": "stdio",
  "command": "C:\\\\python313\\\\python.exe",
  "args": ["C:\\\\Users\\\\Admin\\\\Documents\\\\GitHub\\\\gptr-mcp\\\\server.py"],
  "env": {
    "OPENAI_API_KEY": "'${OPENAI_API_KEY}'",
    "TAVILY_API_KEY": "'${TAVILY_API_KEY}'"
  }
}'
```

**Notes:**
- Use `\\\\` for each backslash on Windows
- Environment variables expanded at configuration time
- `--scope user` for global availability

### Add MCP Server (Method 2: add with flags)

**Alternative approach using --env flags:**

```bash
claude mcp add --scope user <server-name> \
  --command <path-to-executable> \
  --arg <arg1> --arg <arg2> \
  --env KEY1=value1 --env KEY2=value2
```

**Example:**
```bash
claude mcp add --scope user gptr-mcp \
  --command C:\\python313\\python.exe \
  --arg C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py \
  --env OPENAI_API_KEY=$OPENAI_API_KEY \
  --env TAVILY_API_KEY=$TAVILY_API_KEY
```

**Warning:** On Windows, path escaping may be inconsistent. Prefer `add-json`.

### Remove MCP Server

```bash
claude mcp remove <server-name>
```

**Example:**
```bash
claude mcp remove gptr-mcp
```

### Scope Flags

| Flag | Scope | Location | Use Case |
|------|-------|----------|----------|
| `--scope user` | Global | `~/.claude.json` root `mcpServers` | General-purpose tools |
| `--scope local` (default) | Project | `~/.claude.json` under project path | Project-specific servers |
| `--scope project` | Project | Same as local | Alias for local |

### Common Workflows

**Add a new global MCP server:**
```bash
# 1. Prepare JSON config
# 2. Add with --scope user
claude mcp add-json --scope user my-server '{...}'

# 3. Verify
claude mcp get my-server

# 4. Check in Claude Code
# Type /mcp in Claude Code to see server list
```

**Update API keys (environment variable rotation):**
```bash
# 1. Update OS environment variables
export OPENAI_API_KEY="new-key"

# 2. Remove old config
claude mcp remove gptr-mcp

# 3. Re-add with new key
claude mcp add-json --scope user gptr-mcp '{...}'
```

**Troubleshoot connection issues:**
```bash
# 1. Check if server is configured
claude mcp list | grep my-server

# 2. Inspect configuration
claude mcp get my-server

# 3. Check debug logs
cat ~/.claude/debug/*.txt | grep my-server
```
