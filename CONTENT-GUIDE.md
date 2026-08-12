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
| Avatar photo | `assets/avatar.jpg` | Add/replace the file (not rendered if missing — no error). Keep it web-sized before committing — Hugo only ever displays it downscaled to ~210px, so there's no reason to commit a full multi-megabyte camera original into git history. |
| Homepage content below the bio (currently two link cards: Projects, About) | `layouts/home.html` | Project-root override of the theme's home template — replaces the theme's default "Recent Posts" feed. Card text/links are in this file; card styling is `.home-links`/`.home-link*` in `assets/sass/_custom.scss`. |
| Site-wide fallback meta description | `hugo.toml` | `params.description` |
| Default color mode (light/dark/auto) | `hugo.toml` | `params.defaultColor` |
| Nav menu items, order, links | `hugo.toml` | `[[menu.main]]` blocks — `name`, `url`, `pageRef`, `weight` (lower weight = further left) |
| Social icons | `hugo.toml` | `[[params.socialIcons]]` — `name` must match an icon the theme ships (`github`, `twitter`, `Rss`, etc. — see `themes/hugo-blog-awesome/layouts/_partials/svgs/svgs.html` for the full list) |
| Favicon / touch icons | `assets/icons/<same filename as theme's>` | Add a same-named file at the project root to override the theme's default (e.g. `assets/icons/favicon.ico`) |
| Home page node (mostly vestigial — not visibly rendered) | `content/_index.md` | `title` in front matter |
| About page | `content/pages/about.md` | `title`, `description` front matter + Markdown body |
| Projects list page heading | `content/projects/_index.md` | `title` in front matter |
| itch.io link icon next to a project title | `content/projects/<slug>.md` | `itchio = "https://..."` front matter field. Renders a small itch.io icon linking out, next to the title, only when this field is set — no field, no icon. Icon file is `static/images/icons/itchio.png` (itch.io's own favicon, fetched once and saved locally). Template logic lives in `layouts/projects/single.html`; styling in `.project-title .itch-link`/`.itch-icon` in `assets/sass/_custom.scss`. |
| A project page | `content/projects/<slug>.md` | `title`, `date`, `draft`, `description` front matter + Markdown body. `date` is set but not displayed — the Projects section uses its own portfolio-style templates (`layouts/projects/single.html`, `layouts/projects/list.html`, `layouts/_partials/projectCard.html`) that drop the date and make the title the bold, primary element. Blog posts are unaffected — they still use the theme's generic date-showing templates. `description` doubles as the one-line teaser shown under the title on the Projects list page. |
| Embedded video/media inside a page | Inside that page's `.md` body | Raw HTML (e.g. an `<iframe>`) — `markup.goldmark.renderer.unsafe = true` in `hugo.toml` allows this |
| Video embed sizing/styling | `assets/sass/_custom.scss` | `.video-embed` rule (already wired up; wrap new embeds in a `<div class="video-embed">`) |
| Still-image lead media (a project whose header media is a PNG/JPG instead of a video) | `assets/sass/_custom.scss` `.image-embed` rule + `static/images/projects/<file>` | Same 16:9 box as `.video-embed`, sharing the `media-embed-box` mixin. Put the image file at `static/images/projects/<name>.<ext>` (Hugo copies `static/` to the site root as-is, so it's served at `/images/projects/<name>.<ext>`), then reference it as `<div class="image-embed"><img src="/images/projects/<name>.<ext>" alt="..."></div>` in the project's body. The box holds its 16:9 shape via CSS even before the file exists, so the page structure can be built ahead of the actual image — see `content/projects/the-salad-bowl.md` for a working example (`static/images/projects/the-salad-bowl.jpg`). Image is `object-fit: cover`, so a source that isn't exactly 16:9 gets center-cropped (top/bottom if it's relatively narrower than 16:9, left/right if wider) rather than distorted. |
| Smaller supporting image inside a subheader section (as opposed to the full-width lead media above) | `assets/sass/_custom.scss` `.section-image`/`.section-images` rules + `static/images/projects/<file>` | Natural aspect ratio, capped at 500px wide, centered — not the fixed 16:9 crop-to-fill box `.image-embed` uses. Put the file at `static/images/projects/<name>.<ext>` same as any other project image, then use `<img class="section-image" src="/images/projects/<name>.<ext>" alt="...">` directly (no wrapping `<div>` needed, unlike `.image-embed`). For two images side by side instead of stacked, wrap them: `<div class="section-images"><img class="section-image" ...><img class="section-image" ...></div>`. |
| Any other custom CSS | `assets/sass/_custom.scss` | Add rules — this file overrides the theme's own (empty) `_custom.scss` via the asset overlay |
| Table of contents (on/off, default open) | `hugo.toml` | `params.toc`, `params.tocOpen` |
| "Go to top" button | `hugo.toml` | `params.goToTop` |
| Date format shown on pages | `hugo.toml` | `params.dateFormat` |
| RSS feed depth (summary vs full post) | `hugo.toml` | `params.rssFeedDescription` |
| Google Analytics / Disqus comments | `hugo.toml` | `[services.googleAnalytics] id`, `[services.disqus] shortname` |
| New project page starting template | `themes/hugo-blog-awesome/archetypes/default.md` (read-only reference) | `hugo new projects/your-slug.md` scaffolds from it automatically — includes `description`, `isStarred`, `exclude_from_rss` fields |

## Notes
- **`description` vs the body**: `description` in a page's front matter feeds meta tags, RSS, and search engine snippets — it is not shown on the page itself unless a template explicitly prints it (this theme's `single.html`/`list.html` don't). The visible page content is always the Markdown body.
- **Starting a new project the safe way**: `hugo new projects/your-slug.md` — fills in front matter from the theme's own archetype. Remove `draft = true` to publish.
- **No blog/posts section**: the site only has Home, Projects, and About — the theme's blog-post scaffolding (`content/posts/`, `params.mainSections`, the "Recent Posts" homepage feed) has been removed entirely, not just unlinked from the nav.
- **Multilingual config**: the theme's own example config (`themes/hugo-blog-awesome/exampleSite/hugo.toml`) is written for multiple languages via `[Languages.<code>]` blocks. This site intentionally uses the flat single-language form instead (`[params]` directly) — don't copy patterns from the exampleSite config wholesale, they won't apply as-is.
- Anything under `themes/hugo-blog-awesome/` reflects the upstream theme. If you need to change something there permanently, copy the file to the matching path under the project root instead of editing it in place — that's what `assets/sass/_custom.scss` already does.
