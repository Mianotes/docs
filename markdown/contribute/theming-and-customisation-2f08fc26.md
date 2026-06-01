# Theming and customisation

Created: 2026-05-31T11:55:21Z

## Note

Mianotes has two kinds of customisation:

* runtime configuration for the web service, parser, storage, and model providers;
* static publishing themes used when notes are exported as documentation sites.

Treat the main web app visual design as application code. Static site publishing themes use a small theme boundary.

## Static publishing themes

Mianotes ships two static publishing themes:

| Theme ID   | Name     | Purpose                            |
| ---------- | -------- | ---------------------------------- |
| `mialight` | Mialight | Default light documentation theme. |
| `miadark`  | Miadark  | Dark documentation theme.          |
| `miadev`   | Miadev   | A theme for developers.            |
| `miadocs`  | Miadocs  | A calm light blue theme.           |

Theme assets live in:

```text
src/mianotes_web_service/publishing/themes/<theme-id>/
```

Each theme directory contains:

```text
theme.json
styles.css
site.js
```

The `theme.json` file defines the theme metadata returned by `GET /api/publish/themes,` `styles.css` controls the published site appearance. `site.js` handles static site behaviour such as navigation, search wiring, article state, and previous version links.

Static themes should render these Markdown features well:

* headings and article title hierarchy
* paragraphs, ordered lists, and unordered lists
* inline code and fenced code blocks
* Markdown tables
* links
* GitHub-style admonitions such as `[!TIP]`, `[!NOTE]`, and `[!WARNING]`
* the generated **On this page** column
* previous-version links when `showPreviousVersions` is enabled

When adding or changing a theme, run the publishing tests and manually publish a small site that includes code blocks, tables, admonitions, and enough headings to populate the right-hand article navigation.

```bash
pytest tests/test_api_publish.py
```

## Runtime customisation

Supported configuration areas:

* service host and port
* allowed storage locations
* LLM provider and model
* image OCR fallback model
* service-wide API key
* scoped agent tokens
* parser adapters

## Ports

```text
8200  web service
8201  alternate local service
```

## Workspace customisation

Default:

```text
data/
data/.mianotes/mia.db
```

Admins can switch between allowed local folders from the Settings screen.

## LLM customisation

Mia supports:

```text
openai
local
openai-compatible
```

Example local setup:

```env
MIANOTES_LLM_PROVIDER=local
MIANOTES_LLM_MODEL=llama3.2:3b
MIANOTES_LLM_BASE_URL=http://127.0.0.1:11434/v1
MIANOTES_LLM_API_KEY=ollama
```

## Web app theme conventions

Keep these decisions documented when changing the web app visual system:

* colour tokens
* typography scale
* spacing scale
* dark mode behaviour
* logo and app icon placement
* Markdown rendering styles
* note status badges
* accessibility contrast rules
* whether themes are per-user, per-workspace, or build-time only
