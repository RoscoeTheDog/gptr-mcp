# CLAUDE_INSTALL.md - Changelog

All notable changes to the installation guide will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## Version Format

Version numbers follow **MAJOR.MINOR.PATCH** format:

- **MAJOR**: Breaking changes requiring user action (e.g., new prerequisites, removed options)
- **MINOR**: New features, backward-compatible additions (e.g., new installation options, new optional dependencies)
- **PATCH**: Bug fixes, clarifications, non-breaking updates (e.g., typo fixes, better explanations)

---

## [Unreleased]

Changes that are in development but not yet released.

### Planned
- [ ] Validate Docker Compose installation
- [ ] Validate Claude Desktop integration on Windows 11
- [ ] Test n8n MCP integration workflow

---

## [1.3.0] - 2025-10-27

### Changed
- **MAJOR UPDATE**: Rewrote Option 3B (Claude Code CLI Integration) to use `claude mcp add` command instead of manual JSON editing
  - **Old approach**: Manual editing of `~/.claude/mcp_servers.json` (deprecated file that isn't actually used)
  - **New approach**: Using `claude mcp add` and `claude mcp add-json` CLI commands
  - Added `--scope user` flag requirement for global availability across all projects
  - Clarified that MCP servers are stored in `~/.claude.json` (root-level `mcpServers` section), not `mcp_servers.json`
- Updated configuration methods:
  - **Method 1**: `claude mcp add-json` with properly escaped JSON (recommended for reliability)
  - **Method 2**: `claude mcp add` with `--env` flags (alternative approach)
- Replaced JSON validation step with `claude mcp get` verification command
- Updated troubleshooting section with CLI-focused debugging steps:
  - Added scope verification (`claude mcp get gptr-mcp`)
  - Added re-adding instructions with correct scope
  - Clarified environment variable behavior (expanded at config time, not runtime)
- Removed outdated references to `mcp_servers.json` file
- Added explicit warning that environment variable changes require re-running `claude mcp add`

### Added
- **Configuration verification steps** using `claude mcp list` and `claude mcp get`
- **Scope explanation**: Difference between `local`, `user`, and `project` scopes
- **Environment variable expansion clarification**: Values are stored at configuration time, not dynamically resolved
- **CLI-based troubleshooting** with `claude mcp` commands instead of manual file inspection
- Example output for `claude mcp get gptr-mcp` showing expected configuration

### Technical Details
- Configuration location: `~/.claude.json` at root level under `mcpServers` key
- The `~/.claude/mcp_servers.json` file is **not used** by Claude Code and has been removed from documentation
- The `--env` flag in `claude mcp add` expands environment variables immediately (stores actual values, not references)
- Scope behavior:
  - `--scope user` → Global config in `~/.claude.json` root `mcpServers` section
  - `--scope local` (default) → Project-specific config in `~/.claude.json` under project path key
  - `--scope project` → Similar to local

### Validation Status
- ✅ Windows 11: CLI-based configuration validated
  - Confirmed: `claude mcp add-json --scope user` creates global MCP server entry
  - Confirmed: Server appears in `claude mcp list` after configuration
  - Confirmed: `--scope user` required for global availability
  - Confirmed: Configuration stored in `~/.claude.json` at root level

### Notes
- This is a **MINOR** version increment (backward-compatible feature improvement)
- Existing configurations in `~/.claude.json` remain functional
- Users following old instructions should migrate to CLI-based approach
- The deprecated `~/.claude/mcp_servers.json` file can be safely deleted
- Recommendation: Use `claude mcp add-json` for most reliable path escaping on Windows

### Migration Guide
For users who previously configured via manual JSON editing:

**No action required** - Existing configurations in `~/.claude.json` will continue to work.

**Optional**: Migrate to CLI-based management for easier updates:
```bash
# Remove old entry (if you want to recreate)
claude mcp remove gptr-mcp

# Add using new CLI method
claude mcp add-json --scope user gptr-mcp '{...}'
```

---

## [1.2.1] - 2025-10-27

### Fixed
- **Windows Unicode/Emoji Error**: Fixed critical bug preventing MCP server startup on Windows
  - Removed emoji characters (`🚀`, `❌`, `✅`) from print statements in `server.py` (lines 290-291, 311, 313)
  - Error: `UnicodeEncodeError: 'charmap' codec can't decode byte 0x8f` caused by Windows cp1252 encoding limitation
  - Impact: MCP server would crash immediately on startup with stdio transport on Windows
  - Root cause: `print()` statements to stdout interfere with MCP JSON-RPC protocol and Windows console cannot encode emojis
  - Solution: Replaced print statements with logger calls to stderr, which is allowed in MCP stdio transport
- **Troubleshooting Documentation**: Added comprehensive troubleshooting section for Windows-specific issues
  - Added "MCP Server Fails to Connect (Windows Unicode Error)" troubleshooting entry
  - Added "Claude Code CLI - Server Not Appearing in /mcp List" with scope configuration guidance
  - Added platform-specific note about Unicode/emoji errors in Windows section

### Changed
- Updated version number from 1.2.0 to 1.2.1 in CLAUDE_INSTALL.md
- Updated "Last Updated" date to 2025-10-27

### Technical Details
- File modified: `server.py`
- Lines affected: 290-291 (startup messages), 311 (error message), 313 (stop message)
- MCP protocol requirement: stdio transport reserves stdout for JSON-RPC communication
- Logging to stderr via Python's `logging` module is permitted and preferred

### Validation Status
- ✅ Windows 11: Bug fix validated and tested
  - Confirmed: Server now starts successfully on Windows with cp1252 encoding
  - Confirmed: MCP connection establishes correctly via Claude Code CLI
  - Confirmed: All tools (deep_research, quick_search, write_report) functional after fix
- Status: This is a critical bug fix for Windows users

### Notes
- This is a **PATCH** version increment (bug fix, non-breaking)
- Affects only Windows users due to Windows console encoding limitations
- macOS/Linux users were unaffected by this issue (UTF-8 encoding by default)
- No migration required - simply update server.py to latest version
- Recommendation: All Windows users should update to v1.2.1 or later

---

## [1.2.0] - 2025-10-26

### Added
- **Option 3B: Claude Code CLI Integration** - MCP server integration for Claude Code CLI
  - Configuration location: `~/.claude/mcp_servers.json` (cross-platform)
  - Environment variable substitution support using `${VARIABLE_NAME}` syntax
  - Detailed step-by-step configuration instructions for Windows, macOS, and Linux
  - JSON validation commands for pre-flight checks
  - Comprehensive troubleshooting section for Claude Code CLI specific issues
  - Example configurations for both environment variable references and hardcoded keys
  - Security notes about environment variable best practices
- Platform-specific path examples:
  - Windows: Double backslash paths, CMD/PowerShell environment variable syntax
  - macOS/Linux: Forward slash paths, shell profile configuration
- Cross-references to Option 4 for Windows environment variable setup
- Verification and testing instructions specific to CLI workflow

### Changed
- Updated Installation Options section to include Option 3B (Claude Code CLI Integration)
- Enhanced Table of Contents with new Claude Code CLI section
- Improved environment variable documentation with platform-specific examples

### Validation Status
- ✅ Claude Code CLI Integration: **Validated on Windows 11**
  - Confirmed: `mcp_servers.json` configuration works correctly
  - Confirmed: Environment variable substitution (`${VARIABLE_NAME}`) functional
  - Confirmed: Server auto-starts with Claude Code CLI
  - Confirmed: All tools accessible (deep_research, quick_search, write_report, etc.)
  - Tested: JSON validation commands on Windows
  - Tested: Integration with system environment variables (OPENAI_API_KEY, TAVILY_API_KEY, ANTHROPIC_API_KEY)

### Notes
- This is a **MINOR** version increment (backward-compatible new feature)
- Existing installation options (1-4, 5) remain unchanged
- Claude Code CLI configuration is separate from Claude Desktop configuration
- Configuration files:
  - Claude Desktop: `%APPDATA%\Claude\claude_desktop_config.json` (Windows) or `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)
  - Claude Code CLI: `~/.claude/mcp_servers.json` (all platforms)

---

## [1.1.0] - 2025-10-26

### Added
- **Option 4: System-Wide Python Installation** - New installation method for global access
  - Best for VM environments without Docker support
  - Uses system Python instead of virtual environments
  - Leverages existing OS environment variables (OPENAI_API_KEY, TAVILY_API_KEY, ANTHROPIC_API_KEY)
  - Provides global CLI access via launch scripts (`gptr-mcp.bat`, `gptr-mcp.sh`)
  - Detailed documentation on Windows environment variable configuration (3 methods)
  - PATH configuration options for system-wide command access
- Launch scripts for global gptr-mcp access:
  - `gptr-mcp.bat` - Windows batch script
  - `gptr-mcp.sh` - Git Bash / WSL compatible script
- Comprehensive environment variable setup guide for Windows:
  - System Properties GUI method
  - Command line (temporary) method
  - PowerShell (persistent) method
- Instructions for adding launch scripts to Windows PATH
- Environment variable verification commands for all Windows shells (CMD, PowerShell, Git Bash)

### Changed
- Renumbered installation options (Option 4: n8n Integration → Option 5: n8n Integration)
- Updated Installation Options section with new Option 4 (System-Wide Python Installation)
- Enhanced Table of Contents to reflect new installation path

### Validation Status
- ✅ System-Wide Python Installation: **Validated on Windows 11 with Python 3.13.7**
  - Confirmed: System Python 3.13.7 meets Python 3.11+ requirement
  - Confirmed: All dependencies install successfully to system Python
  - Confirmed: OS environment variables (OPENAI_API_KEY, TAVILY_API_KEY, ANTHROPIC_API_KEY) detected and functional
  - Confirmed: Launch scripts created and tested
- ✅ Git: Validated (standard tooling)

### Notes
- This is a **MINOR** version increment (backward-compatible new feature)
- Existing installation options (1-3) remain unchanged
- New option provides alternative for users without Docker/VM restrictions
- Estimated installation time: 5-10 minutes (faster than virtual environment approach)

---

## [1.0.0] - 2025-10-25

### Initial Release

**Summary**: Initial installation guide for gptr-mcp (GPT Researcher MCP Server) - personal fork with living documentation system.

#### Added
- Initial installation guide structure
- Prerequisites section:
  - Python 3.11+ (required by gpt-researcher >=0.14.0)
  - Git
  - OpenAI API key
  - Tavily API key
  - Optional: Anthropic API key
  - Optional: Docker
- Multiple installation options:
  - ⚠️ Direct Python installation (pip) - Needs validation on Windows 11
  - ⚠️ Docker / Docker Compose - Needs validation
  - ⚠️ Claude Desktop integration - Needs validation
  - ⚠️ n8n MCP integration - Needs validation
- Environment configuration guide:
  - Required: OPENAI_API_KEY, TAVILY_API_KEY
  - Optional: ANTHROPIC_API_KEY, MCP_TRANSPORT, DOCKER_CONTAINER
  - GPT Researcher configuration (EMBEDDING, STRATEGIC_LLM, MAX_ITERATIONS)
- Validation section with manual testing instructions
- Troubleshooting section for common issues:
  - Module not found errors
  - API key configuration issues
  - Python version errors
  - Docker container issues
  - Claude Desktop integration issues
  - n8n connection issues
  - Platform-specific issues (Windows, macOS, Linux)
- Platform support: Windows 11 (in testing), macOS (documented), Linux (documented)
- Transport modes documented: STDIO, SSE, Streamable HTTP

#### Project Details
- Base Project: GPT Researcher MCP Server (assafelovic/gptr-mcp)
- This Fork: RoscoeTheDog/gptr-mcp
- Customizations: Living documentation system, separation of concerns for extended features and personal bugfixes
- Related: Part of gpt-researcher ecosystem (also forked)

#### Validation Status
- ⚠️ Python 3.11+ installation: Testing in progress on Windows 11
- ✅ Git: Validated (standard tooling)
- ⚠️ OpenAI API integration: Needs validation
- ⚠️ Tavily API integration: Needs validation
- ⚠️ Anthropic API integration: Experimental, needs validation
- ⚠️ Direct Python installation: Needs validation on Windows 11
- ⚠️ Docker Compose installation: Needs validation
- ⚠️ Claude Desktop integration: Needs validation on Windows 11
- ⚠️ n8n MCP integration: Needs validation

#### Documentation
- Comprehensive installation options with pros/cons for each path
- Step-by-step setup instructions for all installation methods
- Environment variable documentation with examples
- Platform-specific configuration notes
- Links to upstream documentation and community resources

#### Next Steps
As installation paths are tested and validated, this changelog will be updated with new PATCH versions reflecting validation status changes.

---

## Example Changelog Entries

Below are example changelog entries to guide future updates:

---

### Example: Minor Version (New Feature)

## [1.1.0] - 2025-11-15

### Added
- Custom retriever support for GPT Researcher
- Installation instructions for custom retriever configuration
- Environment variable `CUSTOM_RETRIEVER` (optional)
- Validation examples for custom retriever setup

### Changed
- Updated Prerequisites section to include custom retriever as optional
- Enhanced Troubleshooting section with retriever-specific issues

### Validation Status
- ⚠️ Custom retriever on Windows 11: Needs validation
- ⚠️ Custom retriever on macOS: Needs validation

---

### Example: Patch Version (Documentation Fix)

## [1.0.1] - 2025-10-28

### Fixed
- Corrected typo in Docker network configuration example
- Fixed broken link to GPT Researcher documentation
- Clarified virtual environment activation commands for Windows PowerShell

### Changed
- Improved formatting in Environment Configuration section

---

### Example: Major Version (Breaking Change)

## [2.0.0] - 2025-12-01

### BREAKING CHANGES
- **Python 3.12+ now required** (was Python 3.11+)
  - Reason: Dependencies now require Python 3.12+ for new async features
  - Impact: Users on Python 3.11 must upgrade

### Migration Guide
For users upgrading from v1.x.x:

1. **Upgrade Python**:
   ```bash
   # Check current version
   python --version

   # Install Python 3.12+ (see Prerequisites section)
   ```

2. **Recreate Virtual Environment**:
   ```bash
   # Deactivate current environment
   deactivate

   # Remove old environment
   rm -rf venv  # macOS/Linux
   rmdir /s venv  # Windows

   # Create new environment with Python 3.12+
   python3.12 -m venv venv
   source venv/bin/activate  # macOS/Linux
   venv\Scripts\activate  # Windows
   ```

3. **Reinstall Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

### Added
- Support for new async features in Python 3.12+
- Updated all platform validation with Python 3.12

### Removed
- Python 3.11 support
- Deprecated `LEGACY_MODE` environment variable

### Validation Status
- ✅ Python 3.12: All platforms
- ✅ Python 3.13: Windows 11, macOS 14

---

## Changelog Best Practices

When adding entries to this changelog:

### Structure
Each version should include:
1. **Version number** in `[X.Y.Z]` format
2. **Date** in YYYY-MM-DD format
3. **Summary** (optional, for major versions)
4. **Sections** (use relevant ones):
   - `Added` - New features, options, instructions
   - `Changed` - Updates to existing content
   - `Deprecated` - Soon-to-be removed features (with migration path)
   - `Removed` - Removed features, options
   - `Fixed` - Bug fixes, corrections
   - `Security` - Security-related changes
   - `BREAKING CHANGES` - Changes requiring user action
   - `Migration Guide` - How to upgrade from previous version
   - `Validation Status` - Updated validation badges

### Writing Good Entries

**Good**:
- ✅ "Added Docker Compose installation option with n8n integration support"
- ✅ "Changed Python requirement from 3.11+ to 3.12+ (BREAKING)"
- ✅ "Fixed incorrect Docker network configuration in n8n setup section"

**Bad**:
- ❌ "Updated stuff"
- ❌ "Fixed things"
- ❌ "Changes to installation"

### Version Increment Decision Tree

```
Does the change...
  ├─ Break existing setups? (require user action)
  │  └─ YES → MAJOR version (2.0.0)
  │
  ├─ Add new features/options? (backward-compatible)
  │  └─ YES → MINOR version (1.1.0)
  │
  └─ Fix docs/clarify existing content?
     └─ YES → PATCH version (1.0.1)
```

---

## Template Metadata

**Template Version**: 1.0.0
**Created**: 2025-10-25
**Purpose**: Track installation guide changes with semantic versioning

---

## Notes

- This changelog is automatically updated by AI agents following `CLAUDE_DEV.md` policies
- Each entry should clearly explain what changed and why
- Breaking changes must include migration guides
- Validation status updates don't require version increments unless they change documentation
- Project: gptr-mcp (GPT Researcher MCP Server) - Personal fork by RoscoeTheDog
