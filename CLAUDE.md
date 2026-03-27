# CLAUDE.md

This project builds SEMOSS applications. Prefer pragmatic, reviewable changes and keep outputs concise.

## Project Structure

```
my-semoss-app/
├── CLAUDE.md                        # This file — project instructions
├── .mcp.json                        # MCP server configuration (gitignored)
├── semoss_config/
│   └── config.json                  # SEMOSS project metadata
├── portals/
│   └── index.html                   # Single-page HTML application
├── scripts/
│   └── semoss_asset_sync.py         # Upload/download helper for SEMOSS assets
└── temp/
    └── semoss_backups/              # Local backups of overwritten remote files
```

## Core Environment

- **local** means the current machine and files in this workspace
- **semoss** means the remote SEMOSS project
- Python is available for simple file operations (Base64 encode/decode, asset sync)
- Do not install new libraries
- Do not create virtual environments
- Only use the specified SEMOSS MCP servers — do not use alternative infrastructure paths

## Default SEMOSS Values

If the user has not specified an instance, default to:

| Setting | Default |
|---------|---------|
| `base_url` | `https://workshop.cfg.deloitte.com/` |
| `api_module_url` | `/cfg-ai-dev/Monolith` |
| `web_module_url` | `/cfg-ai-dev/SemossWeb` |

If these values are confirmed or changed, update `semoss_config/config.json` and `.mcp.json` accordingly.

## Required Startup Workflow

Before doing substantial work:

1. Check whether the folder is already connected to a SEMOSS project
2. Look for `semoss_config/config.json` first
3. If no config exists, offer to create a SEMOSS project
4. Persist project metadata once known (`project_id`, `module`, `base_url`, `created_on`)

If the user wants a non-default SEMOSS instance, ask which instance to use and record it as `base_url`.

## semoss_config Requirements

Store SEMOSS project metadata as JSON in `semoss_config/config.json`.

Minimum fields:

```json
{
  "project_id": "uuid-of-the-project",
  "module": "/cfg-ai-dev/Monolith",
  "created_on": "2026-03-26",
  "base_url": "https://workshop.cfg.deloitte.com/"
}
```

If a new remote project is created, persist the same config into that remote project's config directory as well.

## MCP Configuration

Claude Code reads `.mcp.json` from the project root automatically.

- Ask the user for SEMOSS `ACCESS_KEY` and `SECRET_KEY` if the `.mcp.json` still has placeholder values
- If credentials are already present, confirm and proceed
- Ask for any unresolved placeholders (`base_url`, module paths, project IDs)
- As an initial orientation step, list the available MCP tools so the user knows what's available

Three SEMOSS MCP servers are configured:

| Server | Purpose |
|--------|---------|
| `Semoss_Platform_Instructions` | Platform documentation and guidance |
| `Semoss_project_manager` | Project management — create, list, upload, publish |
| `Semoss_database_helper` | Database operations — create, query, schema |

## File Sync Rules

When saving files to SEMOSS, use the `scripts/semoss_asset_sync.py` helper or the `ai_server` SDK directly.

### Upload with the helper script

```bash
python scripts/semoss_asset_sync.py upload portals/index.html
```

### Upload with the SDK

```python
from ai_server import ServerClient

access = "first part of bearer token"
secret = "second part of bearer token"
endpoint = "https://workshop.cfg.deloitte.com/cfg-ai-dev/Monolith/api/"

server_connection = ServerClient(base=endpoint, access_key=access, secret_key=secret)
insight_id = server_connection.make_new_insight()
server_connection.upload_files(
    files=["path/to/local/file"],
    project_id="<project_id>",
    insight_id=f"{insight_id}",
    path="/version/assets/<appropriate folder>",
)
```

### Before overwriting a remote file

1. Check whether the remote file already exists
2. Tell the user you plan to delete it
3. Wait for confirmation before deleting
4. Delete the remote file
5. List files to confirm removal
6. Upload the replacement
7. Publish the project so the change takes effect

## Database Rules

If a task involves a database:

1. Ask the user to create the database or provide the database ID
2. After receiving the ID, call `get_schema(database_id)` via the MCP tools
3. The returned schema is Base64-encoded — decode it before use
4. Write the decoded schema into `semoss_config/` for reference

Example Base64 helper:

```python
import base64
from pathlib import Path

encoded = Path("schema.b64").read_text()
decoded = base64.b64decode(encoded).decode("utf-8")
Path("semoss_config/schema.sql").write_text(decoded)
```

When creating temporary Base64 files:
- Put them under a `temp/` directory
- Mirror the source directory structure
- Delete them after use if no longer needed

## User Interaction Rules

- After each meaningful modification, offer to synchronize local changes to SEMOSS
- After sync, offer to publish or share the application URL
- If the user asks for a complex task, present a concise task list and get confirmation before proceeding
- Be concise in code and communication

## URL Patterns

Offer these URLs to the user when relevant:

| Purpose | URL Pattern |
|---------|-------------|
| App view | `<base_url><web_module_url>/packages/client/dist/#/app/<project_id>/view` |
| Database maker | `<base_url><web_module_url>/packages/client/dist/#/app/394404bf-02e5-44b2-bc7c-e93d9b698f58/view` |
| Existing database | `<base_url><web_module_url>/packages/client/dist/#/engine/database/<database_id>` |

## UI Conventions

- Build the UI as a single-page HTML app unless the user specifies otherwise
- Use the SEMOSS directory structure (`portals/index.html` as the entry point)
- Keep implementations small and reviewable

## Constraints

- Do not use alternative infrastructure paths when the SEMOSS MCP tools cover the task
- Do not install packages
- Do not run dangerous or destructive commands
- Always start by getting familiar with the SEMOSS instructions and available MCP tools

## Common Pitfalls

- **Forgetting to publish** — Uploaded files are not visible until the project is published
- **Overwriting without backup** — Always check for existing remote files before upload; use the asset sync script which handles backups automatically
- **Stale credentials** — If MCP tools stop working, check that credentials in `.mcp.json` are still valid
- **Wrong module path** — `api_module_url` (for API calls) and `web_module_url` (for user-facing URLs) are different paths
- **Base64 schema** — Database schemas from `get_schema()` are Base64-encoded; decode before use
