# Content Editing Masterlist

Where to manually change each piece of the site. Theme is
[hugo-blog-awesome](https://github.com/hugo-sid/hugo-blog-awesome),
vendored as a git submodule at `themes/hugo-blog-awesome/` — don't
edit files inside that folder directly (a theme update or re-clone
wipes local edits there). Everything below lives at the project root,
which is the safe place to edit; the theme's asset/layout overlay
system (Hugo resolves `assets/`, `content/`, `layouts/` at the project
root *before* falling back to the theme's copies) is what makes that
possible without forking the theme.

For batching several of these into one request at once, see
`design-process-plan.txt` — it defines a shorthand format and the
verification steps that get run before anything ships.

| Content Type | File | What You Edit |
|---|---|---|
| Site title (browser tab, meta tags) | `hugo.toml` | `title = '...'` |
| Site name (webmanifest, `og:site_name`) | `hugo.toml` | `params.sitename` |
| Homepage name (shown under the avatar) | `hugo.toml` | `params.author.intro` |
| Homepage bio / tagline (shown under the name) | `hugo.toml` | `params.author.description` |
| Author name (schema.org / meta author tags, post bylines) | `hugo.toml` | `params.author.name` |
| Avatar photo | `assets/avatar.jpg` | Add/replace the file (square image; not rendered if missing — no error) |
| Site-wide fallback meta description | `hugo.toml` | `params.description` |
| Default color mode (light/dark/auto) | `hugo.toml` | `params.defaultColor` |
| Nav menu items, order, links | `hugo.toml` | `[[menu.main]]` blocks — `name`, `url`, `pageRef`, `weight` (lower weight = further left) |
| Social icons | `hugo.toml` | `[[params.socialIcons]]` — `name` must match an icon the theme ships (`github`, `twitter`, `Rss`, etc. — see `themes/hugo-blog-awesome/layouts/_partials/svgs/svgs.html` for the full list) |
| Favicon / touch icons | `assets/icons/<same filename as theme's>` | Add a same-named file at the project root to override the theme's default (e.g. `assets/icons/favicon.ico`) |
| Home page node (mostly vestigial — not visibly rendered) | `content/_index.md` | `title` in front matter |
| About page | `content/pages/about.md` | `title`, `description` front matter + Markdown body |
| Posts list page heading | `content/posts/_index.md` | `title` in front matter |
| A blog post | `content/posts/<slug>.md` | `title`, `date`, `draft`, `description`, `isStarred` (adds a star icon on the card), `category` front matter + Markdown body |
| Projects list page heading | `content/projects/_index.md` | `title` in front matter |
| A project page | `content/projects/<slug>.md` | `title`, `date`, `draft`, `description` front matter + Markdown body. `date` is set but not displayed — the Projects section uses its own portfolio-style templates (`layouts/projects/single.html`, `layouts/projects/list.html`, `layouts/_partials/projectCard.html`) that drop the date and make the title the bold, primary element. Blog posts are unaffected — they still use the theme's generic date-showing templates. `description` doubles as the one-line teaser shown under the title on the Projects list page. |
| Embedded video/media inside a page | Inside that page's `.md` body | Raw HTML (e.g. an `<iframe>`) — `markup.goldmark.renderer.unsafe = true` in `hugo.toml` allows this |
| Video embed sizing/styling | `assets/sass/_custom.scss` | `.video-embed` rule (already wired up; wrap new embeds in a `<div class="video-embed">`) |
| Any other custom CSS | `assets/sass/_custom.scss` | Add rules — this file overrides the theme's own (empty) `_custom.scss` via the asset overlay |
| Table of contents (on/off, default open) | `hugo.toml` | `params.toc`, `params.tocOpen` |
| "Go to top" button | `hugo.toml` | `params.goToTop` |
| Date format shown on pages | `hugo.toml` | `params.dateFormat` |
| RSS feed depth (summary vs full post) | `hugo.toml` | `params.rssFeedDescription` |
| Google Analytics / Disqus comments | `hugo.toml` | `[services.googleAnalytics] id`, `[services.disqus] shortname` |
| New page/post starting template | `themes/hugo-blog-awesome/archetypes/default.md` (read-only reference) | `hugo new posts/your-slug.md` or `hugo new projects/your-slug.md` scaffolds from it automatically — includes `description`, `isStarred`, `exclude_from_rss` fields |

## Notes
- **`description` vs the body**: `description` in a page's front matter feeds meta tags, RSS, and search engine snippets — it is not shown on the page itself unless a template explicitly prints it (this theme's `single.html`/`list.html` don't). The visible page content is always the Markdown body.
- **Starting a new post or project the safe way**: `hugo new posts/your-slug.md` or `hugo new projects/your-slug.md` — fills in front matter from the theme's own archetype. Remove `draft = true` to publish.
- **`mainSections`** (`hugo.toml` → `params.mainSections`, currently `['posts']`) controls what shows in the homepage's "Recent Posts" feed. Projects are intentionally left out of that list — they're reached via the Projects nav item instead, so they don't get folded into a feed that's semantically about posts.
- **Multilingual config**: the theme's own example config (`themes/hugo-blog-awesome/exampleSite/hugo.toml`) is written for multiple languages via `[Languages.<code>]` blocks. This site intentionally uses the flat single-language form instead (`[params]` directly) — don't copy patterns from the exampleSite config wholesale, they won't apply as-is.
- Anything under `themes/hugo-blog-awesome/` reflects the upstream theme. If you need to change something there permanently, copy the file to the matching path under the project root instead of editing it in place — that's what `assets/sass/_custom.scss` already does.
