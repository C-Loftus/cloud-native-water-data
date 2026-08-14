# Improving Water Data Access with Cloud Native Geospatial Formats

A 15-slide [Slidev](https://sli.dev) lightning talk on serving national-scale hydrography
straight out of object storage — FlatGeobuf and GeoParquet over HTTP range requests, as used
by the USGS-funded [Geoconnex](https://geoconnex.us) project.

## Run it

```bash
pnpm install
pnpm run dev      # http://localhost:3030
```

`pnpm run build` outputs a static SPA to `dist/`; `pnpm run export` renders a PDF
(requires `playwright-chromium`).

## Files

| File | Purpose |
| --- | --- |
| `slides.md` | All 15 slides, with speaker notes in the trailing HTML comments |
| `style.css` | Water theme — blue palette, card/stat helpers, dark slide gradients |
| `global-bottom.vue` | Animated SVG wave motif rendered behind every slide |

## Notes

- Slide 9 embeds <https://colton.place/flatgeobuf-viewer/> in an iframe, so it needs network
  access while presenting — worth having a screen recording as a conference-wifi fallback.
- `global-bottom.vue` renders *underneath* the slide, so `style.css` deliberately keeps
  `.slidev-layout` transparent and paints the background gradient on `.slidev-slide-content`
  and `.print-slide-container` instead. Reversing that will hide the waves.
- Wave animation is disabled under `prefers-reduced-motion` and in print/PDF export.
- `components/Counter.vue`, `pages/`, and `snippets/` are leftovers from the Slidev starter
  and are no longer referenced.
