# gptr-mcp - Installation Guide

**Version**: 1.3.0
**Last Updated**: 2025-10-27
**For**: AI-assisted and manual installation

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation Options](#installation-options)
- [Step-by-Step Setup](#step-by-step-setup)
- [Environment Configuration](#environment-configuration)
- [Validation](#validation)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Overview

This guide provides interactive installation instructions for **gptr-mcp** (GPT Researcher MCP Server).

**About This Project**:
- Base Project: GPT Researcher MCP Server
- Base Repository: `https://github.com/assafelovic/gptr-mcp`
- This Repository: `https://github.com/RoscoeTheDog/gptr-mcp`
- Customizations: Personal fork with living documentation system, serves as separation of concerns environment for extended features and bugfixes for personal use
- Related: Part of gpt-researcher ecosystem (also forked for personal use)

**Installation Philosophy**:
- Multiple installation paths supported
- Explicit user choices (AI agents will always prompt, never assume)
- Platform-specific instructions where needed
- Validation status tracked for each option

---

## Prerequisites

### Required

#### 1. Python 3.11+ ⚠️ Needs Validation
- **Reason**: gpt-researcher >=0.14.0 requires Python 3.11 or higher
- **Validation**: ⚠️ Testing in progress on Windows 11
- **Installation**:
  - Windows: Download from [python.org](https://www.python.org/downloads/)
  - macOS: `brew install python@3.11`
  - Linux: `sudo apt install python3.11 python3.11-venv`

**Verify installation**:
```bash
python --version  # Should show 3.11 or higher
```

#### 2. Git ✅ Validated
- **Reason**: Clone repository and manage version control
- **Installation**:
  - Windows: Download from [git-scm.com](https://git-scm.com/)
  - macOS: `brew install git` or use Xcode Command Line Tools
  - Linux: `sudo apt install git`

### API Keys Required

#### OpenAI API Key ⚠️ Needs Validation
- **Reason**: Required for GPT models used in research
- **Obtain From**: [OpenAI Platform](https://platform.openai.com/api-keys)
- **Validation**: ⚠️ Needs validation

#### Tavily API Key ⚠️ Needs Validation
- **Reason**: Required for web search functionality
- **Obtain From**: [Tavily](https://app.tavily.com)
- **Validation**: ⚠️ Needs validation

### Optional

#### Anthropic API Key (Optional) ⚠️ Experimental
- **Reason**: For using Claude models instead of GPT
- **Required For**: Alternative LLM backend
- **Obtain From**: [Anthropic Console](https://console.anthropic.com/)
- **Validation**: ⚠️ Experimental

#### Docker (Optional) ⚠️ Needs Validation
- **Reason**: For containerized deployment
- **Required For**: Docker-based installation, n8n integration
- **Installation**:
  - Windows: [Docker Desktop](https://www.docker.com/products/docker-desktop)
  - macOS: [Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Linux: `sudo apt install docker.io docker-compose`
- **Validation**: ⚠️ Needs validation

---

## Installation Options

You will be prompted to choose between multiple installation paths. Here are the available options:

### Option 1: Direct Python Installation (pip) ⚠️ Needs Validation
- **Best For**: Local development, direct control
- **Pros**: Simple, direct access to code, easy debugging
- **Cons**: Requires manual environment setup
- **Validation**: ⚠️ Needs validation on Windows 11
- **Estimated Time**: 10-15 minutes

### Option 2: Docker / Docker Compose ⚠️ Needs Validation
- **Best For**: Isolated environment, production deployment, n8n integration
- **Pros**: Containerized, reproducible, easy cleanup
- **Cons**: Requires Docker installation, slightly more complex
- **Validation**: ⚠️ Needs validation
- **Estimated Time**: 15-20 minutes

### Option 3: Claude Desktop Integration ⚠️ Needs Validation
- **Best For**: Using MCP server directly with Claude Desktop app
- **Pros**: Seamless integration with Claude Desktop
- **Cons**: Requires Claude Desktop app (macOS/Windows)
- **Validation**: ⚠️ Needs validation
- **Estimated Time**: 10-15 minutes (after base installation)

### Option 3B: Claude Code CLI Integration ✅ Validated
- **Best For**: Using MCP server with Claude Code CLI (command-line interface)
- **Pros**: Integrates with CLI workflow, uses system environment variables, automatic startup
- **Cons**: Requires manual JSON configuration
- **Validation**: ✅ Validated on Windows 11 with Claude Code CLI
- **Estimated Time**: 5-10 minutes

### Option 4: System-Wide Python Installation ✅ Validated
- **Best For**: Global access without Docker, VM environments, system-wide CLI usage
- **Pros**: Accessible from any directory, uses OS environment variables, no containers
- **Cons**: Requires system Python configuration, dependencies installed globally
- **Validation**: ✅ Validated on Windows 11 with Python 3.13
- **Estimated Time**: 5-10 minutes

### Option 5: n8n MCP Integration ⚠️ Needs Validation
- **Best For**: Workflow automation with n8n
- **Pros**: Integrate research capabilities into n8n workflows
- **Cons**: Requires n8n and Docker networking setup
- **Validation**: ⚠️ Needs validation
- **Estimated Time**: 20-30 minutes

---

## Step-by-Step Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/RoscoeTheDog/gptr-mcp
cd gptr-mcp
```

### Step 2: Installation Path Selection

**Prompt**: "Which installation method do you prefer?"

#### If Option 1: Direct Python Installation

##### 2.1 Create Virtual Environment

**Why**: Isolate project dependencies from system Python

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate

# macOS/Linux:
source venv/bin/activate

# Verify activation (should show venv path):
which python  # macOS/Linux
where python  # Windows
```

##### 2.2 Install Dependencies

```bash
pip install -r requirements.txt
```

**Note**: If you encounter installation errors, see [Troubleshooting](#troubleshooting).

##### 2.3 Environment Configuration

See [Environment Configuration](#environment-configuration) section below.

##### 2.4 Run the Server

```bash
# For local development (STDIO transport, Claude Desktop compatible)
python server.py

# Or force SSE transport for web-based clients
export MCP_TRANSPORT=sse  # macOS/Linux
set MCP_TRANSPORT=sse     # Windows cmd
python server.py
```

#### If Option 2: Docker / Docker Compose

##### 2.1 Create .env File

```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your API keys (see Environment Configuration section)
```

##### 2.2 Build and Run with Docker Compose

```bash
# Build and run
docker-compose up -d

# Check status
docker ps | grep gptr-mcp

# View logs
docker logs gptr-mcp

# Stop
docker-compose down
```

##### 2.3 Verify Server is Running

```bash
# Health check
curl http://localhost:8000/health

# Get session ID
curl http://localhost:8000/sse
```

##### 2.4 (Optional) Connect to n8n Network

```bash
# If integrating with existing n8n installation
docker network connect n8n-mcp-net gptr-mcp
```

#### If Option 3: Claude Desktop Integration

**Prerequisites**: Complete Option 1 (Direct Python Installation) first.

##### 3.1 Locate Claude Desktop Config File

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

##### 3.2 Add MCP Server Configuration

```json
{
  "mcpServers": {
    "gptr-mcp": {
      "command": "python",
      "args": ["/absolute/path/to/gptr-mcp/server.py"],
      "env": {
        "OPENAI_API_KEY": "your-actual-openai-key-here",
        "TAVILY_API_KEY": "your-actual-tavily-key-here"
      }
    }
  }
}
```

**Important**:
- Use **absolute paths** (e.g., `C:\\Users\\YourName\\Documents\\gptr-mcp\\server.py` on Windows)
- Include API keys in the `env` section (Claude Desktop doesn't read .env files)

##### 3.3 Security Note

```bash
# macOS: Protect your config file
chmod 600 ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Never commit this file to version control** - it contains sensitive API keys.

##### 3.4 Restart Claude Desktop

Completely quit and restart Claude Desktop to load the new configuration.

##### 3.5 Verify Integration

Look for the 🔧 tools icon in Claude Desktop. You should see tools like:
- `deep_research`
- `quick_search`
- `write_report`
- `get_research_sources`
- `get_research_context`

#### If Option 3B: Claude Code CLI Integration ✅ Validated

**Prerequisites**: System Python 3.11+ installed OR complete Option 4 (System-Wide Python Installation).

##### 3B.1 Add MCP Server Using Claude CLI Command

The recommended way to add MCP servers is using the `claude mcp add` command. This ensures proper configuration and global availability.

**Important**: Use `--scope user` to make the server available globally across all projects.

**Method 1: Using JSON String (Recommended)**

This method properly escapes paths and is the most reliable:

**Windows**:
```bash
claude mcp add-json --scope user gptr-mcp '{"type":"stdio","command":"C:\\\\python313\\\\python.exe","args":["C:\\\\Users\\\\YourUsername\\\\Documents\\\\GitHub\\\\gptr-mcp\\\\server.py"],"env":{"OPENAI_API_KEY":"'${OPENAI_API_KEY}'","TAVILY_API_KEY":"'${TAVILY_API_KEY}'","ANTHROPIC_API_KEY":"'${ANTHROPIC_API_KEY}'"}}'
```

**macOS/Linux**:
```bash
claude mcp add-json --scope user gptr-mcp '{"type":"stdio","command":"/usr/local/bin/python3","args":["/absolute/path/to/gptr-mcp/server.py"],"env":{"OPENAI_API_KEY":"'${OPENAI_API_KEY}'","TAVILY_API_KEY":"'${TAVILY_API_KEY}'","ANTHROPIC_API_KEY":"'${ANTHROPIC_API_KEY}'"}}'
```

**Method 2: Using Standard Command (Alternative)**

**Windows**:
```bash
claude mcp add --scope user --transport stdio gptr-mcp --env OPENAI_API_KEY=%OPENAI_API_KEY% --env TAVILY_API_KEY=%TAVILY_API_KEY% --env ANTHROPIC_API_KEY=%ANTHROPIC_API_KEY% -- C:\python313\python.exe C:\Users\YourUsername\Documents\GitHub\gptr-mcp\server.py
```

**macOS/Linux**:
```bash
claude mcp add --scope user --transport stdio gptr-mcp --env OPENAI_API_KEY=$OPENAI_API_KEY --env TAVILY_API_KEY=$TAVILY_API_KEY --env ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY -- /usr/local/bin/python3 /absolute/path/to/gptr-mcp/server.py
```

**Important Notes**:
- **Adjust paths** to match your Python installation and gptr-mcp location
- The `--scope user` flag makes the server available **globally** across all projects
- Without `--scope user`, the server will only be available in the current project (local scope)
- The `--env` flag **expands** environment variables at configuration time (values are stored, not references)
- If you change your API keys later, you must re-run the `claude mcp add` command

##### 3B.2 Verify Configuration

After running the `claude mcp add` command, verify the server was added:

```bash
# List all MCP servers
claude mcp list

# Get details about gptr-mcp specifically
claude mcp get gptr-mcp
```

**Expected Output**:
```
gptr-mcp:
  Scope: User config (available in all your projects)
  Status: ✓ Connected
  Type: stdio
  Command: C:\python313\python.exe
  Args: C:\Users\YourUsername\Documents\GitHub\gptr-mcp\server.py
  Environment:
    OPENAI_API_KEY=sk-proj-...
    TAVILY_API_KEY=tvly-...
    ANTHROPIC_API_KEY=sk-ant-...
```

##### 3B.3 Restart Claude Code CLI (If Currently Running)

**If Claude Code is already running**, you need to restart it to pick up the new MCP server:

```bash
# Exit current session
exit

# Start new session
claude
```

**If Claude Code was not running**, the server will be automatically loaded when you next start Claude Code.

##### 3B.4 Test the Integration

After Claude Code starts (or restarts), verify the MCP server tools are available:

```bash
# In Claude Code, check MCP servers
/mcp
```

**Test a tool**:
```
Can you do a quick_search for "Python async best practices"?
```

**Expected Behavior**:
- The `quick_search` tool should be available and functional
- Results should be returned from GPT Researcher
- All tools (`deep_research`, `quick_search`, `write_report`, etc.) should appear in `/mcp` list

##### 3B.5 Troubleshooting Claude Code CLI Integration

**Issue: gptr-mcp not appearing in MCP list**

1. **Verify the server was added**:
   ```bash
   claude mcp list
   # Should show gptr-mcp in the list
   ```

2. **Check server scope**:
   ```bash
   claude mcp get gptr-mcp
   # Should show "Scope: User config (available in all your projects)"
   # If it shows "Local config", you need to add with --scope user
   ```

3. **Verify Python path**:
   ```bash
   which python3  # macOS/Linux
   where python   # Windows
   # Ensure the path matches what you used in claude mcp add command
   ```

4. **Test server manually**:
   ```bash
   python /path/to/gptr-mcp/server.py
   # Should start without errors (no output is normal for stdio mode)
   # Press Ctrl+C to stop
   ```

5. **Check environment variables are set**:
   ```bash
   # Windows (CMD)
   echo %OPENAI_API_KEY%

   # macOS/Linux
   echo $OPENAI_API_KEY
   ```

6. **Re-add with correct scope if needed**:
   ```bash
   # Remove incorrect entry
   claude mcp remove gptr-mcp

   # Add again with --scope user
   claude mcp add-json --scope user gptr-mcp '{"type":"stdio",...}'
   ```

**Issue: "Connection failed" or "Failed to connect"**

1. **Check debug logs**:
   ```bash
   tail -f ~/.claude/debug/*.txt | grep gptr-mcp
   ```

2. **Look for specific errors** like:
   - Unicode/encoding errors → See Windows Unicode issue in main Troubleshooting section
   - Module not found → Install dependencies with `pip install -r requirements.txt`
   - Python version error → Ensure Python 3.11+ is installed

3. **Test with absolute paths**:
   - Ensure both Python command and server.py use absolute paths
   - Avoid relative paths like `./server.py`

**Issue: Environment variables not being passed to server**

- **Remember**: `claude mcp add --env VAR=$VAR` expands the variable at configuration time
- The **actual value** is stored in `~/.claude.json`, not the variable reference
- If you change your API keys, you **must re-run** `claude mcp add` to update
- Check stored value: `claude mcp get gptr-mcp` (will show actual key values)

#### If Option 4: System-Wide Python Installation

**Prerequisites**: System Python 3.11+ installed and in PATH.

##### 4.1 Verify System Python

```bash
# Check Python version (must be 3.11+)
python --version

# On some systems, use python3
python3 --version

# Verify pip is available
python -m pip --version
```

##### 4.2 Install Dependencies Globally

```bash
# Navigate to gptr-mcp directory
cd /path/to/gptr-mcp

# Install requirements to system Python
python -m pip install -r requirements.txt

# Or use python3 if python points to Python 2
python3 -m pip install -r requirements.txt
```

##### 4.3 Set Windows Environment Variables

**Option A: Via System Properties (Recommended)**

1. Press `Win + R`, type `sysdm.cpl`, press Enter
2. Navigate to **Advanced** tab → **Environment Variables**
3. Under **User variables** or **System variables**, click **New**
4. Add the following variables:
   - `OPENAI_API_KEY` = `your-openai-api-key`
   - `TAVILY_API_KEY` = `your-tavily-api-key`
   - `ANTHROPIC_API_KEY` = `your-anthropic-api-key` (optional)
5. Click **OK** to save
6. **Restart your terminal** for changes to take effect

**Option B: Via Command Line (Temporary)**

```cmd
REM Set environment variables (current session only)
set OPENAI_API_KEY=your-openai-api-key
set TAVILY_API_KEY=your-tavily-api-key
set ANTHROPIC_API_KEY=your-anthropic-api-key
```

**Option C: Via PowerShell (Persistent)**

```powershell
# Set user environment variables (persistent)
[Environment]::SetEnvironmentVariable("OPENAI_API_KEY", "your-openai-api-key", "User")
[Environment]::SetEnvironmentVariable("TAVILY_API_KEY", "your-tavily-api-key", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "your-anthropic-api-key", "User")
```

##### 4.4 Add Launch Scripts to PATH (Optional)

**Option A: Add gptr-mcp directory to PATH**

1. Press `Win + R`, type `sysdm.cpl`, press Enter
2. Navigate to **Advanced** tab → **Environment Variables**
3. Under **User variables**, find **Path**, click **Edit**
4. Click **New**, add: `C:\Users\YourUsername\Documents\GitHub\gptr-mcp`
5. Click **OK** to save
6. Restart your terminal

Now you can run `gptr-mcp.bat` from anywhere.

**Option B: Copy scripts to a directory in PATH**

```bash
# Example: Copy to a directory already in PATH
copy gptr-mcp.bat C:\Windows\System32\
copy gptr-mcp.sh C:\Windows\System32\
```

##### 4.5 Run gptr-mcp Globally

```bash
# From any directory:
gptr-mcp.bat

# Or with Git Bash / WSL:
gptr-mcp.sh

# Or use absolute path:
C:\Users\Admin\Documents\GitHub\gptr-mcp\gptr-mcp.bat

# Or run directly with Python:
cd /path/to/gptr-mcp
python server.py
```

##### 4.6 Verify Environment Variables

```bash
# Check if variables are set (Git Bash):
echo $OPENAI_API_KEY
echo $TAVILY_API_KEY

# Check if variables are set (CMD):
echo %OPENAI_API_KEY%
echo %TAVILY_API_KEY%

# Check if variables are set (PowerShell):
$env:OPENAI_API_KEY
$env:TAVILY_API_KEY
```

##### 4.7 Test the Server

```bash
# Run the server
python server.py

# Expected output:
# 🚀 GPT Researcher MCP Server starting with stdio transport...
# Server running with STDIO transport (for Claude Desktop)
```

#### If Option 5: n8n Integration

**Prerequisites**: Complete Option 2 (Docker installation) first.

##### 4.1 Ensure Docker Network Connectivity

```bash
# Create shared network if needed
docker network create n8n-mcp-net

# Connect gptr-mcp to network
docker network connect n8n-mcp-net gptr-mcp
```

##### 4.2 Get Session ID

```bash
curl http://gptr-mcp:8000/sse
# Look for: data: /messages/?session_id=XXXXX
```

##### 4.3 Initialize MCP in n8n

```bash
curl -X POST http://gptr-mcp:8000/messages/?session_id=YOUR_SESSION_ID \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {"roots": {"listChanged": true}},
      "clientInfo": {"name": "n8n-client", "version": "1.0.0"}
    }
  }'
```

##### 4.4 Call Tools from n8n

Configure n8n to use the MCP endpoint: `http://gptr-mcp:8000/messages/?session_id=YOUR_SESSION_ID`

---

## Environment Configuration

### Create .env File

```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your preferred text editor
# Windows:
notepad .env

# macOS/Linux:
nano .env
# or
vim .env
```

### Required Environment Variables

```bash
# OpenAI API key for GPT models (required)
OPENAI_API_KEY=your-openai-api-key-here

# Tavily API key for web search (required)
TAVILY_API_KEY=your-tavily-api-key-here

# GPT Researcher configuration
EMBEDDING=openai:text-embedding-3-small
STRATEGIC_LLM=openai:gpt-4o-mini

# Maximum number of research iterations
MAX_ITERATIONS=2

# Server configuration
LOG_LEVEL=INFO
PORT=8000
```

### Optional Environment Variables

```bash
# Optional: Anthropic API key if using Claude models
ANTHROPIC_API_KEY=your-anthropic-api-key-here

# Optional: Force specific transport mode
MCP_TRANSPORT=stdio  # or 'sse' or 'streamable-http'

# Optional: Docker container flag (auto-detected)
DOCKER_CONTAINER=true
```

### Platform-Specific Configuration

#### Windows-Specific
```bash
# Use backslashes for Windows paths (if needed)
# Most configurations work without path customization
```

#### macOS/Linux-Specific
```bash
# Use forward slashes for paths (if needed)
# Most configurations work without path customization
```

---

## Validation

### Verify Installation

**Test the server manually**:

```bash
# Activate virtual environment (if using Option 1)
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows

# Run server
python server.py

# Expected output:
# Server running with STDIO transport (for Claude Desktop)
# Or: Server running with SSE transport at 0.0.0.0:8000 (for Docker)
```

### Test MCP Tools (Docker/SSE mode)

```bash
# Test script provided
python test_mcp_server.py
```

**Expected Output**:
```
✅ SSE connection successful
✅ Session ID obtained
✅ MCP initialized
✅ Tools discovered: deep_research, quick_search, write_report
✅ Test query executed successfully
```

### Manual Component Verification

```bash
# Test Python version
python --version  # Should show 3.11 or higher

# Test imports
python -c "import gpt_researcher; print('✅ gpt-researcher installed')"
python -c "import fastmcp; print('✅ fastmcp installed')"

# Test environment variables
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print('✅ OPENAI_API_KEY:', 'configured' if os.getenv('OPENAI_API_KEY') else 'MISSING')"
```

---

## Troubleshooting

### Common Issues

#### Issue: `ModuleNotFoundError: No module named 'X'`
**Cause**: Dependency not installed or wrong Python environment
**Solution**:
```bash
# Verify virtual environment is activated
which python  # Should show venv path

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

#### Issue: `OPENAI_API_KEY not found`
**Cause**: Environment variables not loaded
**Solution**:
- **Direct Python**: Verify .env file exists and contains `OPENAI_API_KEY=...`
- **Docker**: Check docker-compose.yml has correct `env_file: - .env`
- **Claude Desktop**: Verify API keys are in `claude_desktop_config.json` under `env` section

#### Issue: `TAVILY_API_KEY not found`
**Cause**: Missing Tavily API key
**Solution**:
- Sign up at [Tavily](https://app.tavily.com)
- Add key to .env file or Claude Desktop config

#### Issue: Python Version Error
**Cause**: Python version too old (need 3.11+)
**Solution**:
```bash
# Check version
python --version

# Install Python 3.11+ from python.org or package manager
# Recreate virtual environment with correct version
python3.11 -m venv venv
```

#### Issue: Docker Container Not Accessible
**Cause**: Container not running or port conflict
**Solution**:
```bash
# Check container status
docker ps | grep gptr-mcp

# Check logs
docker logs gptr-mcp

# Restart container
docker-compose restart

# Check port availability
netstat -an | grep 8000  # Windows
lsof -i :8000            # macOS/Linux
```

#### Issue: Claude Desktop Not Showing Tools
**Cause**: Config file error or path issues
**Solution**:
1. Verify `claude_desktop_config.json` is valid JSON
2. Use **absolute paths** (not relative)
3. Check API keys are in `env` section
4. Completely restart Claude Desktop (quit fully, not just close window)
5. Test server manually: `python server.py` (should work without errors)

#### Issue: n8n Cannot Connect
**Cause**: Docker networking issue
**Solution**:
```bash
# Verify both containers on same network
docker network inspect n8n-mcp-net

# Reconnect if needed
docker network connect n8n-mcp-net gptr-mcp

# Use container name as hostname in n8n: gptr-mcp
```

#### Issue: MCP Server Fails to Connect (Windows Unicode Error)
**Cause**: Windows console (cp1252 encoding) cannot handle Unicode characters in server output
**Error**: `UnicodeEncodeError: 'charmap' codec can't decode byte`
**Solution**:
- This issue has been fixed in the current version of server.py
- If using an older version, update to the latest code from the repository
- The fix removes emoji characters from print statements that were incompatible with Windows console encoding
- **Technical details**: MCP stdio servers cannot use `print()` to stdout as it interferes with JSON-RPC protocol

#### Issue: Claude Code CLI - Server Not Appearing in /mcp List
**Cause**: Server added to project scope instead of user (global) scope
**Solution**:
```bash
# Remove project-scoped server if it exists
claude mcp remove gptr-mcp -s local

# Add to user scope for global availability
claude mcp add-json --scope user gptr-mcp '{"type":"stdio","command":"C:\\\\python313\\\\python.exe","args":["C:\\\\Users\\\\YourUsername\\\\Documents\\\\GitHub\\\\gptr-mcp\\\\server.py"],"env":{"OPENAI_API_KEY":"your-key","TAVILY_API_KEY":"your-key"}}'

# Verify connection
claude mcp list
```

### Platform-Specific Issues

#### Windows
- **Issue**: `python` command not found
  - **Solution**: Use `py` instead or add Python to PATH
- **Issue**: PowerShell execution policy
  - **Solution**: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
- **Issue**: Unicode/emoji errors in console output
  - **Solution**: Update to latest server.py version (fixed in v1.2.1+)

#### macOS
- **Issue**: SSL certificate errors
  - **Solution**: `pip install --upgrade certifi`
- **Issue**: Command line tools missing
  - **Solution**: `xcode-select --install`

#### Linux
- **Issue**: Missing build tools for dependencies
  - **Solution**: `sudo apt install build-essential python3-dev`

---

## Next Steps

After successful installation:

1. **Read Original Documentation**: See [GPT Researcher MCP Docs](https://docs.gptr.dev/docs/gpt-researcher/mcp-server/getting-started) for usage guides
2. **Test Tools**: Try `deep_research` or `quick_search` with a test query
3. **Explore Examples**: Check the base project for example usage patterns
4. **Join Community**: [GPT Researcher Discord](https://discord.gg/QgZXvJAccX) for support

### For Developers

If you're developing on this fork:
- Read `CLAUDE_DEV.md` for development policies and living documentation system
- Set up git hooks if needed
- Follow the living documentation workflow when making changes

---

## Changelog

See [CLAUDE_INSTALL_CHANGELOG.md](./CLAUDE_INSTALL_CHANGELOG.md) for version history and changes.

---

## Support

- **This Fork Issues**: `https://github.com/RoscoeTheDog/gptr-mcp/issues`
- **Base Project**: `https://github.com/assafelovic/gptr-mcp` (upstream)
- **GPT Researcher Main**: `https://github.com/assafelovic/gpt-researcher`
- **Community**: [Discord](https://discord.gg/QgZXvJAccX)

---

**Template Version**: 1.0.0
**Installation Guide Version**: 1.0.0
