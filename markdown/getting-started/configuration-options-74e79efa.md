# Configuration options

Created: 2026-05-31T09:19:04Z

## Note

Mianotes configuration uses `settings.json` for project defaults and environment variables with the `MIANOTES_` prefix for secrets and local overrides. For normal local use, configure only the LLM provider and API key.

## `settings.json`

The web service reads `settings.json` from the service root when it starts. Environment variables override values from this file.

Example:

```json
{
  "server": {
    "host": "127.0.0.1",
    "port": 8200
  },
  "database": {
    "adapter": "sqlite",
    "url": "env.MIANOTES_DATABASE_URL"
  },
  "llm": {
    "provider": "openai",
    "model": "gpt-5-nano",
    "baseUrl": "",
    "apiKey": "env.MIANOTES_LLM_API_KEY"
  },
  "vlm": {
    "provider": "openai",
    "model": "gpt-5-nano",
    "baseUrl": "",
    "apiKey": "env.MIANOTES_VLM_API_KEY"
  }
}
```

Values beginning with `env.` are read from the process environment or `.env`. Do not put raw secrets in `settings.json`.

The database adapter is currently `sqlite`. If `MIANOTES_DATABASE_URL` is unset, Mianotes uses the default local `data/system.db`.

## Common variables

| Variable                                     | Default                     | Description                                                             |
| -------------------------------------------- | --------------------------- | ----------------------------------------------------------------------- |
| `MIANOTES_HOST`                              | `0.0.0.0` or script default | Host interface for the web service. Use `127.0.0.1` for local-only use. |
| `MIANOTES_PORT`                              | `8200`                      | Web service port.                                                       |
| `MIANOTES_LLM_PROVIDER`                      | `openai`                    | LLM provider: `openai`, `local`, or `openai-compatible`.                |
| `MIANOTES_LLM_MODEL`                         | `gpt-5-nano`                | Text model used by Mia.                                                 |
| `MIANOTES_LLM_BASE_URL`                      | empty                       | Base URL for local or OpenAI-compatible providers.                      |
| `MIANOTES_LLM_API_KEY`                       | empty                       | LLM provider API key or local placeholder.                              |
| `MIANOTES_VLM_API_KEY`                       | empty                       | Optional image-capable model provider key.                              |
| `MIANOTES_LLM_IMAGE_MODEL`                   | empty                       | Optional multimodal OpenAI model for image OCR fallback.                |
| `MIANOTES_API_KEY`                           | empty                       | Service-wide bearer token used by local agents and MCP.                 |
| `MIANOTES_MAX_PUBLISHED_SITE_DOWNLOAD_BYTES` | `262144000`                 | Maximum bytes allowed for a published-site ZIP download.                |
| `MIANOTES_MAX_PUBLISHED_SITE_DOWNLOAD_FILES` | `5000`                      | Maximum file count allowed in a published-site ZIP download.            |

## Ports

Mianotes services use the `8200` range:

```text
8200  web service
8201  alternate local service
```

To start on another port during development:

```bash
MIANOTES_HOST=127.0.0.1 MIANOTES_PORT=8201 ./start-dev.sh
```

## Storage defaults

Generated notes and source files live inside each workspace folder:

```text
<workspace>/markdown/<folder_slug>/<title_slug>-<note_id[:8]>.md
<workspace>/markdown/<folder_slug>/sources/<note_id[:8]>/original.<ext>
```

Folder directories are filesystem-safe slugs and are unique inside a workspace. Source files live under each folder's `sources/` directory.

## LLM provider values

### OpenAI

```env
MIANOTES_LLM_PROVIDER=openai
MIANOTES_LLM_MODEL=gpt-5-nano
MIANOTES_LLM_API_KEY=sk-******
```

### Local Ollama-style provider

```env
MIANOTES_LLM_PROVIDER=local
MIANOTES_LLM_MODEL=llama3.2:3b
MIANOTES_LLM_BASE_URL=http://127.0.0.1:11434/v1
MIANOTES_LLM_API_KEY=ollama
```

### Other OpenAI-compatible endpoint

```env
MIANOTES_LLM_PROVIDER=openai-compatible
MIANOTES_LLM_MODEL=<model-name>
MIANOTES_LLM_BASE_URL=<base-url>
MIANOTES_LLM_API_KEY=<token-or-local-placeholder>
```

## Agent access variables

The service-wide key is for trusted local agents and MCP servers:

```env
MIANOTES_API_URL=http://127.0.0.1:8200
MIANOTES_API_KEY=<api_key_generated_in_settings>
```

Agent clients send it as:

```http
Authorization: Bearer <token>
```

Scoped per-user API tokens are available through `/api/tokens` when an automation needs narrower permissions.
