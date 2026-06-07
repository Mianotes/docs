# API overview

Created: 2026-05-31T09:43:49Z

## Note

The Mianotes web service exposes a local REST API. All endpoints return JSON unless the endpoint explicitly returns stored file bytes.

## Base URL

Local development and local deployments use port `8200` by default:

```text
http://127.0.0.1:8200
```

## Authentication

Protected endpoints accept one of:

* browser session cookie created by the auth flow;
* user-scoped API key installed by **Settings > Connect tools**;
* scoped per-user API token created through `/api/tokens`;
* short-lived agent session token returned by `POST /api/auth/agent-session`.

Bearer token shape:

```http
Authorization: Bearer mia_<token>
```

Agent clients may call the REST API directly with their user-scoped API key, or
exchange that key for a short-lived agent session:

```http
POST /api/auth/agent-session
Authorization: Bearer mia_<token>
```

Mianotes identifies the user from the bearer token. The returned session token
is also sent as `Authorization: Bearer <token>`.

## Common errors

### Not authenticated

```json
{
  "detail": "Not signed in"
}
```

### Permission denied

```json
{
  "detail": "API token requires notes:read scope"
}
```

### Validation error

```json
{
  "detail": [
    {
      "type": "missing",
      "loc": ["body", "email"],
      "msg": "Field required",
      "input": {}
    }
  ]
}
```

## API endpoints

### Health and storage

* `GET /api/health`
* `GET /api/storage`

### Auth

* `POST /api/auth/check-email`
* `POST /api/auth/join`
* `POST /api/auth/login`
* `GET /api/auth/session`
* `POST /api/auth/logout`

### Tokens

* `POST /api/tokens`
* `GET /api/tokens`
* `DELETE /api/tokens/{token_id}`
* `POST /api/settings/api-key`
* `POST /api/install/skill`

### Users

* `POST /api/users`
* `GET /api/users`
* `GET /api/users/{user_id}`
* `PATCH /api/users/{user_id}`
* `POST /api/users/{user_id}/photo`
* `DELETE /api/users/{user_id}`

### Folders

* `POST /api/folders`
* `GET /api/folders`
* `GET /api/folders/{folder_id}`
* `PATCH /api/folders/{folder_id}`
* `DELETE /api/folders/{folder_id}`

### Notes

* `POST /api/notes/from-text`
* `POST /api/notes/from-file`
* `POST /api/notes/from-url`
* `GET /api/notes`
* `GET /api/notes/{note_id}`
* `PATCH /api/notes/{note_id}`
* `PATCH /api/notes/{note_id}/star`
* `DELETE /api/notes/{note_id}`

### Sharing

* `POST /api/notes/{note_id}/share`
* `DELETE /api/notes/{note_id}/share`
* `GET /api/notes/shared/workspaces/{workspace_id}/{token}`
* `GET /api/notes/shared/workspaces/{workspace_id}/{token}/files/{source_file_id}`
* `GET /api/notes/shared/workspaces/{workspace_id}/{token}/avatar`

Guest share routes include the workspace id so the service can resolve the shared note directly from the correct workspace database.

### Tags

* `GET /api/tags`
* `PUT /api/notes/{note_id}/tags`

### Mia

* `POST /api/notes/{note_id}/prompt`

### Search and context

* `GET /api/search`
* `GET /api/context`

### Jobs

* `GET /api/jobs`
* `GET /api/jobs/{job_id}`

### Publishing

* `GET /api/publish/themes`
* `GET /api/publish/draft`
* `POST /api/publish`
* `GET /api/publish/{site_id}/download`

### Settings

* `GET /api/settings/storage`
* `POST /api/settings/storage/locations`
* `PATCH /api/settings/storage/active`
* `POST /api/settings/api-key`

### Stored files

* `GET /markdown/{file_path}`
* `GET /api/workspaces/{workspace_id}/notes/{note_id}/markdown`
* `GET /api/workspaces/{workspace_id}/notes/{note_id}/source-files/{source_file_id}`
* `GET /api/workspaces/{workspace_id}/notes/{note_id}/images/{file_path}`

## Health check

```text
GET /api/health
```

Authentication: none.

Example response:

```json
{
  "status": "ok",
  "service": "mianotes-web-service",
  "version": "0.1.0",
  "storage": {
    "data_dir": "data",
    "database_url": "sqlite:///data/system.db",
    "storage_config_path": "workspaces.json"
  }
}
```

## Storage capacity

```text
GET /api/storage
```

Authentication: browser session or API token with `notes\:read`.

Returns cached capacity information for the drive that stores Mianotes data. The service refreshes this snapshot at most once per hour.
