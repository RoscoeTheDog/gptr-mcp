# gptr-mcp - Installation Guide

**Version**: 1.0.0 | **Updated**: 2025-10-26 | **For**: AI-assisted + manual install

---

## TOC

[Overview](#overview) | [Prerequisites](#prerequisites) | [Options](#installation-options) | [MCP Integration](#mcp-server-integration) | [Setup](#step-by-step-setup) | [Environment](#environment-configuration) | [Validation](#validation) | [Troubleshooting](#troubleshooting) | [Support](#support)

---

## Overview

**About**:
- Base: GPT Researcher MCP Server
- Base Repo: `https://github.com/assafelovic/gptr-mcp.git`
- This Repo: `https://github.com/RoscoeTheDog/gptr-mcp.git`
- Custom: Added custom tools, modified configuration, enhanced documentation

**Philosophy**: Multi-path install, explicit choices (AI always prompts), platform-specific, validation-tracked

---

## Prerequisites

### Required

#### Python 3.11+ ✅W11
- **Reason**: gptr-mcp requires Python 3.11 or higher (gpt-researcher >=0.12.16 requirement)
- **Validation**: ✅W11 [2025-10-26]
- **Install**:
  - W: [python.org](https://python.org/downloads/)
  - M: `brew install python@3.11`
  - L: `sudo apt install python3.11 python3.11-venv`
- **Verify**: `python --version` (should show 3.11+)

#### Git ✅WML
- **Install**:
  - W: [git-scm.com](https://git-scm.com/)
  - M: `brew install git` | Xcode CLI Tools
  - L: `sudo apt install git`

### Optional

#### Anthropic API Key ⚠️
- **Reason**: For using Claude models instead of OpenAI
- **Required-For**: Claude-based research and report generation
- **Validation**: ⚠️ Experimental (optional feature)

---

## Installation Options

### Standard Installation (pip)

**Recommended**: Simple pip-based installation

#### Python Virtual Environment ✅W11
- **Best**: Isolated dependencies, clean setup
- **Pros**: No conflicts with system packages, easy cleanup
- **Cons**: Need to activate venv for each session
- **Time**: 5-10min
- **Validation**: ✅W11 [2025-10-26]

**Steps**:
```bash
# Clone repository
git clone https://github.com/RoscoeTheDog/gptr-mcp.git
cd gptr-mcp

# Create virtual environment
python -m venv venv

# Activate (Windows)
.\venv\Scripts\activate

# Activate (macOS/Linux)
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

---

## MCP Server Integration

> **Note**: This section applies only if this project is an MCP server. Skip if not applicable.

**Purpose**: Configure this MCP server for global availability in Claude Code CLI

**When to Use**: After completing base installation, use this to register the MCP server with Claude Code CLI for use across all projects.

### Prerequisites

- ✅ Claude Code CLI installed (`claude --version`)
- ✅ Base project dependencies installed (see [Step-by-Step Setup](#step-by-step-setup))
- ✅ Required credentials available (see below)

### Installation Wizard

**Agent will automatically check for existing installations** before proceeding.

#### Phase 1: Pre-Installation Check

**Automatic Detection**:
```bash
# Agent runs:
claude mcp list | grep gptr-mcp
```

**Possible States**:

| State | Description | Action |
|-------|-------------|--------|
| **Not Installed** | Server not found in list | ✅ Proceed with fresh installation |
| **User Scope** | Already installed globally | ⚙️  Show Update/Repair/Reinstall/Cancel wizard |
| **Project Scope** | Installed locally only | 🔄 Prompt to migrate to global scope |
| **Broken** | In list but disconnected | 🔧 Prompt to repair installation |

**If Already Installed**:
```
┌─────────────────────────────────────────────┐
│ MCP Server Installation Wizard             │
├─────────────────────────────────────────────┤
│                                             │
│ ✓ Found: gptr-mcp (User scope)             │
│   Status: Connected                         │
│   Command: C:\python313\python.exe          │
│   Entrypoint: C:\Users\Admin\Documents\     │
│               GitHub\gptr-mcp\server.py     │
│                                             │
│ What would you like to do?                  │
│                                             │
│ [1] Update Configuration                    │
│     → Modify credentials or paths           │
│                                             │
│ [2] Repair Installation                     │
│     → Verify files, test connection, fix    │
│                                             │
│ [3] Reinstall Completely                    │
│     → Remove and fresh install              │
│                                             │
│ [4] Cancel                                  │
│     → Keep current configuration            │
│                                             │
│ Choice [1-4]:                               │
└─────────────────────────────────────────────┘
```

**Workflow by Choice**:

- **[1] Update** → Load current config, prompt for credential changes, update & verify
- **[2] Repair** → Verify paths exist, test connection, diagnose & fix issues
- **[3] Reinstall** → Remove existing, proceed to fresh installation
- **[4] Cancel** → Exit, keep current configuration unchanged

#### Phase 2: Credential Discovery (Fresh Install or Reinstall)

**Agent will detect required credentials from**:
1. README.md sections (Prerequisites, Configuration, Environment)
2. .env.example file (credential template)
3. Code search (environment variable patterns across all languages)

**Detected Credentials**:
```
Required:
- OPENAI_API_KEY: OpenAI API key for GPT models
- TAVILY_API_KEY: Tavily API key for web search

Optional:
- ANTHROPIC_API_KEY: Anthropic API key for Claude models (optional)
- EMBEDDING: Embedding model configuration (default: openai:text-embedding-3-small)
- STRATEGIC_LLM: LLM for strategic research (default: openai:gpt-4o-mini)
- MAX_ITERATIONS: Maximum research iterations (default: 2)
- LOG_LEVEL: Server logging level (default: INFO)
- PORT: Server port (default: 8000)
```

**System Environment Check**:
```bash
# Agent checks system environment for each credential

# Windows (CMD)
echo %CREDENTIAL%

# Windows (PowerShell)
$env:CREDENTIAL

# macOS/Linux
echo $CREDENTIAL
```

**Results**:
```
✓ OPENAI_API_KEY: Found in system environment
✗ TAVILY_API_KEY: Not found in system environment
✗ ANTHROPIC_API_KEY: Not found (optional)
```

**Credential Elicitation**:

For each **not found** credential:
```
Please provide [CREDENTIAL_NAME]:
[Masked input: ●●●●●●●●●●●●]

✓ Received: sk-****...****Qhm
```

For each **found** credential:
```
Use existing system value for [CREDENTIAL_NAME]? [Y/N]
→ Y: Use system value (don't show or re-enter)
→ N: Enter new value: [masked input]
```

**Optional: Add to System Environment**:
```
Would you like to add credentials to system environment variables? [Y/N]

→ Y: Agent provides platform-specific instructions
→ N: Skip (credentials stored only in ~/.claude.json)
```

**Platform-Specific Instructions** (if user wants to add to system):

<details>
<summary><strong>Windows (PowerShell - Recommended)</strong></summary>

```powershell
# Set persistent user environment variable
[Environment]::SetEnvironmentVariable("[CREDENTIAL]", "value", "User")

# Restart terminal to apply changes
exit
```
</details>

<details>
<summary><strong>Windows (GUI Method)</strong></summary>

1. Press `Win+R`, type: `sysdm.cpl`, press Enter
2. Navigate to **Advanced** tab → **Environment Variables**
3. Under **User variables**, click **New**
4. Variable name: `[CREDENTIAL]`
5. Variable value: `[value]`
6. Click **OK** → **OK** → Restart terminal
</details>

<details>
<summary><strong>macOS/Linux (Bash)</strong></summary>

```bash
# Add to ~/.bashrc or ~/.bash_profile
echo 'export [CREDENTIAL]="value"' >> ~/.bashrc
source ~/.bashrc
```
</details>

<details>
<summary><strong>macOS/Linux (Zsh)</strong></summary>

```zsh
# Add to ~/.zshrc
echo 'export [CREDENTIAL]="value"' >> ~/.zshrc
source ~/.zshrc
```
</details>

**⚠️  Important Note About Environment Variables**:
```
Even if credentials are in system environment variables,
Claude Code CLI stores LITERAL VALUES in ~/.claude.json
(not variable references like ${VAR} or %VAR%).

This means:
- System env vars are READ ONCE at configuration time
- Actual values are STORED in ~/.claude.json
- If you change env vars later, you MUST re-run installation

Security Note:
- ~/.claude.json contains sensitive data
- Never commit ~/.claude.json to git
- Protect with file permissions: chmod 600 ~/.claude.json (Unix)
```

#### Phase 3: Configure MCP Server

**Auto-Generated Configuration**:

Agent automatically detects:
- **Platform**: Windows | macOS | Linux
- **Runtime Path**: Absolute path to Python/Node/Binary
- **Entrypoint Path**: Absolute path to server file
- **Server Name**: From README, directory name, or package metadata

**Command Generated**:
```bash
claude mcp add-json --scope user gptr-mcp '{
  "type":"stdio",
  "command":"C:\\python313\\python.exe",
  "args":["C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py"],
  "env":{
    "OPENAI_API_KEY":"[your-openai-key]",
    "TAVILY_API_KEY":"[your-tavily-key]",
    "ANTHROPIC_API_KEY":"[your-anthropic-key-optional]"
  }
}'
```

**Example (Python-based server on Windows)**:
```bash
claude mcp add-json --scope user gptr-mcp '{
  "type":"stdio",
  "command":"C:\\python313\\python.exe",
  "args":["C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py"],
  "env":{
    "OPENAI_API_KEY":"sk-proj-...",
    "TAVILY_API_KEY":"tvly-..."
  }
}'
```

**Important Configuration Notes**:
- ✅ **Always use `--scope user`** for global availability across all projects
- ✅ **Use absolute paths** for both command and entrypoint (no relative paths)
- ⚠️  **Credentials are stored literally** in `~/.claude.json` (not as env var references)
- ⚠️  **Cannot use `${VAR}` or `%VAR%`** syntax - values are expanded at config time
- 🔒 **Security**: `~/.claude.json` contains plaintext credentials - never commit to git

#### Phase 4: Verification

**Verify Installation**:
```bash
# List all MCP servers
claude mcp list

# Should show:
gptr-mcp: ✓ Connected (User scope)

# Get detailed configuration
claude mcp get gptr-mcp

# Should show:
gptr-mcp:
  Scope: User config (available in all your projects)
  Status: ✓ Connected
  Type: stdio
  Command: C:\python313\python.exe
  Args: C:\Users\Admin\Documents\GitHub\gptr-mcp\server.py
  Environment:
    OPENAI_API_KEY: sk-****...****
    TAVILY_API_KEY: tvly-****...****
    ANTHROPIC_API_KEY: sk-ant-****...****
```

**Test in Claude Code CLI**:
```bash
# Restart Claude Code CLI (if currently running)
exit
claude

# Check MCP servers
/mcp

# Should list gptr-mcp with available tools
```

#### Phase 5: Post-Installation

**For Fresh Installation**:
```
✅ Installation complete!

Summary:
- Server: gptr-mcp
- Scope: User (global)
- Status: Connected
- Credentials configured: OPENAI_API_KEY, TAVILY_API_KEY, ANTHROPIC_API_KEY

Next steps:
1. ✅ Restart Claude Code CLI (if running)
2. ✅ Run '/mcp' to see available MCP servers
3. ✅ Test tools: deep_research, quick_search, write_report

Available tools:
- deep_research: Performs deep web research on a topic (30s-5min, comprehensive)
- quick_search: Fast web search optimized for speed (5-30s, quick results)
- write_report: Generate research reports from collected data
- get_research_sources: Retrieve sources used in research
- get_research_context: Get full context of completed research

Available prompts:
- research_query: Create structured research query prompts
```

**For Update**:
```
✅ Configuration updated!

Changes:
- OPENAI_API_KEY: Updated
- TAVILY_API_KEY: Unchanged
- ANTHROPIC_API_KEY: Updated

Restart Claude Code CLI to apply changes:
> exit
> claude
```

**For Repair**:
```
✅ Installation repaired!

Issues fixed:
- Runtime path verified: C:\python313\python.exe exists
- Entrypoint verified: C:\Users\Admin\Documents\GitHub\gptr-mcp\server.py exists
- Credentials validated: All API keys confirmed

Status: Connected
```

### Troubleshooting

#### Issue: Server not appearing in `/mcp` list

**Symptoms**: `/mcp` command doesn't show gptr-mcp

**Diagnosis**:
```bash
# Check if server is registered
claude mcp list | grep gptr-mcp

# If found, check details
claude mcp get gptr-mcp
```

**Solutions**:
1. **Not in list at all** → Run installation again
2. **Project scope (local)** → Migrate to user scope:
   ```bash
   claude mcp remove gptr-mcp -s local
   # Then re-run installation with --scope user
   ```
3. **Status: Disconnected** → See "Connection failed" issue below

#### Issue: Connection failed / Status: Disconnected

**Symptoms**: Server appears in list but shows "Disconnected" or "Failed to connect"

**Diagnosis**:
```bash
# Verify runtime exists
C:\python313\python.exe --version

# Example (Python):
python --version
C:\python313\python.exe --version

# Verify entrypoint exists
ls C:\Users\Admin\Documents\GitHub\gptr-mcp\server.py

# Check debug logs
tail -f ~/.claude/debug/*.txt | grep gptr-mcp

# Test server manually
C:\python313\python.exe C:\Users\Admin\Documents\GitHub\gptr-mcp\server.py
# Should start without errors (Ctrl+C to stop)
```

**Common Causes & Fixes**:

1. **Missing dependencies**:
   ```
   Error: ModuleNotFoundError: No module named 'X'

   Fix: Install dependencies
   pip install -r requirements.txt
   ```

2. **Wrong runtime path**:
   ```
   Error: Command not found / File not found

   Fix: Update configuration with correct path
   - Option 1: Re-run installation (choose "Update")
   - Option 2: Manual update (see "Updating Credentials" below)
   ```

3. **Moved project directory**:
   ```
   Error: Entrypoint file not found

   Fix: Update entrypoint path
   - Re-run installation from new project location
   ```

4. **Invalid credentials**:
   ```
   Error: Authentication failed / API key invalid

   Fix: Update credentials
   - Re-run installation
   - Choose "Update Configuration"
   - Provide new credential values
   ```

#### Issue: Credential errors

**Symptoms**: Server connects but tools fail with authentication errors

**Verification**:
```bash
# Check stored credentials (masked)
claude mcp get gptr-mcp

# Shows:
Environment:
  OPENAI_API_KEY: sk-****...****
  TAVILY_API_KEY: tvly-****...****
  ANTHROPIC_API_KEY: sk-ant-****...****
```

**Solution**:
```
# Re-run installation to update credentials
# Agent will detect existing installation
# Choose [1] Update Configuration
# Select credentials to update
# Provide new values
```

#### Issue: Changes not taking effect

**Symptoms**: Updated configuration but still using old values

**Solution**:
```bash
# Claude Code CLI must be restarted to reload config
exit

# Start new session
claude

# Verify changes applied
claude mcp get [MCP_SERVER_NAME]
```

### Updating Credentials Later

**If credentials change** (API key rotated, new service, etc.):

**Method 1: Re-run Installation (Recommended)**:
```
1. Navigate to project directory
2. Tell agent: "Read INSTALL.md and update MCP server configuration"
3. Agent detects existing installation
4. Choose [1] Update Configuration
5. For each credential: "Update? [Y/N]"
   → Y: Enter new value
   → N: Keep current
6. Agent reconfigures automatically
7. Restart Claude Code CLI
```

**Method 2: Manual Update**:
```bash
# Step 1: Remove existing configuration
claude mcp remove gptr-mcp

# Step 2: Re-add with new credentials
claude mcp add-json --scope user gptr-mcp '{
  "type":"stdio",
  "command":"C:\\python313\\python.exe",
  "args":["C:\\Users\\Admin\\Documents\\GitHub\\gptr-mcp\\server.py"],
  "env":{
    "OPENAI_API_KEY":"[new-value]",
    "TAVILY_API_KEY":"[new-value]",
    "ANTHROPIC_API_KEY":"[new-value-optional]"
  }
}'

# Step 3: Verify
claude mcp get gptr-mcp

# Step 4: Restart Claude Code CLI
exit
claude
```

### Security Best Practices

1. **Protect ~/.claude.json**:
   ```bash
   # Unix/macOS
   chmod 600 ~/.claude.json

   # Windows (PowerShell)
   icacls $env:USERPROFILE\.claude.json /inheritance:r /grant:r "$env:USERNAME:(F)"
   ```

2. **Never commit credentials**:
   - Add to .gitignore: `~/.claude.json` (if accidentally copied)
   - Use `.env` files for local development
   - Keep API keys in environment variables when possible

3. **Rotate credentials regularly**:
   - Update MCP server config when rotating keys
   - Test after rotation to ensure functionality

4. **Use least-privilege credentials**:
   - API keys should have minimal required permissions
   - Create service-specific keys when possible

---

## Step-by-Step Setup

### S1: Clone

```bash
git clone https://github.com/RoscoeTheDog/gptr-mcp.git
cd gptr-mcp
```

### S2: Virtual Environment

```bash
# Create
python -m venv venv

# Activate
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS/Linux

# Verify
where python                 # Windows
which python                 # macOS/Linux
```

### S3: Dependencies

```bash
pip install -r requirements.txt
```

### S4: API Keys Setup

Configure required API keys for web search and LLM operations.

#### OpenAI API Key (Required) ✅W11
1. Visit [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Sign up/login to OpenAI account
3. Click "Create new secret key"
4. Copy key (starts with `sk-proj-` or `sk-`)
5. Store securely (can't view again)

#### Tavily API Key (Required) ✅W11
1. Visit [https://app.tavily.com](https://app.tavily.com)
2. Sign up/login to Tavily account
3. Navigate to API keys section
4. Create new API key
5. Copy key (starts with `tvly-`)

#### Anthropic API Key (Optional) ⚠️
- **For**: Using Claude models instead of OpenAI
- **Setup**:
  1. Visit [https://console.anthropic.com](https://console.anthropic.com)
  2. Sign up/login
  3. Create API key
  4. Copy key (starts with `sk-ant-`)

---

## Environment Configuration

### Create .env

```bash
cp .env.example .env

# Edit
notepad .env                # Windows
nano .env                   # macOS/Linux
```

### Required Vars

```bash
# Database
DATABASE_URI=bolt://localhost:7687
DATABASE_USER=neo4j
DATABASE_PASSWORD=your_password

# LLM
[LLM_PROVIDER]_API_KEY=sk-your-key

# Optional
[OPTIONAL_SERVICE]_API_KEY=key  # Only if using [FEATURE]
```

### Platform-Specific

**W**: `DATA_PATH=C:\\Users\\Name\\Documents\\[PROJECT]\\data` (backslashes)
**M/L**: `DATA_PATH=/Users/Name/Documents/[PROJECT]/data` (forward-slashes)

---

## Validation

### Auto-Validation

```bash
python scripts/validate_install.py
```

**Expected**:
```
✅ Python: [PYTHON_VERSION]
✅ Dependencies: OK
✅ Database: OK
✅ API: [LLM_PROVIDER]
⚠️  Optional: [SERVICE] not configured (OK)
```

### Manual

```bash
# Database
python -c "from [MODULE] import db; db.test_connection()"

# LLM
python -c "from [MODULE] import llm; llm.test_api()"
```

---

## Troubleshooting

### Error→Fix Matrix

| Error | Cause | Solution |
|-------|-------|----------|
| `ModuleNotFoundError: X` | Dep not installed | Verify venv active (`which python`), reinstall: `pip install -r requirements.txt --force-reinstall` |
| DB connection failed | DB not running / wrong creds | Cloud: verify URI/user/pass in .env<br>Local: Check service (`sc query [DB]` / `brew services list` / `systemctl status [DB]`)<br>Docker: `docker ps` |
| API key invalid | Wrong key / not activated | Verify in provider dashboard, check .env for spaces, verify permissions |
| Permission denied (W) | PowerShell execution policy | `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| `python` not found (W) | Not in PATH | Use `py` instead or add Python to PATH |
| SSL cert errors (M) | Cert issues | `pip install --upgrade certifi` |
| Missing build tools (L) | No compiler | `sudo apt install build-essential python3-dev` |

---

## Next Steps

After install:
1. **Docs**: https://docs.gptr.dev/docs/gpt-researcher/mcp-server/getting-started
2. **Official Website**: https://gptr.dev
3. **Community**: https://discord.gg/QgZXvJAccX
4. **Base Project**: https://github.com/assafelovic/gpt-researcher

### For Devs

- Read `CLAUDE_README.md` (dev policies)
- Setup hooks: `pre-commit install`
- Run tests: `pytest tests/`

---

## Changelog

See [CLAUDE_INSTALL_CHANGELOG.md](./CLAUDE_INSTALL_CHANGELOG.md)

---

## Support

- Issues: https://github.com/RoscoeTheDog/gptr-mcp/issues
- Discussions: https://github.com/RoscoeTheDog/gptr-mcp/discussions
- Base Project: https://github.com/assafelovic/gptr-mcp

---

**Template**: v1.1.0 | **Guide**: v1.0.0
