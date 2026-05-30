# Sharing and comments

Created: 2026-05-30T18:06:01Z

## Note

Mianotes supports shared discussion inside the workspace and read-only note sharing outside it.

## Share links

Share links are note-level and read-only.

Use a share link when someone outside the Mianotes workspace needs access to one note but should not join the whole workspace.

Important properties:

* a share link does not grant full workspace access
* share links can be revoked
* admins can share any note
* normal users can share notes they created

## Source files on shared notes

When sharing is enabled, a guest-readable shared source file endpoint can return source files attached to the shared note.&#x20;

Only share notes whose source files are safe for the recipient to see.

## Practical sharing checklist

Before sharing a note:

1. Read the full Markdown note.
2. Check attached source files.
3. Remove private comments or copy only the final content into a clean note.
4. Generate the share link.
5. Revoke the link when it is no longer needed.

## Prompting Mia from API comments

If the comment body starts with `@mia`, Mianotes treats it as a private prompt instead of a shared comment.

Example:

```json
{
  "body": "@mia summarise this note in five bullet points"
}
```

Mianotes strips the `@mia` prefix, reads the current note Markdown, sends the prompt and note content to the configured LLM provider, and returns Markdown directly.

Mia prompts:

* are synchronous
* are private
* are not saved as comments
* do not create jobs
* do not update the note
* return Markdown only
