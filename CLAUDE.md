# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A bilingual (DE/EN) marketing site for Robert-Jan Blonk, an independent tiler in
Schwetzingen, Germany. There is no build step and no package.json — the whole site is a
single `.dc.html` "design component" file rendered client-side by a bundled React
runtime, plus two supporting scripts and a set of static images.

## Running locally

Must be served over HTTP, not opened as a `file://` URL — the runtime does a `fetch`
for the image-slot sidecar and injects `<script>` tags dynamically.

```bash
python3 -m http.server 8080
# open http://localhost:8080/Der%20Fliesende%20Hollaender.dc.html
```

There is no lint/test/build command in this repo.

## Deploying (GitHub Pages)

The live site is hosted on GitHub Pages from the `main` branch root. Two gotchas that
have already bitten this project once each — don't reintroduce them:

- **GitHub Pages does not serve dotfiles.** The image-slot sidecar must be named
  `image-slots.state.json`, not `.image-slots.state.json` (it was renamed once already
  after shipping broken images in production). If you ever add another sidecar/data
  file, keep it dotfile-free for the same reason.
- **Pages serves whatever is at `/index.html`.** `index.html` at the repo root is a
  meta-refresh redirect to `Der Fliesende Hollaender.dc.html` — without it, Pages just
  renders `README.md`. Keep the redirect in sync if the main file is ever renamed.

## Architecture

### The `.dc.html` format

`Der Fliesende Hollaender.dc.html` is not plain HTML. It's a template consumed by a
bundled runtime (`support.js`):

- `<x-dc>...</x-dc>` wraps the page markup. Inside it, `{{ expr }}` is a template
  interpolation, `<sc-if value="{{ cond }}">` is conditional rendering, and
  `<sc-for list="{{ arr }}" as="item">` is a loop — all resolved against the component
  instance below.
- A trailing `<script type="text/x-dc" data-dc-script data-props="...">` defines
  `class Component extends DCLogic { ... }`. This is the single source of truth for:
  - **Translations** — the `t` object has parallel `de` and `en` trees (nav, hero,
    services, projects, about, area, contact, map, footer). Every piece of copy on the
    page lives here, not in the markup.
  - **Theme tokens** — `this.themes.light` / `this.themes.dark` are CSS custom
    properties (`--bg`, `--fg`, `--accent`, etc.) applied to `document.documentElement`
    in `applyTheme()`.
  - **State** — `lang`, `theme`, `menuOpen` — toggled via `toggleLang`/`toggleTheme`/
    `toggleMenu`/`closeMenu`, all bound into the template through `renderVals()`.
  - **Scroll interactions** — `initScroll()` wires up the reveal-on-scroll animations
    (`[data-reveal]`, `[data-img-reveal]`), the animated stat counters (`[data-count]`),
    the scroll-scale feature image (`[data-scale-target]`), and the hero parallax. All
    of it is driven by a single `scroll`/`resize` listener with a 4s safety-timeout
    fallback, torn down and re-armed in `componentDidUpdate` (needed because language
    switches re-render text nodes, which would otherwise leave stale counters).
- `support.js` is a generated bundle ("GENERATED from dc-runtime/src/\*.ts — do not
  edit") that parses the `.dc.html`, lazy-loads React/ReactDOM/Babel from unpkg, and
  mounts the component. Treat it as vendored — the actual dc-runtime source isn't in
  this repo.

### Responsive layout

There's one `@media (max-width: 900px)` block plus a `@media (max-width: 420px)` block
near the top of the `<style>` tag, keyed off `data-m="..."` attributes scattered through
the markup (`hdr`, `nav`, `ctrls`, `hero-grid`, `projgrid`, `projitem`, `area`, etc.)
rather than classes. When adjusting mobile layout, grep for the relevant `data-m` value
to find both the CSS override and the markup it targets.

The project grid (`data-m="projgrid"`) needed `min-width: 0 !important` on
`[data-m="projitem"]` and its image children to fix a grid-blowout bug: `<image-slot>`
defaults to `aspect-ratio: 3/2`, and combined with the section's fixed `height`, that
inflates the item's intrinsic content size past the grid track width under the default
`min-width: auto` on grid items. Keep this in mind if new fixed-aspect-ratio media is
added inside a CSS grid.

The mobile nav is a hamburger button (`data-m="menu-btn"`) toggling
`data-m="menu-overlay"`, a `position: absolute; top: 100%` panel anchored to the fixed
`<header>` (so it sits at the header's bottom edge regardless of the header's rendered
height — don't hardcode a pixel offset here, it broke once already at a wrong guessed
height).

### `<image-slot>` (`image-slot.js`)

A custom element (`customElements.define('image-slot', ...)`) used as
`<image-slot id="..." shape="rect" fit="cover" placeholder="...">` throughout the page
for every photo. Content is persisted to a sidecar JSON file (`STATE_FILE` constant,
currently `image-slots.state.json`) fetched relative to the page and keyed by the
slot's `id` attribute — so slot ids must stay unique and stable across the page. Outside
the omelette/design-canvas runtime the slot is read-only (no drag-and-drop upload).

### Translation-driven content

Because copy, nav labels, project captions, etc. all come from `t.de` / `t.en` in the
component script, adding or editing visible text means editing the `t` object in the
`<script data-dc-script>` block, not the `<x-dc>` markup (which just references
`{{ t.some.path }}`).

## Working Priorities

When making changes, prioritize in this order:

1. Maintainability
2. Readability
3. Small, targeted changes
4. Performance

## Coding Rules

- Never use `any` unless unavoidable.
- Prefer existing utilities and patterns over creating new helpers.
- Keep functions focused on a single responsibility.
- Avoid code duplication.
- Match the existing coding style of the repository.

## Before Coding

- Read only the files necessary for the current task.
- Search for existing implementations before creating new ones.
- Avoid rewriting working code unless explicitly requested.
- Minimize edits to unrelated files.
- Preserve the existing project architecture and bilingual (DE/EN) content structure.

## Git

- Keep changes small and focused.
- Do not modify unrelated files.
- Do not rename files unless requested.
- Preserve existing project structure.

## Responses

- Briefly explain what changed. no more than 300 characters
- Mention any assumptions made.
- If requirements are unclear or information is missing, ask instead of guessing.
- Do not overexplain.

