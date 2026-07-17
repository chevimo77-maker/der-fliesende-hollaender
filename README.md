# der fliesende Hollaender

Bilingual (DE/EN) marketing site for Robert-Jan Blonk, an independent tiler
based in Schwetzingen, Germany. Built as a Claude "design component" (`.dc.html`)
page and rendered client-side by a bundled React runtime.

## Stack

- `Der Fliesende Hollaender.dc.html` — page markup + component logic (`x-dc` template,
  `DCLogic` component with theme/language state, scroll-reveal animations, translations).
- `support.js` — bundled dc-runtime (parses `.dc.html`, loads React/ReactDOM/Babel from
  CDN, mounts the component).
- `image-slot.js` — `<image-slot>` custom element for user-fillable images, backed by
  `.image-slots.state.json`.
- `.image-slots.state.json` — persisted image content for each slot (hero, portrait,
  project photos).

## Running locally

This needs to be served over HTTP (not opened as a `file://` URL), since the runtime
does a `fetch` for the image-slot sidecar and injects scripts dynamically.

```bash
python3 -m http.server 8080
# then open http://localhost:8080/Der%20Fliesende%20Hollaender.dc.html
```

## Deploying

Any static host works. Make sure `Der Fliesende Hollaender.dc.html`, `support.js`,
`image-slot.js`, `.image-slots.state.json`, and the `assets/`/`uploads/` folders are
all served from the same directory.
