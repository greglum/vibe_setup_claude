# Scripts

Deployment and asset management tools for SEMOSS projects.

## semoss_asset_sync.py

The primary tool for moving files between your local workspace and SEMOSS. Reads project config from `semoss_config/config.json` and credentials from `.mcp.json`.

### Commands

| Command | Purpose |
|---------|---------|
| `upload <file>` | Upload a single file (backup existing, delete, upload, publish) |
| `bulk-upload <paths...>` | Upload files/directories in one process, publish once at the end |
| `delete <remote-path>` | Delete a remote file or directory contents |
| `publish` | Publish the project without uploading (use after `--no-publish` batches) |
| `sync-from-remote <folder>` | Download a remote folder to local workspace |

### Common Workflows

**First deploy (build + upload everything):**

```bash
cd client && pnpm run build && cd ..
python scripts/semoss_asset_sync.py bulk-upload portals
```

**Redeploy (clean stale assets first):**

```bash
cd client && pnpm run build && cd ..
python scripts/semoss_asset_sync.py delete portals/assets --yes
python scripts/semoss_asset_sync.py bulk-upload portals
```

Vite emits new content-hashed filenames on every build. Deleting `portals/assets` first prevents dead files from accumulating.

**Upload a single file:**

```bash
python scripts/semoss_asset_sync.py upload portals/index.html
```

The shorthand also works (omitting `upload`):

```bash
python scripts/semoss_asset_sync.py portals/index.html
```

**Chain multiple uploads with a single publish:**

```bash
python scripts/semoss_asset_sync.py bulk-upload portals --no-publish
python scripts/semoss_asset_sync.py bulk-upload java --no-publish
python scripts/semoss_asset_sync.py publish
```

**Pull remote files locally:**

```bash
python scripts/semoss_asset_sync.py sync-from-remote portals
python scripts/semoss_asset_sync.py sync-from-remote portals --local-dir portals --overwrite
```

### Flags

| Flag | Available on | Purpose |
|------|-------------|---------|
| `--yes` / `-y` | `upload`, `delete` | Skip confirmation prompts |
| `--no-publish` | `upload`, `bulk-upload` | Defer publish (chain with `publish` command) |
| `--no-delete-existing` | `bulk-upload` | Skip browse-and-delete step (faster on first deploy) |
| `--overwrite` | `sync-from-remote` | Overwrite local files without prompting |
| `--local-dir` | `sync-from-remote` | Override local destination directory |

### How It Works

1. Reads `semoss_config/config.json` for project ID and SEMOSS routing
2. Extracts credentials from `.mcp.json` (Claude Code) or `.vscode/mcp.json` (Copilot)
3. Connects to SEMOSS via the Python `ai-server` SDK
4. For uploads: checks if remote file exists, backs up if so, deletes, uploads, publishes
5. For bulk uploads: walks directories, groups by remote parent, reuses one connection, publishes once

### Prerequisites

The SEMOSS Python SDK (`ai-server`) must be available. Claude Code environments typically have this pre-installed. If not:

```bash
pip install ai-server-sdk
```

### Safety

- **Backups:** Single-file `upload` backs up existing remote files to `temp/semoss_backups/` before overwriting
- **Bulk uploads skip backups:** Build artifacts are reproducible from git; backups would be dead weight
- **Fail-safe publishing:** If `bulk-upload` fails partway through, it exits without publishing — the previously published release stays live
- **Hidden files excluded:** Directory walks skip dotfiles (`.DS_Store`, `.swp`) unless explicitly named
