# MCP server

Created: 2026-06-04T15:45:49Z

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

The MCP command reads the Mianotes API URL and API key from the environment. The easiest setup is to create an install script in Mianotes Settings and run it on the same machine as your MCP client. That command writes the values to `~/.mianotes/env`.

Set `MIANOTES_CLIENT_NAME` in the MCP client configuration when you want Mianotes to attribute jobs and notes to a specific tool.

## Authentication

By default, MCP clients use the user API token installed from Mianotes Settings.

```env
MIANOTES_API_URL=http://mianotes.local:8200
MIANOTES_API_KEY=<generated_by_the_install_script>
MIANOTES_API_USER=user@example.com
```

Codex and Claude do not need to pass `MIANOTES_API_URL` or `MIANOTES_API_KEY` to the MCP command if the process can read the installed environment.

On startup, the MCP server exchanges `MIANOTES_API_KEY` for a short-lived agent session:

```http
POST /api/auth/agent-session
Authorization: Bearer <MIANOTES_API_KEY>
X-Mianotes-Client: Codex
```

The MCP server then uses the returned session token for tool calls. The session token contains the mapped client identity and token reference, not the raw API key. Unknown client names default to `MCP`.

Create a fresh install script from Settings when you need to regenerate a user's agent token.

## Configure `Codex`

Codex needs a stdio MCP server entry that starts `mianotes-mcp`.

**Step 1**. Make sure Mianotes is running.

**Step 2**. Create an install script in Mianotes Settings and run it on the same machine where Codex runs.

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

## Configure `Claude Code`

Claude Code also needs a stdio MCP server entry.

**Step 1**. Make sure Mianotes is running.

**Step 2**. Create an install script in Mianotes Settings and run it on the same machine where Claude Code runs.

**Step 3**. Install [Claude Code CLI](https://code.claude.com/docs/en/setup) and the [Claude Code MCP server](https://code.claude.com/docs/en/mcp) plugin. After installation completes, open a terminal in the project you want to work in and start Claude Code:

```text
claude
```

**Step 4**. Add the MCP server:

* If you installed Mianotes from package, use command `mianotes-mcp`.
* If you installed it from source, use command `/path/to/mianotes-web-service/.venv/bin/mianotes-mcp`.

```bash
claude mcp add-json mianotes '{"type":"stdio","command":"mianotes-mcp","env":{"MIANOTES_CLIENT_NAME":"Claude"}}'
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
