# Storage folders and database

Created: 2026-05-31T09:57:31Z

## Note

Mianotes uses SQLite by default. In the UI, admins choose local workspaces. Inside each workspace, Mianotes keeps private runtime data in a hidden `.mianotes/` directory.

## Default databases

On first start, Mianotes creates a runtime `workspaces.json` file in the web service root if it does not already exist.

A fresh install uses:

```text
data/system.db
data/.mianotes/mia.db
```

`data/system.db` stores global users, sessions, API key verification, and global settings.

`data/.mianotes/mia.db` is the default workspace database. Workspace folders contain Markdown notes, source files, folder directories, and the hidden `.mianotes/` directory.

Mianotes also writes a `.gitignore` into each selected storage folder:

```text
.mianotes/
.mianotes/mia.db
mia.db
system.db
system.db-wal
system.db-shm
system.db-journal
```

This keeps private runtime data out of Git repositories when users choose a project folder.

## `workspaces.json`

`workspaces.json` lists the local workspaces available to the app.

Example:

```json
{
  "activeLocation": "default",
  "databaseFile": ".mianotes/mia.db",
  "allowedStorageLocations": [
    {
      "id": "default",
      "name": "Main workspace",
      "folderPath": "/Users/example/Mianotes/data"
    }
  ]
}
```

Each location points to a workspace folder that contains, or can contain, a `.mianotes/mia.db` database. Each workspace has its own notes, folders, tags, source records, jobs, shares, and publishing history.

Users, sessions, API keys, and global settings live in `data/system.db`.

## Switching workspaces

All signed-in users can switch workspace from the workspace switcher next to the breadcrumb. Admins can add workspaces from Settings.

When a user switches workspace, Mianotes:

1. stores the selected workspace on that user's session;
2. routes workspace-content API calls to the selected `.mianotes/mia.db`;
3. keeps the browser session signed in;
4. leaves other users on their own selected workspace.

Switching workspace is per user session. It is not global process state.

## Creating a new workspace

Admins can add a new local workspace from Settings.

Mianotes creates `.mianotes/mia.db` inside that workspace and initialises the workspace schema. Users remain global and can use the new workspace without signing up again.

## Private files

Do not commit these to a public repository:

```text
workspaces.json
data/
.mianotes/
mia.db
system.db
.env
```

Database files are never served by the file API. Source and Markdown files are served through controlled routes, but `mia.db`, `system.db`, and their SQLite sidecars are blocked.
