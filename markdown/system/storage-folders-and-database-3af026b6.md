# Storage folders and database

Created: 2026-05-31T09:57:31Z

## Note

Mianotes uses SQLite by default. In the UI, admins choose local workspaces. Workspace folders store Markdown notes, source files, published HTML, and other content files. Private SQLite databases live under the Mianotes data directory.

## Default databases

On first start, Mianotes reads `settings.json` if it exists, then creates a runtime `workspaces.json` file in the web service root if it does not already exist.

A fresh install uses:

```text
data/system.db
data/workspaces/default.db
```

`data/system.db` stores global users, sessions, API key verification, and global settings.

`data/workspaces/default.db` is the default workspace database. Additional workspaces use the same pattern:

```text
data/workspaces/<workspace_id>.db
```

This keeps database files in the application data area and leaves selected workspace folders cleaner.

## `workspaces.json`

`workspaces.json` lists the local workspaces available to the app.

Example:

```json
{
  "activeLocation": "default",
  "allowedStorageLocations": [
    {
      "id": "default",
      "name": "Main workspace",
      "folderPath": "/Users/example/Mianotes/data"
    }
  ]
}
```

Each location points to a workspace folder. Each workspace has its own notes, folders, tags, source records, jobs, shares, and publishing history in `data/workspaces/<workspace_id>.db`.

Users, sessions, API keys, and global settings live in `data/system.db`.

## `settings.json`

`settings.json` contains project-level service settings. It can set the server host and port, model providers, binary lookup paths, and the database adapter.

Secrets should not be written directly into `settings.json`. Use environment placeholders such as:

```json
{
  "llm": {
    "provider": "openai",
    "model": "gpt-5-nano",
    "baseUrl": "",
    "apiKey": "env.MIANOTES_LLM_API_KEY"
  }
}
```

Mianotes resolves `env.MIANOTES_LLM_API_KEY` from the process environment or `.env`.

The database section currently supports SQLite:

```json
{
  "database": {
    "adapter": "sqlite",
    "url": "env.MIANOTES_DATABASE_URL"
  }
}
```

If `MIANOTES_DATABASE_URL` is not set, Mianotes uses `data/system.db` for global app state. Other adapters are intentionally rejected for now so workspace data cannot be mixed into one server database before the schema supports it.

## Database initialisation

Mianotes creates and updates database tables in two places:

1. on service startup;
2. when `mianotes-web-service init-db` is run.

Startup initialises `data/system.db`, then initialises every configured workspace database from `workspaces.json`.

SQLite file databases are opened in WAL mode. This creates sidecar files such as `system.db-wal` and `system.db-shm`, which are normal SQLite runtime files and should not be committed.

## Switching workspaces

All signed-in users can switch workspace from the workspace switcher next to the breadcrumb. Admins can add workspaces from Settings.

When a user switches workspace, Mianotes:

1. stores the selected workspace on that user's session;
2. routes workspace-content API calls to `data/workspaces/<workspace_id>.db`;
3. keeps the browser session signed in;
4. leaves other users on their own selected workspace.

Switching workspace is per user session. It is not global process state.

## Creating a new workspace

Admins can add a new local workspace from Settings.

Mianotes creates `data/workspaces/<workspace_id>.db` and initialises the workspace schema. Users remain global and can use the new workspace without signing up again.

## Private files

Do not commit these to a public repository:

```text
workspaces.json
data/
data/workspaces/
system.db
.env
```

Database files are never served by the file API. Source and Markdown files are served through controlled routes, but `system.db`, workspace databases, and SQLite sidecars are blocked.
