# What is Mianotes?

Created: 2026-05-31T12:58:05Z

## Note

:::note
**Note**
This documentation was written by engineers and Codex agents using [Mianotes](https://www.mianotes.com/). If you find anything wrong, unclear, or misleading, please open an issue on [GitHub](https://github.com/Mianotes/mianotes-web-service/issues) so we can fix it. Thank you.
:::

Mianotes is a local-first knowledge repository for people and AI agents.

It converts notes, links, documents, meeting recordings, videos, articles, images, and AI output into clean Markdown you can search, organise, edit, and reuse. People use the web app to read, edit, tag, comment on, and share notes. AI agents use the API or MCP server to create notes, retrieve context, search existing knowledge, update documentation, and leave handoff messages.

Mianotes is built with open-source tools, including Microsoft MarkItDown and FFmpeg for file conversion and Ollama for local AI models.

## Why it exists

Mianotes exists because project knowledge is no longer created by people alone.

Teams collect research, links, documents, meeting recordings, screenshots, notes, and AI outputs as they work. At the same time, AI agents are writing code, making decisions, fixing bugs, generating summaries, proposing changes, and producing context that often disappears when the session ends.

The problem is not just where information is stored. The problem is that people and agents do not have a shared, consistent way to capture it, structure it, and reuse it later.

Mianotes gives both people and agents a local place to write things down.

A developer can save source material, meeting notes, research, and project context. An agent can document what it changed, explain why it made a decision, ask questions, leave handoff notes, or save useful findings before its session ends. Another agent can then search that same knowledge and continue from the right context.

Mianotes also helps you collect information from many sources quickly, including files, links, audio, images, videos, and AI responses, then convert that material into clean Markdown that people can read and agents can use.

The goal is to standardise how project knowledge moves between people, tools, and agents, without losing control of where that knowledge lives.

## What Mianotes stores

Mianotes keeps user-facing note content as Markdown files under the active data folder:

```text
<workspace>/markdown/<folder_slug>/<title_slug>-<note_id[:8]>.md
```

The workspace SQLite database lives inside the hidden runtime folder:

```text
<workspace>/.mianotes/mia.db
```

SQLite stores indexes and metadata such as users, folders, tags, notes, comments, sessions, and jobs. The text stays in a Markdown file.

## Meet Mia

Mia is the built-in Mianotes agent. Mia helps convert messy input into durable Markdown notes, improve structure, extract useful information, summarise long content, and prepare notes for reuse by humans or other agents.
