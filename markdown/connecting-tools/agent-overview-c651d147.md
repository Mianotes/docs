# Agent overview

Created: 2026-05-30T18:21:36Z

## Note

Mianotes is designed for collaboration between humans and AI agents.

AI agents write plans, explain changes, debug issues, summarise research, and leave useful context behind. But most of that work disappears inside temporary chats, IDE sidebars, Slack threads, and terminal sessions. Mianotes gives that work a place to live. Agents can save decisions, implementation notes, summaries, source links, files, images, and project context into clean Markdown notes, so the next agent can pick up the same context without asking you to explain everything again.

Agents should use Mianotes as a local documentation layer. They can save useful context while they work, then retrieve that context later through search, API calls, or supported tool integrations.

## What agents can do

An agent can:

* create folders for tasks, experiments, projects, or research threads
* add notes from text
* create notes from URLs
* upload files through the REST API
* convert documents, links, images, and audio into Markdown notes
* read notes
* update notes
* set tags
* search Markdown notes
* leave handoff messages
* send Mia prompts through the prompt endpoint when an AI provider is configured

## Recommended agent behaviour

Agents should document durable information, not every temporary thought.

Good agent notes include:

* decisions made
* relevant commands run
* files changed
* why an implementation changed
* bugs found and fixed
* test results
* next steps
* links to source material
* warnings about uncertainty

Avoid filling Mianotes with low-value logs that no human or agent will reuse.

## Access model

Agents should use the REST API, not browser cookies. The easiest setup is to open `Settings > Connect tools`, create an install script, and run that command on the machine where Codex, Claude Code, or another compatible tool runs.

The install script writes:

```env
MIANOTES_API_URL=http://127.0.0.1:8200
MIANOTES_API_KEY=<generated_by_the_install_script>
MIANOTES_API_USER=user@example.com
```

The values are saved in `~/.mianotes/env`, and the latest Mianotes `SKILL.md` is installed for Codex and Claude Code. The script never prints the API key.

`MIANOTES_API_KEY` belongs to the signed-in user who generated the install script. Every API request made with that key uses that user's permissions and workspace access. If the user revokes the key in Settings, tools using that key stop working until the user creates a new install script.

Agents can call the REST API directly with:

```http
Authorization: Bearer ${MIANOTES_API_KEY}
```

Tools that need a short-lived session can exchange the API key at `/api/auth/agent-session`, but direct bearer-token API calls are supported.

Browser sessions are for humans using the web app. Do not use browser cookies for agents.

## Workspace targeting

Mianotes can manage multiple workspaces. Agents should target the requested workspace explicitly instead of assuming the active browser workspace.

REST clients should send the workspace id with each workspace-content request:

```http
X-Mianotes-Workspace: <workspace-id>
```

MCP clients use the same backend permission model. When an MCP tool accepts a `workspace` argument, pass the requested workspace there instead.

Use conversational prompts when asking an agent to use Mianotes:

* Search the My Project workspace for deployment notes.
* Get context from the Docs folder in the My Project workspace.
* Search the Docs folder in the My Project workspace for publishing workflow.
* Read the Architecture note in the Docs folder of the My Project workspace.
* Save this in the Architecture folder of the My Project workspace.

Examples:

```text
Before answering, read the Architecture note in the Docs folder of the My Project workspace.
```

```text
Save our architecture discussion in the Architecture folder of the My App workspace.
```

```text
Read docs/ and document this app in the Documentation folder of the Mianotes workspace.
```

When saving into a Mianotes workspace and folder, create a useful title from the content if the user did not provide one.

## Converting source material

Agents can ask Mianotes to convert source material into Markdown notes.

Examples:

```text
Import this link into the Sources folder of the Research workspace:
https://example.com/article
```

```text
Read this PDF and save the key points in the Research folder of the My App workspace.
```

```text
Convert docs/product-spec.docx into a note in the Specs folder of the My App workspace.
```

```text
Transcribe this YouTube video and save it in the Lectures folder of the Course Notes workspace:
https://www.youtube.com/watch?v=...
```

Links and files are ingested as background jobs. Agents should create the note, poll the returned job URL until it succeeds, then fetch the final Markdown note.
