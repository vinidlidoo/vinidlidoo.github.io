# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zola static site blog using the **tabi** theme. Deploys to GitHub Pages at `vinidlidoo.github.io`.

## Commands

```bash
zola serve          # Dev server at http://127.0.0.1:1111 (live reload)
zola build          # Build to public/
zola check          # Validate site without building
```

## Project Structure

```
content/
├── _index.md           # Homepage (pulls recent posts from blog/)
└── blog/
    ├── _index.md       # Blog section config (English)
    ├── _index.fr.md    # Blog section config (French)
    ├── _index.ja.md    # Blog section config (Japanese)
    ├── *.md            # Blog posts (English)
    ├── *.fr.md         # Blog posts (French)
    └── *.ja.md         # Blog posts (Japanese)
config.toml             # Site config (base_url, theme, taxonomies, languages)
i18n/                   # UI string overrides (language_name, tags, newsletter, etc.)
templates/              # Template overrides (index.html, section.html, page.html)
templates/partials/     # Reusable partials (newsletter.html)
themes/tabi/            # Theme (git submodule)
```

## Creating Posts

Posts go in `content/blog/` with TOML frontmatter:

```markdown
+++
title = "Post Title"
date = 2026-01-05
description = "Brief description"

[taxonomies]
tags = ["tag1", "tag2"]
+++

Content here...
```

## Multilingual Support

English (default), French, and Japanese. Translations use Zola's file naming: `post-name.fr.md` / `post-name.ja.md` alongside `post-name.md`.

URLs: English at `/blog/...`, French at `/fr/blog/...`, Japanese at `/ja/blog/...`

## Tabi-Specific Config

- Homepage uses `[extra] section_path = "blog/_index.md"` to show recent posts
- Taxonomies must be declared in `config.toml` before use in posts
- `social_media_card` in frontmatter must be a local file path (not external URLs)
- **Hero/banner images** must be added in TWO places: `social_media_card` in frontmatter AND as the first element in the post body using markdown image syntax (`![alt](/img/banner.webp)`)

## KaTeX (Math Rendering)

Enable per-post with `katex = true` in `[extra]`. Limitations:

- `\mathrm{}` doesn't render inside `<details>` blocks—use plain text instead
- `\begin{cases}` can be flaky—use inline prose for piecewise definitions
- Standard commands (`\frac`, `\sqrt`, `\sum`, `\left`, `\right`) work fine
- Two `$` in the same paragraph (e.g. `$25 ... $49`) get parsed as a math pair. Markdown escapes (`\$`, `\\$`) don't work cleanly with tabi's bundled auto-render. Fix: insert an empty `<span></span>` between the two `$`s to split the text node. Single `$` alone in a paragraph is safe.

## Footnotes

Standard markdown syntax:

```markdown
Text with footnote[^1].

[^1]: Footnote content here.
```

Style footnotes via CSS: `.footnote-definition { font-size: 0.85rem; }`

## Dates

- `post_listing_date = "both"` in `content/blog/_index.md` shows both dates in **listing only**
- For individual posts to show "Updated", add `updated = YYYY-MM-DD` to frontmatter

## Images

- Place in `static/img/`, reference as `/img/filename.ext`
- **Case-sensitive on GitHub Pages** (Linux)—ensure filenames match exactly
- For large images, compress with: `magick input.png -resize 1200x png:- | cwebp -q 90 -o output.webp -- -`

## Custom Stylesheets

Add per-post CSS via frontmatter:

```toml
[extra]
stylesheets = ["css/details.css"]
```

Files go in `static/css/`.

## Responsive Tables

Use the `table` shortcode for responsive tables that scale on mobile:

```markdown
{% table() %}
| col1 | col2 |
|------|------|
| a    | b    |
{% end %}
```

For wide tables with many columns or long content, use `wide=true` to prevent text wrapping (enables horizontal scroll instead):

```markdown
{% table(wide=true) %}
| col1 | col2 | col3 | col4 |
|------|------|------|------|
| long content here | more content | etc | etc |
{% end %}
```

## Mermaid Diagrams

Use the `/mermaid-diagram` skill. See `.claude/skills/mermaid-diagram/SKILL.md`.

## Infrastructure

For details on theme management, search config, Zola installation, syntax highlighting, video/audio hosting, CSP, newsletter, and template overrides, see `.claude/docs/infrastructure.md`.
