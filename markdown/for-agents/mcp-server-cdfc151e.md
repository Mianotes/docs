# MCP server

Created: 2026-05-31T13:02:25Z

## Note

Mianotes ships a stdio MCP server for compatible AI agents. The MCP server calls the same REST API as other agent clients, so normal token scopes and backend permission checks still apply.

## Run the MCP server

If you installed Mianotes from a package, use the packaged MCP command:

```bash
mianotes-mcp
```

If you installed Mianotes from source, run the wrapper from the web service checkout:

```bash
cd /path/to/mianotes-web-service
./scripts/mianotes-mcp.sh
```

The source wrapper uses the project virtual environment, so it does not depend on whichever global `python3` is installed on your machine.

The MCP process needs `MIANOTES_API_KEY` in its environment. Set `MIANOTES_CLIENT_NAME` so Mianotes can attribute jobs and notes to the calling tool.

## Authentication

By default, MCP clients use the service-wide `MIANOTES_API_KEY`.

```env
MIANOTES_API_URL=http://127.0.0.1:8200
MIANOTES_API_KEY=<generated_in_settings>
MIANOTES_CLIENT_NAME=Codex
```

On startup, the MCP server exchanges `MIANOTES_API_KEY` and `MIANOTES_CLIENT_NAME` for a short-lived agent session:

```http
POST /api/auth/agent-session
Authorization: Bearer <MIANOTES_API_KEY>
X-Mianotes-Client: Codex
```

The MCP server then uses the returned session token for tool calls. The session token contains the mapped client identity and token reference, not the raw API key. Unknown client names default to `MCP`.

Use scoped per-user API tokens when an agent should only read notes or work in a limited role.

## Configure Codex

Codex needs a stdio MCP server entry that starts `mianotes-mcp` with the Mianotes API URL, API key, and client name in its environment.

1. Make sure Mianotes is running.
2. Create an API key in Mianotes Settings.
3. Check that the MCP command is available on the same machine where Codex runs:

```bash
command -v mianotes-mcp
```

If it is not in `PATH`, use the full path to the command, for example:

```text
/path/to/mianotes-web-service/.venv/bin/mianotes-mcp
```

1. Add the MCP server. If Codex runs on the same machine as Mianotes, use `http://127.0.0.1:8200`. If Codex runs somewhere else, use the Mianotes URL that machine can reach.

```bash
codex mcp add mianotes \
  --env MIANOTES_API_URL=http://127.0.0.1:8200 \
  --env MIANOTES_API_KEY=<generated_in_settings> \
  --env MIANOTES_CLIENT_NAME=Codex \
  -- mianotes-mcp
```

1. Verify the server is configured:

```bash
codex mcp list
```

1. Start a new Codex session. Ask Codex to use Mia, for example:

```text
Search Mia(workspace: Mianotes, query: Getting started)
```

If you use the Codex app settings screen instead of the CLI, add a custom stdio MCP server with:

```text
Name: Mianotes
Command to launch: mianotes-mcp
Environment variables:
  MIANOTES_API_URL=http://127.0.0.1:8200
  MIANOTES_API_KEY=<generated_in_settings>
  MIANOTES_CLIENT_NAME=Codex
```

Restart the Codex session after saving the server.

## Configure Claude Code

Claude Code also needs a stdio MCP server entry. The most reliable setup is `claude mcp add-json`, because it keeps the command and environment variables together.

1. Make sure Mianotes is running.
2. Create an API key in Mianotes Settings.
3. Make sure Claude Code is installed. If this command fails, install Claude Code first:

```bash
command -v claude
```

3. Check that the MCP command is available on the same machine where Claude Code runs:

```bash
command -v mianotes-mcp
```

If it is not in `PATH`, use the full path to the command in the JSON below.

1. Add the MCP server. If Claude Code runs on the same machine as Mianotes, use `http://127.0.0.1:8200`. If Claude Code runs somewhere else, use the Mianotes URL that machine can reach.

```bash
claude mcp add-json mianotes \
  '{"type":"stdio","command":"mianotes-mcp","env":{"MIANOTES_API_URL":"http://127.0.0.1:8200","MIANOTES_API_KEY":"<generated_in_settings>","MIANOTES_CLIENT_NAME":"Claude Code"}}'
```

1. Verify the server is configured:

```bash
claude mcp get mianotes
```

1. Start a new Claude Code session. Run:

```text
/mcp
```

The Mianotes server should appear in the MCP server list. Then ask Claude Code to use Mia, for example:

```text
Search Mia(workspace: Mianotes, query: Getting started)
```

MCP clients do not automatically inherit the web service `.env` file. If Codex or Claude Code says Mianotes is unreachable or unauthenticated, check that the MCP server configuration includes both `MIANOTES_API_URL` and `MIANOTES_API_KEY`.

## MCP tools

The MCP surface includes tools for:

* listing folders
* creating folders
* listing notes
* reading notes
* creating notes from text
* creating notes from URLs
* updating notes
* sending private synchronous `@mia` prompts through the comments endpoint
* setting tags
* searching Markdown notes.

## URL ingestion results

`create_note_from_url` returns the same ingestion response as the REST API. It does not wait for the page to be converted before returning.

The response includes:

* `note_id`
* `job_id`
* `job_status`
* `note_api_url`
* `job_api_url`
* the nested note object
* the nested job object

Agents should poll `job_api_url` until the job reaches `succeeded`, then call `get_note` with `note_id` to retrieve the finished Markdown content.

## File ingestion

File ingestion is available through the REST API:

```text
POST /api/notes/from-file
```

Use the REST file upload endpoint when an agent needs to send local files to Mia.

## Design rule

The MCP server stays thin. It does not read database files, bypass REST permissions, or write files directly. The API remains the single authority for permissions, parsing, job state, and persistence.
