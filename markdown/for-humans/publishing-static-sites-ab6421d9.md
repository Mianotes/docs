# Publishing static sites

Created: 2026-05-30T17:57:16Z

## Note

Mianotes can publish selected notes as a static documentation site. This is useful when a folder has become reference material, project documentation, or a small knowledge base that should be browsed outside the main app.

Publishing creates HTML files under the active data folder. It does not replace the original Markdown notes or their source files.

## Publish from the web app

1. Open the Mianotes web app.
2. Click **Publish** in the sidebar.
3. Choose all folders, one folder, or another available scope.
4. Review the generated navigation.
5. Review the site configuration.
6. Click **Publish**.

After publishing, Mianotes shows actions to preview the static site in your browser or download the generated site as a ZIP file.

## Navigation and new notes

The **Navigation** block always contains every note that will appear in the static site for the selected scope. It is regenerated from the current publishable notes each time you open the publish draft.

The **New notes** block is only a change summary. It lists notes whose generated paths were not present in the previous publish navigation. If you publish again without adding notes, **New notes** should be empty.

## Site configuration

```json
{
  "brand": "Mianotes",
  "version": "1.0.0",
  "headerLinks": [
    {
      "title": "GitHub",
      "url": "https://github.com/Mianotes"
    },
    {
      "title": "Website",
      "url": "https://www.mianotes.com/"
    }
  ],
  "showPreviousVersions": true,
  "footerHtml": "(c) Copyright Mianotes."
}
```

| Key                    | Description                                                            |
| ---------------------- | ---------------------------------------------------------------------- |
| `brand`                | Name shown in the static site header.                                  |
| `version`              | Version folder used for the generated site, such as `1.0.0`.           |
| `headerLinks`          | External links shown after version links.                              |
| `showPreviousVersions` | When `true`, show up to the latest 3 published versions in the header. |
| `footerHtml`           | Footer HTML shown at the bottom of article pages.                      |

## Themes

Mianotes ships two static publishing themes:

| Theme ID   | Name     | Use                                |
| ---------- | -------- | ---------------------------------- |
| `mialight` | Mialight | Default light documentation theme. |
| `miadark`  | Miadark  | Dark documentation theme.          |

Both themes render Markdown tables, fenced code blocks, admonitions, article navigation, search, and an **On this page** column generated from page headings.

## Markdown rendering

The static renderer converts common Markdown into HTML before writing the site. This includes headings, paragraphs, bullet lists, fenced code blocks, Markdown tables, inline code, strong text, links, and GitHub-style admonitions such as `[!TIP]` and `[!WARNING]`.

If the first Markdown heading repeats the note title, Mianotes removes the duplicate heading so article pages do not show the title twice.

## Generated files

A published version is written under:

```text
data/html/<version>/
```

The public root redirect and version navigation live under:

```text
data/html/index.html
data/html/navigation.js
```

Generated HTML includes:

```html
<meta name="generator" content="Mianotes - https://github.com/Mianotes">
```

## Downloading the site

The publish response includes a `download_url`. The ZIP archive contains the version directory plus root files needed for version navigation.

```text
<version>-static-site/
  index.html
  navigation.js
  <version>/
    index.html
    styles.css
    site.js
    search.js
```

## Serving published files

Static HTML is public under `/html/...`.

Published Markdown is available under `/markdown/...`. Authenticated users and valid bearer-token clients can read stored Markdown files. Public callers can only read Markdown or source files that belong to notes marked as published.

Database files are never served.
