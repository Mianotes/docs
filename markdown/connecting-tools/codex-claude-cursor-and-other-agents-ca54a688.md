# Codex, Claude, Cursor, and other agents

Created: 2026-06-07T14:29:31Z

## Note

Mianotes is intended to be useful for Codex, Claude Code, OpenClaw, Cursor, VS Code agents, Slack bots, and local automation.

The core idea is the same for every tool: give the agent safe Mianotes API credentials and clear instructions, then ask it to write durable notes as it works.

## Installed agent instructions

The Mianotes setup scripts install local Mianotes skill files for Codex and Claude:

```text
~/.codex/skills/mianotes/SKILL.md
~/.claude/skills/mianotes/SKILL.md
```

The web app can generate a one-time install script that writes the agent environment to `~/.mianotes/env` and installs the skill files for Codex and Claude.

## Environment for agents

Agents need access to the API URL and user token:

```env
MIANOTES_API_URL=http://mianotes.local:8200
MIANOTES_API_KEY=<generated_by_the_install_script>
MIANOTES_API_USER=user@example.com
```

Generate the install URL from Mianotes Settings and run the command on the machine where the agent runs. The command can be used once and expires after 24 hours.

## MCP-capable agents

For MCP-capable clients, configure the client to start:

```bash
mianotes-mcp
```

The MCP server reads `MIANOTES_API_URL` and `MIANOTES_API_KEY`. It exchanges the API key for a short-lived agent session, then calls the REST API with that session token.

## REST-capable agents

Any agent that can make HTTP requests can use the REST API directly.

Minimal search example:

```bash
MIANOTES_SESSION_TOKEN="$(
  curl -sS -X POST \
    -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
    "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/auth/agent-session" \
    | python3 -c 'import json, sys; print(json.load(sys.stdin)["token"])'
)"

curl -sS \
  -H "Authorization: Bearer ${MIANOTES_SESSION_TOKEN}" \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/search?q=settings"
```

Minimal note creation example:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_SESSION_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "folder_id": "<folder-id>",
    "title": "Agent worklog",
    "text": "# Agent worklog\n\nThe agent started documenting this task."
  }' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/notes/from-text"
```

## Safety rule

Treat any agent with a valid Mianotes token as trusted. Mianotes can authenticate and audit the caller, but it cannot sandbox an agent that already has filesystem access to sensitive local files.
