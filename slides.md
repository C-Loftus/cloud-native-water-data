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

<p class="mt-6 text-xl muted">
  Serving national-scale hydrography from a bucket — no API to maintain
</p>

<div class="mt-14 flex justify-center gap-3 text-sm">
  <span class="card px-4 py-2">FlatGeobuf</span>
  <span class="card px-4 py-2">GeoParquet</span>
  <span class="card px-4 py-2">HTTP range requests</span>
</div>

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

# The problem usually isn't spatial efficiency

<div v-click class="mt-8 text-4xl font-bold" style="color: var(--wtr-accent)">
  It's infrastructure.
</div>

<div v-click class="mt-10 max-w-3xl mx-auto muted text-lg">
  Most organizations already have the data. What they don't have is somebody
  to keep a database, an API, and a tile server alive for the next ten years.
</div>

<!--
The framing for the whole talk. We are not here to shave bytes off a geometry encoding.
We are here because staffing a 24/7 API is the thing that actually kills data publishing
at small agencies and research groups.
-->

---

# The usual way to put data on the web

<div class="mt-8 flex items-stretch justify-between gap-2">
  <div class="pipe-box">Source<br>data</div>
  <div class="pipe-arrow">→</div>
  <div class="pipe-box">ETL<br>job</div>
  <div class="pipe-arrow">→</div>
  <div class="pipe-box">PostGIS</div>
  <div class="pipe-arrow">→</div>
  <div class="pipe-box">API<br>server</div>
  <div class="pipe-arrow">→</div>
  <div class="pipe-box">Tile<br>server</div>
  <div class="pipe-arrow">→</div>
  <div class="pipe-box">CDN /<br>cache</div>
</div>

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

<!--
Every box here is a thing with a version number, a security advisory feed, and an on-call
rotation. For a lot of teams this is the reason the data never ships at all.
-->

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

<!--
This is the second half of the pain. Not just "maintaining infra is expensive" but
"the workarounds for not having infra rot silently."
-->

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

<!--
This is the pivot. Cloud native geospatial formats move the index from the server into
the file, which means the "server" can be dumb storage.
-->

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

<!--
Credit where due: FlatGeobuf is Björn Harrtell's work. The key design decision is the
packed Hilbert R-tree at the front of the file — that's what makes range requests useful
instead of just "downloading a big file slowly."
-->

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

<!--
This is the punchline of the FlatGeobuf half. Internally our microservices read the exact
same object the frontend reads. Update the file, everything updates. Nothing drifts,
because there's nothing to drift from.
-->

---

# Geoconnex, in practice

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
    <div class="text-sm mt-1">Databases or API servers provisioned to serve them</div>
  </div>
</div>

<div class="grid grid-cols-2 gap-6 mt-10">

<div v-click>
<div class="eyebrow mb-2">Who reads it</div>

- The Geoconnex reference-feature microservices
- The public map at geoconnex.us
- Anyone with `ogr2ogr` and the URL

</div>

<div v-click>
<div class="eyebrow mb-2">What we deleted</div>

- A PostGIS instance and its backup schedule
- A tiling pipeline and its cache invalidation
- The "which export is current?" conversation

</div>

</div>

<!--
Geoconnex is USGS-funded, part of the Internet of Water. Reference features are the
persistent identifiers for hydrologic features — catchments and mainstems are the backbone.

[Confirm the exact catchment count and file size against the current build before presenting.]
-->

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

<!--
Demo beats slides. Point at the network tab: pan the map, watch a handful of 206 Partial
Content responses come back instead of a multi-gigabyte download. Nothing is running
server-side — that's a static bucket behind a CDN.

Fallback if the venue wifi is bad: open the viewer in a browser tab beforehand and keep
a screen recording ready.
-->

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

<!--
FlatGeobuf answers "give me the features here." GeoParquet answers "compute something
over a lot of features." Different questions, same hosting story.
-->

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

<!--
Worth saying out loud: this is ordinary SQL against a URL. No connection string, no
credentials, no server to scale when a class of 200 students hits it at once.
-->

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

<!--
Don't oversell GeoParquet for rendering — it has no spatial index in the FlatGeobuf sense,
and the bbox-covering approach is newer and less universally supported.
-->

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

<!--
Be honest here. Lightning talks that only show upside don't get believed. The write path
is the big caveat and it's exactly why this works so well for reference data.
-->

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

<!--
Tie it back to the opening claim: we didn't win by making the data smaller. We won by
deleting infrastructure.
-->

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

<!--
Questions. Likely ones: how big are the files, how often do you rebuild them, does this
work behind auth, and what about very high feature counts in one viewport.
-->
