# SEMOSS Vibe Coding Setup — Claude Code

Build SEMOSS applications with Claude Code. Clone, set your credentials, run `claude`.

## Quick Start

### 1. Clone and enter the project

```bash
git clone <repo-url> my-semoss-app
cd my-semoss-app
```

### 2. Set your SEMOSS credentials

Get your access key and secret key from SEMOSS: **Settings > My Profile**.

**macOS / Linux (zsh or bash):**

```bash
export SEMOSS_ACCESS_KEY="your-access-key"
export SEMOSS_SECRET_KEY="your-secret-key"
```

To persist across sessions, add those lines to `~/.zshrc` or `~/.bashrc`.

**Windows (PowerShell):**

```powershell
$env:SEMOSS_ACCESS_KEY = "your-access-key"
$env:SEMOSS_SECRET_KEY = "your-secret-key"
```

To persist, add to your PowerShell profile (`$PROFILE`).

### 3. Run Claude Code

```bash
claude
```

Claude will automatically connect to the SEMOSS MCP servers, verify your setup, and walk you through any remaining configuration (like setting your project ID).

## Alternative: Let Claude Guide You

Just clone and run `claude` without setting environment variables first. Claude will detect the missing credentials and tell you exactly what to set. You'll need to restart `claude` after setting them so the MCP servers reconnect.

## Prerequisites

| Tool | Version | Purpose | Install |
|------|---------|---------|---------|
| **Claude Code** | Latest | AI coding assistant CLI | [claude.ai/download](https://claude.ai/download) |
| **Node.js** | 24+ | Required for `npx mcp-remote` bridge | [nodejs.org](https://nodejs.org) or `brew install node` |
| **Python** | 3.10+ | Asset sync script | Pre-installed on most systems |
| **SEMOSS credentials** | — | `ACCESS_KEY` and `SECRET_KEY` from Settings > My Profile | — |

Verify prerequisites:

```bash
claude --version    # Claude Code CLI
node --version      # v24.x+
python3 --version   # 3.10+
npx --version       # 11.x+
```

## What's Included

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project instructions for Claude Code — SEMOSS workflows, conventions, constraints |
| `.mcp.json` | MCP server config with 3 SEMOSS servers — uses `${env:VAR}` references for credentials |
| `semoss_config/config.json` | SEMOSS project metadata (project ID, base URL, module) |
| `scripts/semoss_asset_sync.py` | Upload local files to SEMOSS and sync remote assets to local |
| `.gitignore` | Git ignore rules for temp files and OS artifacts |

## How Credentials Work

`.mcp.json` uses Claude Code's `${env:VAR}` syntax to reference environment variables:

```json
"--header",
"Authorization:Bearer${env:SEMOSS_ACCESS_KEY}:${env:SEMOSS_SECRET_KEY}"
```

Claude Code resolves these at MCP server startup. Since the file contains only variable references (not actual secrets), it's safe to track in git.

The `scripts/semoss_asset_sync.py` script also resolves `${env:VAR}` references when reading `.mcp.json`, so it works seamlessly with the same credential setup.

## .mcp.json Configuration

Claude Code uses a project-level `.mcp.json` file. Three SEMOSS MCP servers are pre-configured:

| Server | Engine ID | Purpose |
|--------|-----------|---------|
| `Semoss_Platform_Instructions` | `67aa0dcf-04f5-460f-9075-bad8eeedad7e` | SEMOSS platform documentation and guidance |
| `Semoss_project_manager` | `03e8cbeb-2f76-4213-9766-57448a3837a4` | Project management — create, list, upload, publish |
| `Semoss_database_helper` | `394404bf-02e5-44b2-bc7c-e93d9b698f58` | Database operations — create, query, schema |

If you're using a different SEMOSS instance, update `base_url` and `module` in both `.mcp.json` URLs and `semoss_config/config.json`.

## Asset Sync Script

`scripts/semoss_asset_sync.py` handles file synchronization between local and SEMOSS.

### Upload a file

```bash
python scripts/semoss_asset_sync.py upload portals/index.html
# or shorthand (upload is the default command):
python scripts/semoss_asset_sync.py portals/index.html
```

The script infers the remote path from the local workspace path. For example, `portals/index.html` uploads to `version/assets/portals/index.html` in SEMOSS.

### Sync from remote

```bash
python scripts/semoss_asset_sync.py sync-from-remote portals
python scripts/semoss_asset_sync.py sync-from-remote portals --local-dir portals --overwrite
```

### Behavior

- Reads project config from `semoss_config/config.json`
- Reads credentials from `.mcp.json` (resolves `${env:VAR}` references) or falls back to `.vscode/mcp.json`
- Backs up existing remote files before overwrite (saved to `temp/semoss_backups/`)
- Publishes the project after upload so changes take effect
- Uses only Python standard library modules (`requests` used for downloads if available, otherwise `urllib`)

## SEMOSS URL Patterns

| Purpose | URL Pattern |
|---------|-------------|
| App view | `<base_url><web_module_url>/packages/client/dist/#/app/<project_id>/view` |
| Database maker | `<base_url><web_module_url>/packages/client/dist/#/app/394404bf-02e5-44b2-bc7c-e93d9b698f58/view` |
| Existing database | `<base_url><web_module_url>/packages/client/dist/#/engine/database/<database_id>` |

With defaults:

- App view: `https://workshop.cfg.deloitte.com/cfg-ai-dev/SemossWeb/packages/client/dist/#/app/<project_id>/view`
- Database maker: `https://workshop.cfg.deloitte.com/cfg-ai-dev/SemossWeb/packages/client/dist/#/app/394404bf-02e5-44b2-bc7c-e93d9b698f58/view`

## Working Conventions

- Build the UI as a single-page HTML app unless specified otherwise
- Keep implementations small and reviewable
- Do not install new libraries — use what's available
- Prefer the SEMOSS MCP tools and the asset sync script for all SEMOSS operations
- After each meaningful change, sync to SEMOSS and offer the app URL
