# SEMOSS Vibe Coding Setup — Claude Code

A template for building SEMOSS web applications with Claude Code. **Clone this repo once per application** — each clone becomes an independent project.

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│  client/         React SPA (scaffolded by Claude)        │
├──────────────────────────────────────────────────────────┤
│  portals/        Build output (deployed to SEMOSS)       │
├──────────────────────────────────────────────────────────┤
│  SEMOSS Platform  (remote host, DB, LLM, SDK)            │
└──────────────────────────────────────────────────────────┘
```

**Data flow:** Page → Hook → Service → `runPixel()` → SEMOSS SDK → Java Reactor → Database

## Quick Start

```bash
# Clone for a new app
git clone <repo-url> my-semoss-app
cd my-semoss-app

# Set credentials (from SEMOSS Settings > My Profile)
export SEMOSS_ACCESS_KEY="your-access-key"
export SEMOSS_SECRET_KEY="your-secret-key"

# Start Claude — it handles the rest
claude
```

Claude will connect to SEMOSS, prompt you for a project ID (or create one), scaffold the React app, and walk you through development.

To persist credentials, add the exports to `~/.zshrc` or `~/.bashrc`. On Windows, use `$env:SEMOSS_ACCESS_KEY = "..."` in PowerShell.

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Claude Code | Latest | [claude.ai/download](https://claude.ai/download) |
| Node.js | 20.19+ | [nodejs.org](https://nodejs.org) or `brew install node` |
| pnpm | 10.x | `corepack enable && corepack prepare pnpm@latest --activate` |
| Python | 3.10+ | Pre-installed on most systems |

## Directory Guide

| Path | Purpose | Docs |
|------|---------|------|
| `CLAUDE.md` | Project instructions — scaffolding, conventions, SEMOSS workflows | — |
| `.mcp.json` | MCP server config (3 SEMOSS servers, uses `${env:VAR}` for credentials) | — |
| `semoss_config/config.json` | SEMOSS project metadata (project ID, module, database ID) | — |
| `scripts/` | Deployment & asset sync tooling | [scripts/README.md](scripts/README.md) |
| `client/` | React 19 SPA (scaffolded by Claude) | — |
| `portals/` | Build output (gitignored) — deployed to SEMOSS | — |
| `.env.example` | Environment variable template | — |

## Development

Once Claude scaffolds the React app in `client/`:

```bash
cd client
pnpm install        # Install dependencies (pnpm only)
pnpm run dev        # Dev server with HMR
pnpm run build      # Type-check + build to ../portals/
pnpm run check:fix  # Lint & format (Biome)
pnpm run test:run   # Run tests
```

Tech stack: React 19, TypeScript, Vite 8, Tailwind CSS v4, shadcn/ui (Base UI), TanStack Query v5, React Router v7, Biome.

## Deploy

Use `scripts/semoss_asset_sync.py` for deployment — it handles the full workflow (stale asset cleanup, bulk upload, publish):

```bash
cd client && pnpm run build && cd ..
python scripts/semoss_asset_sync.py delete portals/assets --yes
python scripts/semoss_asset_sync.py bulk-upload portals
```

`bulk-upload` walks `portals/` in a single Python process and publishes once at the end. Delete old assets first because Vite emits new content hashes on every build.

See [scripts/README.md](scripts/README.md) for all commands and workflows.

## Credentials

`.mcp.json` uses `${env:SEMOSS_ACCESS_KEY}` and `${env:SEMOSS_SECRET_KEY}` references — Claude Code resolves these at startup. No secrets in git.

If MCP tools stop working, check that your env vars are still valid and restart `claude`.

## Multiple Applications

Each app gets its own clone:

```bash
git clone <repo-url> inventory-app    # App 1
git clone <repo-url> dashboard-app    # App 2
```

Each clone maintains its own project ID, `client/` source, and git history. The template provides the scaffolding machinery; Claude builds the app.

## SEMOSS URLs

| Purpose | URL |
|---------|-----|
| App view | `https://workshop.cfg.deloitte.com/cfg-ai-dev/SemossWeb/packages/client/dist/#/app/<project_id>/view` |
| Database maker | `https://workshop.cfg.deloitte.com/cfg-ai-dev/SemossWeb/packages/client/dist/#/app/394404bf-02e5-44b2-bc7c-e93d9b698f58/view` |

Replace `<project_id>` with the value from `semoss_config/config.json`.
