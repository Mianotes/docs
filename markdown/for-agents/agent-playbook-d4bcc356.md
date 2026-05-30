# Agent playbook

Created: 2026-05-30T20:09:34Z

## Note

This page gives practical patterns for agents using Mianotes.

## Before starting work

1. Search Mianotes for relevant context.
2. Read the most relevant notes fully.
3. Create a new note if the task is new.
4. Add tags that will help other agents find it.

Example search:

```bash
curl -sS \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/search?q=api%20token&limit=10"
```

## After file or URL ingestion

If the agent creates a note from a file or URL, the response will include a `job_api_url`. Poll the job until it succeeds.

User prompts may look like:

```text
Import this link into Mia(workspace: Research, folder: Sources):
https://example.com/article
```

```text
Read this PDF and save the key points in Mia(workspace: My App, folder: Research).
```

```text
Transcribe this YouTube video and save it in Mia(workspace: Course Notes, folder: Lectures):
https://www.youtube.com/watch?v=...
```

```bash
curl -sS \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  "${job_api_url}"
```

Then call the note URL to read the final Markdown.
