# Core concepts

Created: 2026-05-30T17:23:44Z

## Note

This page explains the words used throughout the documentation.

## `Workspace`

A Mianotes workspace is a top-level knowledge area backed by a local folder. The workspace folder contains Markdown notes, source files, and published output. The private SQLite database for that workspace lives in the Mianotes data directory at `data/workspaces/<workspace_id>.db`.

Small groups can share a Mianotes app by using the same master password. Everyone can read shared notes, browse by folder, browse by user, and add notes to active folders.

## `User`

A user is a human account in the current Mianotes folder. The first user becomes the admin for that folder.

Browser users authenticate with a session cookie. Agents should use API tokens instead of browser sessions.

## `Folder`

A folder is a shared workspace for related notes.

Folder rows are stored in SQLite, while each folder also owns a filesystem directory under the workspace `markdown/` directory. A folder can be archived. When archived, the folder is hidden from normal lists and moved under the workspace archive area so active folders stay clean.

## `Note`

A note is the main unit of knowledge.

The note body is a Markdown file on disk. Metadata lives in SQLite. A note can have source files, tags, share links, and job history.

A note filename normally looks like:

```text
<title_slug>-<note_id>.md
```

Example:

```text
planning-trip-to-mallorca-4a95f146.md
```

## `Source file`

A source file is the original file or downloaded HTML used to create a note.

For file uploads, Mianotes stores the uploaded file under:

```text
<workspace>/markdown/<folder_slug>/sources/<note_id[:8]>/original.<ext>
```

Source files are kept next to the generated Markdown note but ignored by the folder-level `.gitignore` so Git backups can store notes without committing original uploads.

## `Tag`

Tags provide cross-folder grouping. Notes can be tagged with short labels such as `research`, `planning`, `client`, or `release`.

Tags are useful for both humans and agents because they make related notes easier to discover.

## `Job`

A job represents background work, such as parsing an uploaded file or indexing a URL.

Jobs move through clear status transitions:

```text
queued -> running -> succeeded
queued -> running -> failed
queued -> cancelled
```

Clients should poll the job endpoint when a file or URL note is still being parsed.

## `Mia`

Mia is the built-in AI assistant. Mia works through backend service boundaries, not frontend-only behaviour. The backend owns permissions, persistence, parsing, and the LLM provider calls.
