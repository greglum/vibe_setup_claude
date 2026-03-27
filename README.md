# SEMOSS Vibe Coding Setup — Claude Code

A self-contained starter template for building SEMOSS applications with Claude Code. Copy this directory into a new project to get started.

## What This Template Includes

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project instructions for Claude Code — SEMOSS workflows, conventions, constraints |
| `mcp.json.example` | Example `.mcp.json` with 3 SEMOSS MCP servers pre-configured |
| `semoss_config.example.json` | Example `semoss_config/config.json` for project metadata |
| `scripts/semoss_asset_sync.py` | Upload local files to SEMOSS and sync remote assets to local |

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

## Setup Steps

### 1. Copy the template into your project

```bash
mkdir my-semoss-app && cd my-semoss-app
git init

# Copy template files
cp -r path/to/vibe-coding-setup/* .
```

### 2. Configure SEMOSS project metadata

```bash
mkdir -p semoss_config
cp semoss_config.example.json semoss_config/config.json
```

Edit `semoss_config/config.json` with your project details:

```json
{
  "project_id": "your-actual-project-id",
  "module": "/cfg-ai-dev/Monolith",
  "created_on": "2026-03-26",
  "base_url": "https://workshop.cfg.deloitte.com/"
}
```

### 3. Configure MCP servers

```bash
cp mcp.json.example .mcp.json
```

Edit `.mcp.json` and replace the credential placeholders:

- Replace `YOUR_ACCESS_KEY` with your SEMOSS access key
- Replace `YOUR_SECRET_KEY` with your SEMOSS secret key

Claude Code reads `.mcp.json` from the project root automatically — no merge step needed.

> **Security:** Add `.mcp.json` to your `.gitignore` — it contains credentials.

### 4. Start Claude Code

```bash
claude
```

Claude will automatically discover the MCP servers defined in `.mcp.json` and load the project instructions from `CLAUDE.md`.

As an initial step, ask Claude to list the available SEMOSS MCP tools so you know what's available.

## .mcp.json Configuration

Claude Code uses a project-level `.mcp.json` file (not a global config). The template defines three SEMOSS MCP servers:

| Server | Engine ID | Purpose |
|--------|-----------|---------|
| `Semoss_Platform_Instructions` | `67aa0dcf-04f5-460f-9075-bad8eeedad7e` | SEMOSS platform documentation and guidance |
| `Semoss_project_manager` | `03e8cbeb-2f76-4213-9766-57448a3837a4` | Project management — create, list, upload, publish |
| `Semoss_database_helper` | `394404bf-02e5-44b2-bc7c-e93d9b698f58` | Database operations — create, query, schema |

If you're using a different SEMOSS instance, update `base_url` and `module` in both `.mcp.json` URLs and `semoss_config/config.json`.

### Credential security improvement

Instead of hardcoding credentials in `.mcp.json`, you can use environment variables:

```bash
# Add to ~/.zshrc or ~/.bashrc
export SEMOSS_ACCESS_KEY="your-access-key"
export SEMOSS_SECRET_KEY="your-secret-key"
```

Then reference them in `.mcp.json` using Claude Code's `${env:VAR}` syntax:

```json
"--header",
"Authorization:Bearer${env:SEMOSS_ACCESS_KEY}:${env:SEMOSS_SECRET_KEY}"
```

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
- Reads credentials from `.mcp.json` (Claude Code format) or falls back to `.vscode/mcp.json` (Copilot format)
- Backs up existing remote files before overwrite (saved to `temp/semoss_backups/`)
- Publishes the project after upload so changes take effect

### Dependencies

The script uses only Python standard library modules. If `requests` is installed, it uses that for downloads; otherwise falls back to `urllib`.

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

## Additional Resources

For more context on SEMOSS MCP setup across different AI coding assistants, see:

- [SEMOSS MCP Quick Start](../semoss-mcp-quick-start.md)
- [SEMOSS MCP Client Setup Guide](../semoss-mcp-client-setup-guide.md)
