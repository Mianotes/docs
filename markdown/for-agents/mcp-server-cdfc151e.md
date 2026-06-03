# MCP server

Created: 2026-06-03T15:30:01Z

## Note

Mianotes ships a stdio MCP server for compatible AI agents. The MCP server calls the same REST API as other agent clients, so normal token scopes and backend permission checks still apply.

## Run the MCP server

If you installed Mianotes from a package, use the packaged MCP command:

```bash
mianotes-mcp
```

If you installed Mianotes from source, run the wrapper from the web service local repo:

```bash
/path/to/mianotes-web-service/.venv/bin/mianotes-mcp
```

The MCP command loads the Mianotes API URL and API key automatically from the service environment file. Package installs use the packaged `mianotes.env`; source installs use the web service `.env` next to the `.venv` folder.

Set `MIANOTES_CLIENT_NAME` in the MCP client configuration when you want Mianotes to attribute jobs and notes to a specific tool.

## Authentication

By default, MCP clients use the service-wide `MIANOTES_API_KEY` saved by Mianotes Settings.

```env
MIANOTES_API_URL=http://127.0.0.1:8200
MIANOTES_API_KEY=<generated_in_settings>
MIANOTES_CLIENT_NAME=Codex
```

Codex and Claude do not need to pass `MIANOTES_API_URL` or `MIANOTES_API_KEY` to the MCP command. Pass only `MIANOTES_CLIENT_NAME` if you want a friendly client name in Mianotes.

On startup, the MCP server exchanges `MIANOTES_API_KEY` and `MIANOTES_CLIENT_NAME` for a short-lived agent session:

```http
POST /api/auth/agent-session
Authorization: Bearer <MIANOTES_API_KEY>
X-Mianotes-Client: Codex
```

The MCP server then uses the returned session token for tool calls. The session token contains the mapped client identity and token reference, not the raw API key. Unknown client names default to `MCP`.

Use scoped per-user API tokens when an agent should only read notes or work in a limited role.

## Configure Codex

Codex needs a stdio MCP server entry that starts `mianotes-mcp`. The MCP command loads the Mianotes API URL and API key automatically.

**Step 1**. Make sure Mianotes is running.

**Step 2**. Create an API key in Mianotes Settings. Mianotes writes it to the service environment file.

**Step 3**. Check that the MCP command is available on the same machine where Codex runs:

```bash
command -v mianotes-mcp
```

If it is not in `PATH`, use the full path to the command, for example:

```text
/path/to/mianotes-web-service/.venv/bin/mianotes-mcp
```

**Step 4**. Install [Codex CLI](https://developers.openai.com/codex/cli)

**Step 5**. Add the MCP server:

* If you installed Mianotes from package, use command `mianotes-mcp`.
* If you installed it from source, use command `/path/to/mianotes-web-service/.venv/bin/mianotes-mcp`.

```bash
codex mcp add mianotes --env MIANOTES_CLIENT_NAME=Codex -- mianotes-mcp
```

**Step 6**. Verify the server is configured:

```bash
codex mcp list
```

**Step 7**. Within Codex (desktop app), ask Mia to search for a note:

```text
Search Mia(workspace: Mianotes, query: Getting started)
```

## Configure Claude Code

Claude Code also needs a stdio MCP server entry. The most reliable setup is `claude mcp add-json`, because it keeps the command and environment variables together.

**Step 1**. Make sure Mianotes is running.

**Step 2**. Create an API key in Mianotes Settings. Mianotes writes it to the service environment file.

**Step 3**. Install [Claude Code CLI](https://code.claude.com/docs/en/setup). After installation completes, open a terminal in the project you want to work in and start Claude Code:

```text
claude
```

**Step 4**. Add the MCP server:

* If you installed Mianotes from package, use command `mianotes-mcp`.
* If you installed it from source, use command `/path/to/mianotes-web-service/.venv/bin/mianotes-mcp`.

```bash
claude mcp add-json mianotes \
  '{"type":"stdio","command":"mianotes-mcp","env":{"MIANOTES_CLIENT_NAME":"Claude"}}'
```

**Step 5**. Verify the server is configured:

```bash
$ claude mcp get mianotes

mianotes:
  Scope: Local config (private to you in this project)
  Status: ✓ Connected
  Type: stdio
  Command: mianotes-mcp
  Args:
  Environment:
    MIANOTES_CLIENT_NAME=Claude
```

**Step 6**. Within Claude Code (desktop app), check server status:

```text
/mcp
```

The Mianotes server should appear in the MCP server list. Then ask Claude Code to use Mia, for example:

```text
Search Mia(workspace: Mianotes, query: Getting started)
```

## MCP tools

The MCP surface includes tools for:

* listing folders
* creating folders
* listing notes
* reading notes
* creating notes from text
* creating notes from URLs
* updating notes
* setting tags
* searching notes.

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
