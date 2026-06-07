# MCP server

Created: 2026-06-07T14:27:27Z

## Note

Mianotes ships a stdio MCP server for compatible AI agents. The MCP server calls the same REST API as other agent clients, so normal token scopes and backend permission checks still apply.

## Install Script

The easiest setup is to create an `Install Script` in Mianotes `Settings > Connect tools` and run it on the same machine as your MCP client.

That command writes these API values to `~/.mianotes/env`:

```env
MIANOTES_API_URL=http://mianotes.local:8200
MIANOTES_API_KEY=<generated_by_the_install_script>
MIANOTES_API_USER=user@example.com
```

MCP clients can then start `mianotes-mcp` without passing API credentials manually.

On startup, the MCP server exchanges `MIANOTES_API_KEY` for a short-lived agent session.

The MCP server then uses the returned session token for tool calls. Mianotes identifies the user from the API key, so MCP requests use the same user permissions and workspace access as direct API requests.

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
codex mcp add mianotes -- mianotes-mcp
```

**Step 6**. Verify the server is configured:

```bash
codex mcp list
```

**Step 7**. Within Codex (desktop app), ask Mia to do something:

```text
Mia, list all the folders in the main workspace
```

## Configure `Claude Code`

Claude Code also needs a stdio MCP server entry.

**Step 1**. Make sure Mianotes is running.

**Step 2**. Create an install script in Mianotes Settings and run it on the same machine where Claude Code runs.

**Step 3**. Install [Claude Code CLI](https://code.claude.com/docs/en/setup). Claude Code supports MCP servers, so you can add Mianotes as a stdio server. After installation completes, open a terminal in the project you want to work in and start Claude Code:

```text
claude
```

**Step 4**. Add the MCP server:

* If you installed Mianotes from package, use command `mianotes-mcp`.
* If you installed it from source, use command `/path/to/mianotes-web-service/.venv/bin/mianotes-mcp`.

```bash
claude mcp add-json mianotes '{"type":"stdio","command":"mianotes-mcp"}'
```

**Step 5**. Verify the server is configured:

```bash
$ claude mcp get mianotes

mianotes:
  Scope: Local config (private to you in this project)
  Status: ✓ Connected
  Type: stdio
  Command: mianotes-mcp
```

**Step 6**. Within Claude Code (desktop app), check server status:

```text
/mcp
```

The Mianotes server should appear in the MCP server list. Then ask Claude Code to use Mia, for example:

```text
Mia, list all the folders in the main workspace
```

## MCP tools

The MCP surface includes these tools:

| Tool                    | Purpose                                                    |
| ----------------------- | ---------------------------------------------------------- |
| `list_workspaces`       | List configured Mianotes workspaces.                       |
| `list_folders`          | List folders in a workspace.                               |
| `create_folder`         | Create a folder.                                           |
| `list_notes`            | List notes.                                                |
| `read_note_context`     | Read a note by workspace, folder name, and note title.     |
| `get_note`              | Get a note by ID.                                          |
| `create_note`           | Create a note from text using a folder ID.                 |
| `create_note_in_folder` | Create a note from text using a workspace and folder name. |
| `create_note_from_url`  | Create a note from a URL and queue parsing.                |
| `update_note`           | Update a note.                                             |
| `set_tags`              | Replace a note's tags.                                     |
| `search_notes`          | Search Markdown notes.                                     |

Most tools accept a `workspace` argument. Agents should pass the workspace name when the user provides one. If omitted, Mianotes uses the current or default workspace.

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
