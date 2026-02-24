# Infrastructure

## Zola Installation

Zola is installed via cargo (not Homebrew) to enable Japanese search:

```bash
cargo install --git https://github.com/getzola/zola --features indexing-ja --locked
```

## CI/CD (GitHub Actions)

Deploy workflow at `.github/workflows/deploy.yml` builds Zola from source (for `indexing-ja`) and deploys to GitHub Pages.

- Zola version is pinned via `tag:` in the cargo-install step — **update this when upgrading Zola**
- Tabi theme version is tracked by the git submodule (no workflow change needed on update)
- First build after cache expiry (~7 day TTL) takes ~7 min; cached builds are fast

## Theme (Git Submodule)

The tabi theme is a git submodule. After cloning on a new machine:

```bash
git submodule update --init
```

To update theme:

```bash
git submodule update --remote themes/tabi
```

Currently tracking `main` (ahead of v4.1.0) for Japanese search fix (tabi#620).

## Search

Elasticlunr client-side search is enabled for all three languages. Each language with `build_search_index = true` needs its own `[languages.XX.search]` section matching the global `[search]` config, or Zola crashes with an index out of bounds error.

## Syntax Highlighting (Zola 0.22+)

```toml
[markdown.highlighting]
style = "class"
theme = "catppuccin-frappe"
```

- `style = "class"` generates `giallo.css` in `static/` with CSS classes
- Tabi's CSS provides the actual colors; the theme just defines token types
- Old format (`[markdown]` with `highlight_code`/`highlight_theme`) no longer works

## Video and Audio

For large media files, use Cloudflare R2 bucket (`vinidlidoo-blog`):

```bash
aws s3 cp file.mp4 s3://vinidlidoo-blog/video/file.mp4 \
  --endpoint-url https://93e9358874da65cc09f1d1f51d83848a.r2.cloudflarestorage.com \
  --profile r2
```

Public URL: `https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/path/file`

Embed with HTML5 tags:

```html
<video autoplay loop muted playsinline>
  <source src="https://pub-....r2.dev/video/file.mp4" type="video/mp4">
</video>

<audio controls>
  <source src="https://pub-....r2.dev/audio/file.mp3" type="audio/mpeg">
</audio>
```

## Content Security Policy

CSP is configured in `config.toml` under `allowed_domains`. When adding external media:

- Add domains to `media-src` for video/audio
- Add domains to `img-src` for images
- Add domains to `form-action` for form submissions (e.g., `buttondown.com`)
- **Inline styles are blocked**—use CSS classes in `static/css/` instead (e.g., `.centered`)

## Newsletter (Buttondown)

Email subscription form via Buttondown, configured globally in `config.toml`:

```toml
[extra]
newsletter_action = "https://buttondown.com/api/emails/embed-subscribe/vinidlidoo"
```

- Form HTML lives in `templates/partials/newsletter.html` (single source of truth)
- Included in homepage (`index.html`), blog section (`section.html`), and posts (`page.html`)
- Labels and button text use i18n strings: `newsletter_label` and `newsletter_button` in `i18n/*.toml`
- Styled via `static/css/newsletter.css` (responsive, theme-aware)
- Giscus comments are disabled but config is preserved in `config.toml`

## Template Overrides

Custom templates in `templates/` override the tabi theme:

- `index.html` — Homepage: renders intro text above posts, newsletter form below
- `section.html` — Blog section: adds newsletter form after post list
- `page.html` — Post pages: adds newsletter form after article content (where comments were)
- `partials/newsletter.html` — Reusable newsletter form partial

Tabi uses `load_data` for i18n (not Zola's built-in `trans()`). Strings are accessed via `language_strings.key_name`, set up in `base.html`.
