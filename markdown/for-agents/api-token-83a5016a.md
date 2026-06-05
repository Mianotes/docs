# API token

Created: 2026-05-30T18:19:05Z

## Note

Mianotes API clients authenticate with bearer tokens. A token lets an agent, script, or other tool call the same REST API that the web app uses.

The easiest way to get started is through the web app. Sign in, open **Settings**, and create an install script. Mianotes creates a one-time install link that expires after one hour. Run the command on the same machine as Codex, Claude Code, or another local tool.

```bash
curl -fsSL "http://mianotes.local:8200/skill/install.sh?code=<one_time_code>" | bash
```

The installer writes the connection details to `~/.mianotes/env` and installs the Mianotes skill for Codex and Claude:

```env
MIANOTES_API_URL="http://mianotes.local:8200"
MIANOTES_API_KEY="mia_apikey_..."
MIANOTES_API_USER="user@example.com"
```

```text
~/.codex/skills/mianotes/SKILL.md
~/.claude/skills/mianotes/SKILL.md
```

The downloaded shell script does not display the API key. It exchanges the one-time code while it runs, writes the key to `~/.mianotes/env`, and marks the code as used. Rerun the Settings action if you need a new command.

Agents use that value to create an agent session:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  -H "X-Mianotes-Client: Codex" \
    "${MIANOTES_API_URL}/api/auth/agent-session"
```

The response contains a short-lived bearer token. Use that session token for follow-up requests. Mianotes maps known client names from `X Mianotes-Client` to a stable client ID, stores that identity inside the signed token, and uses it to attribute jobs to the right tool in the Console. Unknown client names default to `MCP`. The raw API key is never embedded in the session token.

## How Mianotes stores the key

The raw key is a secret. Treat it like a password.

Mianotes compares bearer tokens by hashing the presented token and comparing it with the public verifier stored in `data/system.db`. The database does not store the raw key.

There are two normal places the raw key lives:

* the one-time installer writes it to `~/.mianotes/env` on the agent machine
* tools can receive the same variables through their own environment

On install script redemption, Mianotes stores only the derived public verifier in `data/system.db`. The generated token belongs to the signed-in user and keeps working until that user regenerates it or revokes it.

## Get a key in the web app

This is the recommended path for most people because it avoids hand-writing API requests.

1. Start the Mianotes web service.
2. Open the Mianotes web app.
3. Sign in as an admin user.
4. Open **Settings**.
5. Click **Create install script**.
6. Copy and run the command on the machine where your local agent runs.
7. Open a new terminal session so the exported environment is available.

For a local shell:

```bash
export MIANOTES_API_URL="http://127.0.0.1:8200"
export MIANOTES_API_KEY="generated_by_the_install_script"
export MIANOTES_API_USER="user@example.com"
```

For a project `.env` file:

```env
MIANOTES_API_URL=http://127.0.0.1:8200
MIANOTES_API_KEY=<generated_by_the_install_script>
MIANOTES_API_USER=user@example.com
```

Then create an agent session:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  -H "X-Mianotes-Client: Codex" \
  "${MIANOTES_API_URL}/api/auth/agent-session"
```

Use a client name that describes the tool, such as `Codex`, `Claude`, `Cursor`, or `Slack`. Mianotes maps known names and aliases to a stable client ID for Console display. Unknown names are accepted and shown as `MCP`.

## Get a key from the API

You can also create the service API key directly from the REST API. Use this when you are setting up Mianotes from a script or when the web app is not available.

You must call this endpoint as an admin. The simplest way is to sign in through the browser first, then send the request with the browser session cookie.

If you are scripting the whole flow, first resolve the admin user's ID:

```bash
curl -sS \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com"}' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/auth/check-email"
```

Then sign in and save the session cookie:

```bash
curl -sS \
  -X POST \
  -c cookies.txt \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "paste-user-id-from-check-email",
    "password": "your-admin-password"
  }' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/auth/login"
```

Then create a one-time skill installer with that saved cookie. The endpoint returns a command, not the raw API key:

```bash
curl -sS \
  -X POST \
  -b cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"api_url":"http://127.0.0.1:8200","client_name":"Codex"}' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/install/skill"
```

The response contains a short-lived install command:

```json
{
  "install_url": "http://127.0.0.1:8200/skill/install.sh?code=abc123",
  "command": "curl -fsSL \"http://127.0.0.1:8200/skill/install.sh?code=abc123\" | bash",
  "expires_at": "2026-06-05T12:00:00Z"
}
```

Run the command on the agent machine. When the script runs, Mianotes creates the per-user API token and writes it to `~/.mianotes/env`.

After the installer has run, exchange the API key for a short-lived agent session:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  -H "X-Mianotes-Client: Codex" \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/auth/agent-session"
```

If you already have a bearer token for your user, you can create a one-time skill installer without a browser cookie:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"api_url":"http://127.0.0.1:8200","client_name":"Codex"}' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/install/skill"
```

## Use the token from JavaScript

```js
const apiUrl = process.env.MIANOTES_API_URL ?? "http://127.0.0.1:8200";
const apiKey = process.env.MIANOTES_API_KEY;

const sessionResponse = await fetch(`${apiUrl}/api/auth/agent-session`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${apiKey}`,
    "X-Mianotes-Client": "Codex",
  },
});

if (!sessionResponse.ok) {
  throw new Error(`Mianotes API returned ${sessionResponse.status}`);
}

const { token } = await sessionResponse.json();
const response = await fetch(`${apiUrl}/api/search?q=settings`, {
  headers: { Authorization: `Bearer ${token}` },
});
const results = await response.json();
console.log(results);
```

## Use the token from Python

```python
import json
import os
import urllib.parse
import urllib.request

api_url = os.environ.get("MIANOTES_API_URL", "http://127.0.0.1:8200")
api_key = os.environ["MIANOTES_API_KEY"]
query = urllib.parse.urlencode({"q": "settings"})

request = urllib.request.Request(
    f"{api_url}/api/auth/agent-session",
    data=b"{}",
    method="POST",
    headers={
        "Authorization": f"Bearer {api_key}",
        "X-Mianotes-Client": "Codex",
        "Content-Type": "application/json",
    },
)

with urllib.request.urlopen(request) as response:
    session_token = json.loads(response.read().decode("utf-8"))["token"]

search_request = urllib.request.Request(
    f"{api_url}/api/search?{query}",
    headers={"Authorization": f"Bearer {session_token}"},
)

with urllib.request.urlopen(search_request) as response:
    print(response.read().decode("utf-8"))
```

## Use the token with MCP

The bundled MCP server reads the same environment variables:

```bash
export MIANOTES_API_URL="http://127.0.0.1:8200"
export MIANOTES_API_KEY="generated_by_the_install_script"
mianotes-mcp
```

If your MCP client starts the server from a different shell, make sure that shell can see the same environment variables.

## User tokens and scoped tokens

`MIANOTES_API_KEY` is the key installed for the current user. It is best for trusted local agents, MCP servers, and scripts that should act as that user.

Mianotes also supports scoped per-user API tokens through `/api/tokens`. Use scoped tokens when a tool should have narrower permissions, such as read-only note access.

Create a scoped token:

```bash
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Read-only search client",
    "scopes": ["notes:read", "folders:read", "tags:read"]
  }' \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/tokens"
```

Example response:

```json
{
  "id": "8d70ce3f-e0b9-47f8-a72d-3ad3cc3cc61f",
  "name": "Read-only search client",
  "token_prefix": "mia_abc12345",
  "scopes": ["notes:read", "folders:read", "tags:read"],
  "token": "mia_raw_scoped_token_returned_once"
}
```

The raw scoped token is also returned only once.

## Common checks

Check that your shell has the key:

```bash
test -n "${MIANOTES_API_KEY}" && echo "Mianotes API key is set"
```

Check that the service is reachable:

```bash
curl -sS "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/health"
```

Check authentication:

```bash
curl -i \
  -H "Authorization: Bearer ${MIANOTES_API_KEY}" \
  "${MIANOTES_API_URL:-http://127.0.0.1:8200}/api/auth/session"
```

If you get `401`, the bearer token is missing or invalid.

If you get `403`, the token was accepted but does not have the required scope
for that endpoint.

## Keep tokens private

Do not commit API tokens to git. Do not paste real tokens into examples, screenshots, issues, or logs.

If a token may have been exposed, create a new key and update the environments used by your agents and tools.
