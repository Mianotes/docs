# Mia prompts and jobs API

Created: 2026-05-29T17:20:37Z

## Note

This page covers private Mia prompts and background jobs.

## Send a private Mia prompt

Use the note prompt endpoint:

```text
POST /api/notes/{note_id}/prompt
```

Authentication: session cookie or bearer token with `notes\:write` or `admin`.

```json
{
  "prompt": "summarise this text"
}
```

Response: a prompt response with `200 OK`.

```json
{
  "type": "prompt",
  "prompt": "summarise this text",
  "note_id": "4a95f146-9d27-4c79-b7d8-34739aef8998",
  "text": "## Summary\n\nThe note explains the Mallorca trip plan...",
  "format": "markdown"
}
```

Mia prompt responses do not create jobs and do not update the note.

## Job object

```json
{
  "id": "dc6d54d2-f6ac-4a87-9d54-12e93243db4e",
  "user": {
    "id": "c5ddebcc-e434-4e1a-bc8a-48263eb0095d",
    "email": "matt@example.com",
    "name": "Matt",
    "username": "matt",
    "phone": null,
    "role": null,
    "photo_url": null,
    "is_admin": true
  },
  "client": {
    "key": "codex",
    "name": "Codex"
  },
  "note_id": "4a95f146-9d27-4c79-b7d8-34739aef8998",
  "job_type": "parse_file",
  "status": "queued",
  "input": {
    "note_id": "4a95f146-9d27-4c79-b7d8-34739aef8998",
    "source_file_id": "6aa09fd4-ef76-4cb4-88df-7136d2d85738",
    "operation": "parse_file"
  },
  "result": {},
  "error": null,
  "created_at": "2026-05-15T10:45:00Z",
  "updated_at": "2026-05-15T10:45:00Z",
  "started_at": null,
  "finished_at": null
}
```

## List jobs

```text
GET /api/jobs
```

Authentication: session cookie or bearer token with `notes\:read` or `admin`.

Admins can see all jobs. Normal users see only their own jobs.

Query parameters:

| Parameter | Description |
|---|---|
| `note_id` | Return jobs related to one note. |
| `status` | Return jobs with a specific status. |

## Get job

```text
GET /api/jobs/{job_id}
```

Authentication: session cookie or bearer token with `notes\:read` or `admin`.

## Job statuses

```text
queued
running
succeeded
failed
cancelled
```

Agents should poll the job endpoint when file or URL parsing is pending.
