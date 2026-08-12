# Design Process Plan

Portfolio site for Evan McCaig — `evancmccaig/portfolio`. This file is
written for a **cold-start AI agent** picking up this project with zero
prior conversation history. Read this whole file before touching
anything. It replaces needing the original chat history — if something
here is wrong or missing, fix this file too, not just the site.

## 1. Stack & deployment

- **Static site generator**: Hugo (extended). This machine has
  `0.164.0` installed locally (`hugo version` to confirm).
- **Theme**: [hugo-blog-awesome](https://github.com/hugo-sid/hugo-blog-awesome),
  vendored as a **git submodule** at `themes/hugo-blog-awesome/`.
  **Never edit files inside that folder** — a theme update or fresh
  clone wipes local edits there. Requires Hugo extended `>= 0.160.0`.
- **Hosting**: Cloudflare Pages, auto-builds on every push to `main`.
  Live at `https://portfolio-3i9.pages.dev/` — no custom domain, by
  choice (free subdomain).
- **`HUGO_VERSION` MUST be pinned** in the Cloudflare Pages dashboard
  → Settings → Environment variables → `0.164.0`, for **both**
  Production and Preview. Without this, Cloudflare's default build
  image is old enough to fail on template syntax this theme uses
  (`.Site.Language.Locale`). This bit us once already — see §8.
- No blog/posts section exists. It was fully removed (not just
  unlinked) — don't recreate `content/posts/` or `params.mainSections`
  without being asked.

## 2. Site structure (as of this writing)

Nav: **Home / Projects / About** only.

```
content/
  _index.md              home page node (vestigial — home.html doesn't read its body)
  pages/about.md         About page
  projects/
    _index.md            Projects list page (title only)
    hand-to-hand-wizardry.md
    the-salad-bowl.md
    last-night-at-deer-lake.md   (tracked, but has local uncommitted edits — see §9)

layouts/                 PROJECT-ROOT OVERRIDES (take precedence over the theme's own)
  home.html               replaces theme's "Recent Posts" feed with avatar/bio + Projects/About link cards
  projects/single.html    portfolio-style project page: no date, centered bold title, optional itch.io icon
  projects/list.html      portfolio-style project grid: no date, bold title-first cards
  _partials/projectCard.html

assets/
  avatar.jpg              web-sized (~800px, EXIF-stripped)
  sass/_custom.scss        ALL custom CSS — see §7

static/images/
  icons/itchio.png         itch.io favicon, fetched directly from itch.io (not hand-drawn)
  projects/                every project image/gif lives here, referenced as /images/projects/<file>
```

**Per-project status:**
| Project | Lead media | Subsection images | itch.io link |
|---|---|---|---|
| Hand to Hand Wizardry | `.video-embed` (YouTube iframe) | 4x `.image-embed` (16:9 crop) — **inconsistent with Salad Bowl, this was reverted back from `.section-image` by a direct edit outside an agent session; don't "fix" it without being asked** | `vfs-gdpg.itch.io` |
| The Salad Bowl | `.section-image` (natural aspect) | 3x `.section-image`, real screenshots (`TSB_Gameplay.png`/`2`/`3`) | `squidypal.itch.io` |
| Last Night at Deer Lake | `.image-embed` | — | none set |

All three currently use placeholder/`_Insert..._`-style body text in
most sections — only the top bio paragraphs are real copy. Don't
assume `_italicized placeholder text_` is meant to ship as-is.

## 3. The `[Label]/[Content]` shorthand

The user may hand you a batch of changes in this shorthand instead of
plain English:

```
[Label] Name of the element
[Content] "The value to put there"
```

- **Label** = which kind of element (see legend below). Contextual,
  not a fixed mapping — `[Header]` means something different next to
  "Title page" than next to "Projects Section". The **Name** line
  disambiguates.
- **Name** = which specific instance.
- **Content** = the literal value (quoted text, or a URL/file path).

Multiple instructions can be stacked in one message to batch unrelated
changes together.

| Label | What it controls | Where it lives |
|---|---|---|
| `[Header]` | Site identity, or a new section's heading | `hugo.toml` `params.author.name`/`sitename`, or `content/<section>/_index.md` title |
| `[Subheader]` | Heading for one item inside a section | `content/<section>/<slug>.md` title |
| `[Sub-bio]` | Secondary line under a heading | Homepage tagline → `hugo.toml` `params.author.description`; per-item → body text |
| `[Intro]` | Line above the tagline | `hugo.toml` `params.author.intro` |
| `[Link]` | Embedded link/media | Raw HTML in the `.md` body (`markup.goldmark.renderer.unsafe = true` allows this) |
| `[About]` | About page body | `content/pages/about.md` |
| `[Social]` | A social/contact link | `hugo.toml` `[[params.socialIcons]]` |
| `[Avatar]` | Profile photo | `assets/avatar.jpg` |
| `[Menu]` | Nav entry | `hugo.toml` `[[menu.main]]` |

If a Name doesn't map to anything that exists yet (e.g. a brand new
section), that's a signal to **build the structure first** (§4 step 2),
not an error to flag back to the user.

For the full "what file do I edit for X" reference beyond this
shorthand, see **`CONTENT-GUIDE.md`** — it's the authoritative,
exhaustive content-editing table and is kept in sync with this file.

## 4. Workflow — execute every change this way

1. **Parse.** Resolve every instruction to an exact file + field.
   Only stop to ask the user if something is genuinely ambiguous — a
   brand-new Name is a build step, not ambiguity.
2. **Build structure if needed**, before writing content into it:
   - New content section → `content/<section>/_index.md` + detail
     pages. Check whether the theme's generic templates already
     render it acceptably before writing custom `layouts/` overrides.
   - New nav entry → `[[menu.main]]` in `hugo.toml`, `weight` controls
     position.
   - New visual/embed pattern → add a class to `assets/sass/_custom.scss`
     (never edit the theme's copy) rather than one-off inline styles.
3. **Apply content.**
4. **Build locally**: `hugo --gc --minify` from repo root. Zero errors
   or warnings required. A single malformed content file (bad TOML, a
   misplaced `+++`, an unparsable `date`) fails the **entire site**,
   not just that page — always read the actual error before assuming
   a CSS/rendering bug (see §8).
5. **Verify by grepping `public/`** — don't trust "it compiled". Check
   the actual built HTML for the changed string, class, or URL. Watch
   for minified HTML dropping attribute quotes (`class=foo` not
   `class="foo"`) — match your grep pattern accordingly.
6. **Check visuals — don't just reason about CSS.** Grepping proves
   markup exists, not that it's visible or laid out correctly.
   ```
   hugo server --port 1313 &
   # poll http://localhost:1313/ until it responds — don't sleep-and-hope
   npx --yes playwright screenshot --wait-for-timeout=1500 \
     --full-page "http://localhost:1313/<path>/" out.png
   ```
   `chromium-cli` isn't installed on this machine — Playwright's
   one-shot `screenshot` command is the fallback (`npx playwright
   install chromium` once if the browser binary is missing). For
   testing a *new CSS pattern* not yet wired into real content, build
   an **isolated test page** (compiled CSS + a real asset file in a
   plain HTML file, outside the Hugo content tree) rather than editing
   real project pages just to check styling — keeps verification
   separate from content changes. **Actually look at the screenshot.**
   Stop the dev server afterward (kill whatever's listening on the
   port — `netstat -ano | grep ':1313' | grep LISTENING | awk
   '{print $5}' | sort -u | xargs -r -I{} taskkill //F //PID {}` on
   this Windows/Git-Bash environment; `Stop-Process` via PowerShell
   also works) so it doesn't linger.
7. **Run the domain verification checklist** (§5).
8. **Commit** (one commit per logical batch, descriptive message) and
   **push** — ask for confirmation before pushing unless told not to
   for that session. **Stage deliberately** — `git status` first, and
   only `git add` files actually in scope for the request. This repo
   regularly has unrelated in-progress edits sitting in the working
   tree (the user edits files directly outside agent sessions too);
   don't sweep those into an unrelated commit, but don't silently
   discard them either — leave them uncommitted and mention them.
9. **Re-check live** after push — fetch the live URL and confirm the
   deploy actually picked up the change. A successful push is not a
   successful deploy (Cloudflare can still fail the build).

## 5. Domain verification checklist (run after every push)

- [ ] `baseURL` in `hugo.toml` matches the live URL
      (`https://portfolio-3i9.pages.dev/`)
- [ ] Cloudflare Pages "Deployments" tab shows the latest commit hash
      with a **successful** build
- [ ] `HUGO_VERSION` is pinned for both Production and Preview (§1)
- [ ] Live page HTML actually contains the new content — fetch it and
      grep, don't eyeball a screenshot (CDN/browser caching can show a
      stale page even after a good deploy)
- [ ] If live looks stale but the deployment succeeded: hard refresh /
      incognito before concluding anything is broken — see §8, this
      has produced multiple false-alarm "bug reports" already
- [ ] If a Retry/"force redeploy" of a failed build shows the
      identical error after you fixed the cause, don't trust it —
      Retry can re-run against the config snapshotted at original
      trigger time. Push a new commit (an empty one is fine) to force
      a genuinely fresh build.

## 6. Custom asset workflow (compression)

- **Oversized images/GIFs are a recurring problem** — content gets
  dropped in from cameras/screen recorders at full resolution.
  **Always check dimensions/file size before committing new binary
  assets.** Rough thresholds that have warranted action so far: any
  single image over ~2MB, any GIF over ~10MB.
- **JPEG/PNG photos**: Python + Pillow.
  ```python
  from PIL import Image, ImageOps
  img = Image.open(path)
  img = ImageOps.exif_transpose(img)   # bake in rotation before stripping EXIF
  img = img.convert('RGB')
  img.thumbnail((1600, 1600), Image.LANCZOS)  # 800 for an avatar-sized image
  img.save(path, 'JPEG', quality=85, optimize=True)
  ```
- **GIFs**: `gifsicle` via `npx` (no local install needed, pulls a
  prebuilt Windows binary):
  ```
  npx --yes gifsicle -O3 --lossy=80 --colors=128 --resize-width 640 \
    input.gif -o output.gif
  ```
  This has produced ~90% size reductions with no visible quality loss
  on real gameplay-capture GIFs. Verify by extracting a mid-animation
  frame (`img.seek(N)`) and viewing it before committing.
- Cloudflare Pages has historically enforced a **~25MB per-file**
  limit on static assets — anything near/over that will likely fail
  to deploy outright, not just load slowly. Flag this to the user
  before committing something that large rather than finding out via
  a failed deploy.

## 7. CSS component reference (`assets/sass/_custom.scss`)

All of it is layered in via Hugo's asset overlay (project-root
`assets/` resolves before the theme's own, which has this exact same
filename but empty — that's the hook, not a coincidence).

| Class | Purpose | Usage |
|---|---|---|
| `.video-embed` | Full-width 16:9 box for a YouTube iframe (project lead media) | `<div class="video-embed"><iframe ...></iframe></div>` |
| `.image-embed` | Same 16:9 box, for a still image instead of video. `object-fit: cover` crops to fill. | `<div class="image-embed"><img src="..." alt="..."></div>` |
| `.section-image` | Smaller inline image inside a body subsection. Natural aspect ratio, max-width 500px, **left-aligned**, no wrapping div. | `<img class="section-image" src="/images/projects/<f>" alt="...">` |
| `.section-images` | Wrapper for 2+ `.section-image` side by side instead of stacked (each capped at `50% - gap` so they actually fit) | `<div class="section-images">...</div>` |
| `.section-gif` | Like `.section-image` but larger (640px) and bordered, for animated GIFs. Plain `<img>` autoplays/loops a GIF natively — no JS. | `<img class="section-gif" src="/images/projects/<f>.gif" alt="...">` |
| `.section-heading` | A body heading styled to match the page title's weight (2em/800). Set explicitly rather than reusing `.header-title`, which is scoped to `.header .header-title` and won't apply in body content. Use Hugo's heading-attribute syntax so it stays in the Table of Contents (raw HTML headings don't). | `## Heading Text {.section-heading}` |
| `.project-title` | The project page's own `<h1>` — bold weight; also hosts `.itch-link`/`.itch-icon` styling | applied by `layouts/projects/single.html` |
| `.itch-link` / `.itch-icon` | Small itch.io icon inline in the (centered) title, right after the title text, opt-in per project | set `itchio = "https://..."` in that project's front matter — see `layouts/projects/single.html` |
| `.project-list` / `.project-item` / `.project-item-title` / `.project-item-description` | Portfolio-style Projects grid cards (bold title, no date) | rendered by `layouts/_partials/projectCard.html` |
| `.home-links` / `.home-link` / `.home-link-title` / `.home-link-desc` | Homepage Projects/About nav cards | rendered by `layouts/home.html` |

**Dark mode**: the theme's own dark-mode mixin (in the *theme's*
`_dark.scss`) only recolors theme-native classes. Every custom
project-root class above that has its own colors (link colors,
borders) needs its **own** dark-mode override, added inside the
`@mixin custom-dark-mode { ... }` block at the bottom of
`_custom.scss`, triggered the same way the theme triggers its own:
```scss
@media (prefers-color-scheme: dark) { html:not(.light) { @include custom-dark-mode; } }
html.dark { @include custom-dark-mode; }
```
Forgetting this is an easy, easy-to-miss mistake — nothing errors,
it just looks wrong (or invisible) in dark mode only. Always screenshot
both light and dark when adding a new custom class with its own colors.

## 8. Gotchas — read before you repeat these

- **One malformed content file fails the whole build.** An unparsable
  `date`, or a `+++` closing delimiter misplaced after HTML instead of
  right after the front matter fields, are both fatal TOML parse
  errors that take down every page, not just the offending one. If a
  user reports "X isn't showing," run a clean build and read the
  actual error before assuming a CSS/template bug.
- **"It's not showing" is very often stale cache, not a real bug.**
  This has happened repeatedly: content is correct in the source file
  AND correct in the live HTML (verified via direct `curl`, not just
  eyeballing a browser) AND renders correctly in a fresh headless
  screenshot — yet gets reported as missing. Always verify via curl +
  screenshot before changing anything; don't assume the report is
  accurate just because it's confidently stated.
- **`.header-title`'s sizing is scoped to `.header .header-title`** in
  the theme's SCSS — reusing that class alone on an element outside
  the `<header>` wrapper won't pick up the font-size. Build a
  dedicated class instead (`.section-heading` does this correctly).
- **`justify-content` is a no-op on anything that isn't a flex/grid
  container** — a `display: block` `<img>` with `justify-content: center`
  does nothing; the real centering/alignment mechanism for a block
  element is its `margin` (`auto` = centered, `0` = left-aligned).
  This exact mistake shipped once (a redundant `justify-content: left`
  sitting next to the actual `margin: auto` that was doing the
  centering) — check for this pattern before trusting a stray
  `justify-content` declaration on a non-flex element.
- **Side-by-side flex items need width capped relative to the
  container, not to a fixed pixel value** — two elements each with
  `max-width: 500px` in a ~900px container don't fit side by side and
  silently wrap to stacked, which looks *identical* to just not using
  the side-by-side wrapper at all (no visual error, just quietly
  wrong). Use `max-width: calc(50% - <half the gap>)` scoped to the
  side-by-side context.
- **`object-fit: cover` crop math**: when a source image's aspect
  ratio doesn't match its box, `cover` scales to whichever dimension
  is more constraining and crops the other, centered by default. Do
  the ratio comparison before shipping if the crop might cut off
  something that matters (it crops top/bottom if the source is
  relatively *narrower* than the box, left/right if *wider*).
- **Cloudflare's edge cache is separate per URL** — a brand-new path
  that's never been requested before will serve the fresh deployment
  immediately, while a previously-cached URL (like `/`) can lag behind
  even on a successful deploy. Don't conclude a deploy failed just
  because one specific URL looks stale.
- **`git status` regularly shows edits this session didn't make** —
  the user edits files directly outside agent sessions in parallel.
  Read the diff before assuming it's noise; these are usually
  legitimate in-progress content and should be preserved (folded into
  a commit if in scope, left alone and mentioned if not), never
  silently reverted.

## 9. Open / pending items

- `content/projects/last-night-at-deer-lake.md` is tracked but has
  **local uncommitted edits** right now — real board-game-photo
  content (`LNADL_OverviewShot.jpg`), no `itchio` link set, no
  subsection headings yet. Ask before committing/publishing further
  changes to it — it hasn't explicitly been signed off as ready.
- `static/images/projects/TheSaladBowlMainMenu.png` is sitting
  **untracked**, unaddressed — purpose/destination unconfirmed.
- Hand-to-Hand Wizardry's 4 subsection images are on `.image-embed`
  (16:9 crop) while The Salad Bowl's equivalent images are on
  `.section-image` (natural aspect) — a real, known inconsistency
  from a direct revert outside an agent session (§2 table). Don't
  "fix" this without being asked which way to standardize.
- Most subsection body text across all three projects is still
  `_Insert blurb..._`-style placeholder — not ready to be treated as
  final copy.
- No custom domain (by choice, on the free `*.pages.dev` subdomain).
- Roadmap items requiring work outside this repo (recording footage,
  editing video, writing docs) — not implementable by an agent alone:
  - Level Design Documentation video (combat calculator, modular kit
    building, timelapse layouts, narrated design reasoning, edited in
    Premiere Pro)
  - Git page for the Hand-to-Hand Wizardry gameplay event bus (readable
    script, demo folder, screenshots, linked from the portfolio)
