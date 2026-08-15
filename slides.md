---
theme: default
title: Improving Water Data Access with Cloud Native Geospatial Formats
info: |
  ## Improving Water Data Access with Cloud Native Geospatial Formats

  A lightning talk on how the Geoconnex project (funded by the U.S. Geological Survey)
  uses FlatGeobuf and GeoParquet to serve national-scale hydrography straight out of
  object storage — no database, no API server, no derived subsets to keep in sync.
author: Colton Loftus
colorSchema: dark
highlighter: shiki
lineNumbers: false
transition: slide-left
mdc: true
drawings:
  persist: false
duration: 10min
class: text-center
---

<div class="eyebrow mb-4">Lightning talk</div>

# Improving Water Data Access with Cloud Native Geospatial Formats

<div class="abs-bl m-8 text-left text-sm muted">
  Colton Loftus · Internet of Water / Geoconnex<br>
  Funded by the U.S. Geological Survey
</div>

<div class="abs-br m-8 text-sm muted">
  <a href="https://geoconnex.us" target="_blank">geoconnex.us</a>
</div>

<!--
Hi — I'm Colton. Ten minutes on a boring-sounding claim: the hardest part of publishing
geospatial data on the web usually has nothing to do with geospatial.
-->

---
layout: center
class: text-center
---

# Who actually has this data?

<div v-click class="text-lg muted max-w-2xl mx-auto">
  There's no single U.S. water data agency. The federal government, states, utility companies, and universities each have valuable but fragmented datasets.
</div>

<figure class="mt-6">
<svg viewBox="0 0 640 400" role="img" aria-label="Federal agencies, states, community organizations, academia, and utilities form a mesh, each needing to coordinate directly with every other, with no central water data agency in the middle" style="width:100%;max-width:480px;height:auto;margin:0 auto;display:block;color:var(--wtr-text)">
  <g stroke="var(--wtr-muted)" stroke-opacity="0.3" stroke-width="1.3">
    <line x1="320" y1="60" x2="463" y2="164"/>
    <line x1="320" y1="60" x2="408" y2="331"/>
    <line x1="320" y1="60" x2="232" y2="331"/>
    <line x1="320" y1="60" x2="177" y2="164"/>
    <line x1="463" y1="164" x2="408" y2="331"/>
    <line x1="463" y1="164" x2="232" y2="331"/>
    <line x1="463" y1="164" x2="177" y2="164"/>
    <line x1="408" y1="331" x2="232" y2="331"/>
    <line x1="408" y1="331" x2="177" y2="164"/>
    <line x1="232" y1="331" x2="177" y2="164"/>
  </g>
  <circle cx="320" cy="210" r="54" fill-opacity="0.92" stroke-width="2" stroke-dasharray="5 5" style="fill: var(--wtr-deep); stroke: var(--wtr-sand)"/>
  <text x="320" y="223" text-anchor="middle" style="font-size:38px;fill: var(--wtr-sand)">?</text>
  <g style="font-size:12.5px;fill:var(--wtr-text)" text-anchor="middle">
    <rect x="250" y="38" width="140" height="44" rx="8" fill="rgba(11,79,119,0.85)" stroke="rgba(165,227,255,0.4)"/>
    <text x="320" y="65">Federal agencies</text>
    <rect x="393" y="142" width="140" height="44" rx="8" fill="rgba(11,79,119,0.85)" stroke="rgba(165,227,255,0.4)"/>
    <text x="463" y="169">States</text>
    <rect x="338" y="309" width="140" height="44" rx="8" fill="rgba(11,79,119,0.85)" stroke="rgba(165,227,255,0.4)"/>
    <text x="408" y="336">Community orgs</text>
    <rect x="162" y="309" width="140" height="44" rx="8" fill="rgba(11,79,119,0.85)" stroke="rgba(165,227,255,0.4)"/>
    <text x="232" y="336">Academia</text>
    <rect x="107" y="142" width="140" height="44" rx="8" fill="rgba(11,79,119,0.85)" stroke="rgba(165,227,255,0.4)"/>
    <text x="177" y="169">Utilities</text>
  </g>
</svg>
</figure>

---
layout: center
class: text-center
---

# The problem is more fundammental than scale

<div v-click class="mt-8 text-4xl font-bold" style="color: var(--wtr-accent)">
  It's infrastructure.
</div>

<div v-click class="mt-10 max-w-3xl mx-auto muted text-lg">
  Most organizations already have the data. What they don't have is somebody
  to keep a database, an API, and a tile server alive for the next ten years.
</div>

---

# The usual way to put data on the web

<figure class="mt-6">
<svg viewBox="0 0 900 180" role="img" aria-label="A pipeline from source data through an ETL job, PostGIS, an API server, a tile server, and a CDN cache. Five of the six stages must run continuously for years; only the source data is captured once." style="width:100%;height:auto;color:var(--wtr-text)">
  <defs>
    <marker id="pipe-arrow-head" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <g stroke="currentColor" stroke-width="1.5" opacity="0.85" marker-end="url(#pipe-arrow-head)">
    <line x1="142" y1="60" x2="182" y2="60"/>
    <line x1="286" y1="60" x2="326" y2="60"/>
    <line x1="430" y1="60" x2="470" y2="60"/>
    <line x1="574" y1="60" x2="614" y2="60"/>
    <line x1="718" y1="60" x2="758" y2="60"/>
  </g>
  <g fill="rgba(11,79,119,0.7)" stroke="rgba(165,227,255,0.4)" stroke-width="1.2">
    <rect x="38" y="25" width="104" height="70" rx="10"/>
    <rect x="182" y="25" width="104" height="70" rx="10"/>
    <rect x="326" y="25" width="104" height="70" rx="10"/>
    <rect x="470" y="25" width="104" height="70" rx="10"/>
    <rect x="614" y="25" width="104" height="70" rx="10"/>
    <rect x="758" y="25" width="104" height="70" rx="10"/>
  </g>
  <g transform="translate(90,45)" fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="-8" y="-9" width="16" height="18" rx="1.5"/>
    <path d="M4,-9 v5 h5"/>
    <line x1="-4" y1="-2" x2="4" y2="-2"/>
    <line x1="-4" y1="2" x2="4" y2="2"/>
    <line x1="-4" y1="6" x2="2" y2="6"/>
  </g>
  <g transform="translate(234,45)" fill="none" stroke="currentColor" stroke-width="1.4">
    <circle r="6"/>
    <circle r="1.8" fill="currentColor" stroke="none"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none" transform="rotate(60)"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none" transform="rotate(120)"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none" transform="rotate(180)"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none" transform="rotate(240)"/>
    <rect x="-1.1" y="-9.5" width="2.2" height="4" fill="currentColor" stroke="none" transform="rotate(300)"/>
  </g>
  <g transform="translate(378,45)" fill="none" stroke="currentColor" stroke-width="1.4">
    <ellipse cx="0" cy="-7" rx="8" ry="3"/>
    <path d="M-8,-7 v10 a8,3 0 0 0 16,0 v-10"/>
    <path d="M-8,-2 a8,3 0 0 0 16,0" opacity="0.6"/>
  </g>
  <g transform="translate(522,45)" fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="-8" y="-9" width="16" height="18" rx="1.5"/>
    <line x1="-8" y1="-3" x2="8" y2="-3"/>
    <line x1="-8" y1="3" x2="8" y2="3"/>
    <circle cx="5" cy="-6" r="1" fill="currentColor" stroke="none"/>
    <circle cx="5" cy="0" r="1" fill="currentColor" stroke="none"/>
  </g>
  <g transform="translate(666,45)" fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="-8" y="-8" width="7" height="7"/>
    <rect x="1" y="-8" width="7" height="7"/>
    <rect x="-8" y="1" width="7" height="7"/>
    <rect x="1" y="1" width="7" height="7"/>
  </g>
  <g transform="translate(810,45)" fill="none" stroke="currentColor" stroke-width="1.3">
    <circle cx="-4" cy="-2" r="4"/>
    <circle cx="2" cy="-4" r="5"/>
    <circle cx="6" cy="-1" r="3.5"/>
    <rect x="-8" y="-2" width="16" height="6" rx="3"/>
  </g>
  <g style="font-size:12px;fill:var(--wtr-text)" text-anchor="middle">
    <text x="90" y="74">Source</text>
    <text x="90" y="87">data</text>
    <text x="234" y="74">ETL</text>
    <text x="234" y="87">job</text>
    <text x="378" y="80">PostGIS</text>
    <text x="522" y="74">API</text>
    <text x="522" y="87">server</text>
    <text x="666" y="74">Tile</text>
    <text x="666" y="87">server</text>
    <text x="810" y="74">CDN /</text>
    <text x="810" y="87">cache</text>
  </g>
  <line x1="234" y1="95" x2="234" y2="115" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="378" y1="95" x2="378" y2="115" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="522" y1="95" x2="522" y2="115" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="666" y1="95" x2="666" y2="115" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="810" y1="95" x2="810" y2="115" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <rect x="182" y="115" width="680" height="6" rx="3" fill-opacity="0.55" style="fill: var(--wtr-accent)"/>
  <polygon points="862,112 872,118 862,124" opacity="0.55" style="fill: var(--wtr-accent)"/>
  <text x="182" y="136" text-anchor="start" style="font-size:12px;fill: var(--wtr-accent)">today</text>
  <text x="862" y="136" text-anchor="end" style="font-size:12px;fill: var(--wtr-accent)">10 years later</text>
  <text x="522" y="158" text-anchor="middle" style="font-size:12.5px;fill: var(--wtr-muted)">must stay up — patched, paid for, paged</text>
</svg>
<figcaption class="text-sm muted mt-2 text-center">Five of six stages must run non-stop for years — only the source data is a one-time thing.</figcaption>
</figure>

<div v-click class="mt-10 grid grid-cols-3 gap-4 text-sm">
  <div class="card">
    <div class="eyebrow mb-1">Runs forever</div>
    Patching, upgrades, certs, backups, connection pools
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Costs forever</div>
    Idle compute billed at 3am for a dataset nobody queried
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Fails loudly</div>
    One OOM and every downstream map is blank
  </div>
</div>

<div v-click class="mt-8 text-lg">
  Five moving parts to answer one question: <em>what's near this point?</em>
</div>


---

# And then the copies drift

<div class="grid grid-cols-2 gap-6 mt-6">

<div>

Because the full dataset is too big for the browser, somebody makes a subset:

- a simplified GeoJSON for the web map
- a state-by-state export for partners
- a "just the mainstems" file for the demo
- vector tiles baked last quarter

</div>

<div v-click class="card card-warn">
<div class="eyebrow mb-2" style="color: var(--wtr-sand)">Six months later</div>

The source is updated. **The subsets are not.**

The map, the API, and the download page now disagree about how many stream reaches exist — and no one can say which is right.

</div>

</div>

<div v-click class="mt-8 text-center text-xl">
  Every derived copy is a <span v-mark.underline.cyan="3">synchronization bug</span> waiting to happen.
</div>
---
layout: center
class: text-center
---

# What if the file *is* the API?

<div class="mt-10 flex items-center justify-center gap-4">
  <div class="pipe-box" style="min-width: 9rem">Browser</div>
  <div class="pipe-arrow text-sm">Range: bytes=<br>4096-65535 →</div>
  <div class="pipe-box card-accent" style="min-width: 9rem">
    One file<br><span class="muted text-xs">S3 / R2 / Azure Blob</span>
  </div>
</div>

<div v-click class="mt-12 grid grid-cols-3 gap-4 text-sm max-w-4xl mx-auto">
  <div class="card">Object storage already speaks <strong>HTTP range requests</strong></div>
  <div class="card">The format puts a <strong>spatial index in the file</strong></div>
  <div class="card">So the client fetches <strong>only the bytes it needs</strong></div>
</div>

<div v-click class="mt-10 text-lg muted">
  No database. No API process. No autoscaling group. Just a bucket and a CDN.
</div>

---

# FlatGeobuf

<div class="grid grid-cols-2 gap-8 mt-4">

<div>

A single binary file: FlatBuffers-encoded features with a **packed Hilbert R-tree** written
into the head of the file.

<div class="mt-6 space-y-2">
  <div class="card flex items-center gap-3">
    <span class="eyebrow" style="min-width: 4.5rem">Request 1</span>
    <span class="text-sm">Header — schema, CRS, feature count</span>
  </div>
  <div class="card flex items-center gap-3">
    <span class="eyebrow" style="min-width: 4.5rem">Request 2</span>
    <span class="text-sm">Walk the R-tree for your bounding box</span>
  </div>
  <div class="card card-accent flex items-center gap-3">
    <span class="eyebrow" style="min-width: 4.5rem">Request 3</span>
    <span class="text-sm">Fetch just those feature byte ranges</span>
  </div>
</div>

</div>

<div>

<div class="eyebrow mb-2">Why it fits water data</div>

- **Streamable** — features arrive and render progressively
- **Lossless** — full geometry, full attributes, no tile generalization
- **Boring to host** — static file + CORS + range support
- **Universally readable** — GDAL/OGR, QGIS, and a small JS library

<div v-click class="card card-accent mt-6 text-sm">
The spatial index is the part you would otherwise have paid a database to hold.
</div>

</div>

</div>

---

# The same file, two very different consumers

<div class="grid grid-cols-2 gap-6 mt-6">

```ts {all|3-4|6-8}
// Browser: draw what's in the viewport
import { deserialize } from 'flatgeobuf/lib/mjs/geojson.js'

const url = 'https://reference.geoconnex.us/catchments.fgb'
const bbox = map.getBounds().toArray().flat()

for await (const feature of deserialize(url, bbox)) {
  source.addFeature(feature) // renders as it streams
}
```

```python {all|3-4}
# Backend microservice: same URL, same bytes
import geopandas as gpd

gdf = gpd.read_file(
    "https://reference.geoconnex.us/catchments.fgb",
    bbox=(-105.1, 39.6, -104.7, 39.9),
)
```

</div>

<div v-click class="mt-6 card card-accent text-center">
  One artifact is the map layer <strong>and</strong> the service dependency.
  There is no "web version" to keep in sync, because there is no second copy.
</div>


---

# Geoconnex, in practice: one file, not one copy per service

<div class="grid grid-cols-3 gap-4 mt-8">
  <div class="card">
    <div class="stat-num">2.7M</div>
    <div class="text-sm mt-1">NHDPlus V2 catchments — the whole country in one file</div>
  </div>
  <div class="card">
    <div class="stat-num">All</div>
    <div class="text-sm mt-1">Mainstem rivers in the United States, same treatment</div>
  </div>
  <div class="card card-accent">
    <div class="stat-num">0</div>
    <div class="text-sm mt-1">Database connections to open, pool, or exhaust</div>
  </div>
</div>

<div class="grid grid-cols-2 gap-6 mt-10">

<div v-click>
<div class="eyebrow mb-2">Internal microservices read the same file</div>

- Every Geoconnex reference-feature microservice fetches from the same `.fgb`, not a private copy or its own DB
- No connection pools, no query load to capacity-plan around — just HTTP range requests
- The public map and `ogr2ogr` on someone's laptop hit the exact same bytes

</div>

<div v-click>
<div class="eyebrow mb-2">Reducing duplication</div>

- One canonical file instead of a database per service, each with its own drift
- Scaling is copying: another CDN edge, another replica — no schema, no migrations
- Update the file once and every consumer, internal or external, sees it

</div>

</div>

---
layout: full
class: p-0 overflow-hidden
---

<div class="w-full h-full flex flex-col">
  <div class="px-6 pt-4 pb-2 flex items-baseline gap-4">
    <h2 class="!m-0 !text-2xl">Live: range requests against a 2.7M-feature file</h2>
    <a href="https://colton.place/flatgeobuf-viewer/" target="_blank" class="text-sm muted !border-0">
      colton.place/flatgeobuf-viewer
    </a>
  </div>
  <iframe
    src="https://colton.place/flatgeobuf-viewer/"
    class="flex-1 w-full border-0"
    title="FlatGeobuf viewer"
    loading="lazy"
    allow="fullscreen"
  />
</div>
---

# GeoParquet: when the question is analytical

<div class="grid grid-cols-2 gap-8 mt-4">

<div>

Columnar storage, geometry as WKB, plus a `geo` metadata block. The wins come from the
layout rather than from a spatial index:

- **Row groups + column chunks** with min/max statistics
- **Predicate pushdown** — skip whole row groups that can't match
- **Column pruning** — read 3 columns out of 40
- **`bbox` covering columns** (GeoParquet 1.1) make spatial filters prunable

</div>

<div>

<div class="eyebrow mb-2">In the browser</div>

DuckDB-WASM (or hyparquet) issues its own HTTP range requests against the same static file.

<div class="card card-accent mt-4 text-sm">
Aggregate across the entire United States — <em>count, join, group by</em> — with no
query service in front of it.
</div>

<div v-click class="card mt-4 text-sm">
Compression matters here: a national table that is gigabytes as GeoJSON is often a few
hundred megabytes as Parquet, and you only ever pull a slice of it.
</div>

</div>

</div>


---

# Querying the country from a browser tab

```sql {all|5-6|8-11|13}
-- DuckDB-WASM, running entirely client-side
INSTALL spatial; LOAD spatial;

SELECT
  uri,
  name_at_outlet
FROM 'https://reference.geoconnex.us/mainstems.parquet'
WHERE bbox.xmin < -104.7
  AND bbox.xmax > -105.1
  AND bbox.ymin <  39.9
  AND bbox.ymax >  39.6
ORDER BY length_km DESC
LIMIT 25;
```

<div class="grid grid-cols-3 gap-4 mt-6 text-sm">
  <div v-click class="card">The <code>bbox</code> struct is a plain column — its statistics let DuckDB skip row groups</div>
  <div v-click class="card">Only the matching row groups and the three named columns are fetched</div>
  <div v-click class="card card-accent">Result: a national query answered with a few megabytes of range requests</div>
</div>


---

# Picking between them

| | **FlatGeobuf** | **GeoParquet** |
|---|---|---|
| Best at | Fetch features in a bbox, stream + render | Filter, aggregate, join across many rows |
| Index | Packed Hilbert R-tree in the file | Row-group statistics, `bbox` covering columns |
| Access pattern | Spatial window → geometries | Analytical query → columns |
| Browser story | Tiny JS lib, progressive rendering | DuckDB-WASM / hyparquet |
| Use it for | Map layers, reference geometry | Attribute-heavy analysis, summaries |

<div class="mt-8 grid grid-cols-2 gap-6">
  <div class="card card-accent">
    <div class="eyebrow mb-1">Our rule of thumb</div>
    If a map is going to draw it, FlatGeobuf. If SQL is going to chew on it, GeoParquet.
  </div>
  <div v-click class="card">
    <div class="eyebrow mb-1">Both, often</div>
    Same source of truth, two encodings, one build step. Still zero servers.
  </div>
</div>


---

# The fine print

<div class="grid grid-cols-2 gap-6 mt-6">

<div class="space-y-3">
  <div class="card">
    <div class="eyebrow mb-1">Your bucket must cooperate</div>
    <span class="text-sm">Range requests <em>and</em> CORS headers. Put a CDN in front or every pan pays full latency.</span>
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Writes are whole-file</div>
    <span class="text-sm">Great for reference data that changes on a release cadence. Bad for a live edit queue.</span>
  </div>
</div>

<div class="space-y-3">
  <div class="card">
    <div class="eyebrow mb-1">Cache invalidation is now your job</div>
    <span class="text-sm">Version the object name, or you will serve yesterday's hydrography from the edge.</span>
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Not a replacement for everything</div>
    <span class="text-sm">Complex server-side joins, auth per feature, or heavy write paths still want a database.</span>
  </div>
</div>

</div>

<div v-click class="mt-8 text-center text-lg">
  The trade is real — you give up query flexibility and buy back <strong>an operations budget</strong>.
</div>


---

# What actually changed for us

<div class="grid grid-cols-2 gap-6 mt-8">

<div class="card">
<div class="eyebrow mb-2">For the team</div>

- One artifact to publish instead of a pipeline to babysit
- No 3am pages for a service that only serves geometry
- Storage-shaped costs instead of compute-shaped costs

</div>

<div class="card card-accent">
<div class="eyebrow mb-2">For everyone else</div>

- The map draws the *real* geometry, not a stale simplification
- Anyone can point QGIS or GDAL at the same URL
- Frontend and backend can never disagree about the data

</div>

</div>

<div v-click class="mt-10 text-center text-xl">
  Less data to maintain, and a better experience at the other end of the wire.
</div>


---
layout: center
class: text-center
---

# Take one thing away

<div class="mt-8 text-2xl max-w-3xl mx-auto">
  Before you provision a database to publish geospatial data,
  <span v-mark.circle.cyan="1" class="px-1">check whether a file in a bucket will do</span>.
</div>

<div class="mt-14 grid grid-cols-3 gap-4 text-sm max-w-4xl mx-auto">
  <div class="card">
    <div class="eyebrow mb-1">Geoconnex</div>
    <a href="https://geoconnex.us" target="_blank">geoconnex.us</a>
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Demo viewer</div>
    <a href="https://colton.place/flatgeobuf-viewer/" target="_blank">colton.place/flatgeobuf-viewer</a>
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Formats</div>
    <a href="https://flatgeobuf.org" target="_blank">flatgeobuf.org</a> ·
    <a href="https://geoparquet.org" target="_blank">geoparquet.org</a>
  </div>
</div>

<div class="mt-14 muted">
  Thanks — and thanks to the U.S. Geological Survey for funding this work.
</div>
