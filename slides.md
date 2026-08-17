---
theme: default
title: Improving Water Data Access with Cloud Native Geospatial Formats
info: |
  ## Improving Water Data Access with Cloud Native Geospatial Formats

  A lightning talk on how the Geoconnex project (funded by the U.S. Geological Survey)
  uses FlatGeobuf and GeoParquet to serve national-scale hydrography straight out of
  object storage — no database, no API server, no derived subsets to keep in sync.
author: Colton Loftus
favicon: /cgs-logo.png
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

# Improving Water Data Access with Cloud Native Geospatial Formats

<div class="abs-bl m-8 text-left text-sm muted byline">
  Colton Loftus <br> Center for Geospatial Solutions <br>
  <a href="https://cgsearth.org" target="_blank">cgsearth.org</a>
</div>

<!-- ---
layout: center
class: text-center
--- -->

<!-- # Environmental Data is Highly Fragmented

<div v-click class="max-w-2xl mx-auto">
  <b>Example</b>: no single U.S. water data agency. We need federal government, states, utility companies, and universities to all contribute
</div>

<figure class="mt-3">
<svg viewBox="0 0 640 400" role="img" aria-label="Federal agencies, states, community organizations, academia, and utilities form a mesh, each needing to coordinate directly with every other, with no central water data agency in the middle" style="width:100%;max-width:640px;height:auto;margin:0 auto;display:block;color:var(--wtr-text)">
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
    <rect x="250" y="33" width="140" height="54" rx="8" fill="rgba(22,76,107,0.85)" stroke="rgba(201,222,232,0.42)"/>
    <text x="320" y="79">Federal agencies</text>
    <rect x="393" y="137" width="140" height="54" rx="8" fill="rgba(22,76,107,0.85)" stroke="rgba(201,222,232,0.42)"/>
    <text x="463" y="183">States</text>
    <rect x="338" y="304" width="140" height="54" rx="8" fill="rgba(22,76,107,0.85)" stroke="rgba(201,222,232,0.42)"/>
    <text x="408" y="350">Community orgs</text>
    <rect x="162" y="304" width="140" height="54" rx="8" fill="rgba(22,76,107,0.85)" stroke="rgba(201,222,232,0.42)"/>
    <text x="232" y="350">Academia</text>
    <rect x="107" y="137" width="140" height="54" rx="8" fill="rgba(22,76,107,0.85)" stroke="rgba(201,222,232,0.42)"/>
    <text x="177" y="183">Utilities</text>
  </g>
  <g fill="none" stroke="var(--wtr-sand)" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round">
    <g transform="translate(320,48)" aria-hidden="true">
      <path d="M-9,7 H9 M-9,7 V0 L0,-7 L9,0 V7 M-6,7 V0 M-2,7 V0 M2,7 V0 M6,7 V0"/>
    </g>
    <g transform="translate(463,152)" aria-hidden="true">
      <path d="M0,-9 C4.4,-9 8,-5.6 8,-1.5 C8,4 0,10 0,10 C0,10 -8,4 -8,-1.5 C-8,-5.6 -4.4,-9 0,-9 Z"/>
      <circle cx="0" cy="-1.5" r="2.4" fill="var(--wtr-sand)" stroke="none"/>
    </g>
    <g transform="translate(408,319)" aria-hidden="true">
      <circle cx="-4" cy="-5" r="3"/>
      <circle cx="4.5" cy="-3.5" r="3"/>
      <path d="M-9,8 C-9,2.5 -6,-0.5 -4,-0.5 C-2,-0.5 0,1.5 0,5"/>
      <path d="M0,8 C0,3 2,0.5 4.5,0.5 C7.5,0.5 9,3.5 9,8"/>
    </g>
    <g transform="translate(232,319)" aria-hidden="true">
      <path d="M-10,-2 L0,-7 L10,-2 L0,3 Z"/>
      <path d="M-5,0 V5 C-5,7.2 5,7.2 5,5 V0"/>
      <line x1="10" y1="-2" x2="10" y2="4"/>
    </g>
    <g transform="translate(177,152)" aria-hidden="true">
      <path d="M0,-9 C4,-4 7.5,0.3 7.5,4 C7.5,8.4 4.1,11 0,11 C-4.1,11 -7.5,8.4 -7.5,4 C-7.5,0.3 -4,-4 0,-9 Z"/>
    </g>
  </g>
</svg>
</figure> -->

---
layout: center
class: text-center statement
---

# Sharing environmental data often has an issue more fundamental than scaling:

<div v-click class="mt-7">
  <span class="punchline">Infrastructure.</span>
</div>

<div v-click class="mt-9 max-w-3xl mx-auto text-xl leading-relaxed">
  Database and API maintenance is expensive, time consuming, and requires expertise.
</div>

<div v-click class="mt-6 grid grid-cols-2 gap-4 max-w-3xl mx-auto text-left">
  <div class="card">Researchers and non-profits often don't have these resources long term</div>
  <div class="card">Government IT policy slows down deployment</div>
</div>

---
class: no-title-rule
---

# PostGIS is great, but isn't always the right fit for projects with resource constraints

<figure class="mt-6">
<svg viewBox="0 0 900 225" role="img" aria-label="A pipeline from source data through PostGIS, an API server, and a CDN cache. All but the source data must run continuously for years, and the database carries the most upkeep — maintenance and connection pooling." style="width:100%;max-width:760px;height:auto;margin:0 auto;display:block;color:var(--wtr-text)">
  <defs>
    <marker id="pipe-arrow-head" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <g stroke="currentColor" stroke-width="1.6" opacity="0.85" marker-end="url(#pipe-arrow-head)">
    <line x1="210" y1="80" x2="257" y2="80"/>
    <line x1="427" y1="80" x2="474" y2="80"/>
    <line x1="644" y1="80" x2="691" y2="80"/>
  </g>
  <g stroke-width="1.4">
    <rect x="40" y="25" width="170" height="110" rx="14" fill="rgba(22,76,107,0.7)" stroke="rgba(201,222,232,0.42)"/>
    <rect x="257" y="25" width="170" height="110" rx="14" fill="rgba(22,76,107,0.92)" stroke="var(--wtr-emphasis)" stroke-width="2.4"/>
    <rect x="474" y="25" width="170" height="110" rx="14" fill="rgba(22,76,107,0.7)" stroke="rgba(201,222,232,0.42)"/>
    <rect x="691" y="25" width="170" height="110" rx="14" fill="rgba(22,76,107,0.7)" stroke="rgba(201,222,232,0.42)"/>
  </g>
  <g transform="translate(125,55) scale(1.6)" fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="-8" y="-9" width="16" height="18" rx="1.5"/>
    <path d="M4,-9 v5 h5"/>
    <line x1="-4" y1="-2" x2="4" y2="-2"/>
    <line x1="-4" y1="2" x2="4" y2="2"/>
    <line x1="-4" y1="6" x2="2" y2="6"/>
  </g>
  <g transform="translate(342,55) scale(1.6)" fill="none" stroke="var(--wtr-emphasis)" stroke-width="1.5">
    <ellipse cx="0" cy="-7" rx="8" ry="3"/>
    <path d="M-8,-7 v10 a8,3 0 0 0 16,0 v-10"/>
    <path d="M-8,-2 a8,3 0 0 0 16,0" opacity="0.6"/>
  </g>
  <g transform="translate(559,55) scale(1.6)" fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="-8" y="-9" width="16" height="18" rx="1.5"/>
    <line x1="-8" y1="-3" x2="8" y2="-3"/>
    <line x1="-8" y1="3" x2="8" y2="3"/>
    <circle cx="5" cy="-6" r="1" fill="currentColor" stroke="none"/>
    <circle cx="5" cy="0" r="1" fill="currentColor" stroke="none"/>
  </g>
  <g transform="translate(776,55) scale(1.6)" fill="none" stroke="currentColor" stroke-width="1.3">
    <circle cx="-4" cy="-2" r="4"/>
    <circle cx="2" cy="-4" r="5"/>
    <circle cx="6" cy="-1" r="3.5"/>
    <rect x="-8" y="-2" width="16" height="6" rx="3"/>
  </g>
  <g style="font-size:15px;fill:var(--wtr-text)" text-anchor="middle">
    <text x="125" y="105">Source</text>
    <text x="125" y="122">data</text>
    <text x="342" y="113" style="font-size:16px;font-weight:700;fill:var(--wtr-text)">PostGIS</text>
    <text x="559" y="105">API</text>
    <text x="559" y="122">server</text>
    <text x="776" y="105">CDN /</text>
    <text x="776" y="122">cache</text>
  </g>
</svg>
</figure>

<div v-click class="mt-5 grid grid-cols-2 gap-4 text-base max-w-3xl mx-auto">
  <div class="card card-accent">
    <div class="eyebrow mb-1">Database maintenance</div>
    Upgrades, monitoring & backups are time-consuming. Managed DB services are expensive.
  </div>
  <div class="card card-accent">
    <div class="eyebrow mb-1">Boundaries to Sharing</div>
    Many permissions, settings, and export options. Researchers want the simplicity of a shapefile.
  </div>
</div>


---
layout: center
class: text-center
---

# What we needed for our Hydrological Data Pipelines

<div class="grid grid-cols-2 gap-5 mt-12 max-w-4xl mx-auto text-left text-lg">

  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <path d="M4 9 L10 3 L20 7 L18 18 L7 20 Z"/>
      <circle cx="4" cy="9" r="1.5"/>
      <circle cx="10" cy="3" r="1.5"/>
      <circle cx="20" cy="7" r="1.5"/>
      <circle cx="18" cy="18" r="1.5"/>
      <circle cx="7" cy="20" r="1.5"/>
    </svg>
    <div>Lossless geometries for hydrology analysis</div>
  </div>

  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <ellipse cx="11" cy="6" rx="7" ry="2.8"/>
      <path d="M4 6 v10 a7 2.8 0 0 0 14 0 V6"/>
      <path d="M4 11 a7 2.8 0 0 0 14 0"/>
      <line x1="3" y1="21" x2="21" y2="3"/>
    </svg>
    <div>No database connections or auth to manage</div>
  </div>

  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <circle cx="6" cy="12" r="2.6"/>
      <circle cx="18" cy="5.5" r="2.6"/>
      <circle cx="18" cy="18.5" r="2.6"/>
      <line x1="8.3" y1="10.7" x2="15.7" y2="6.8"/>
      <line x1="8.3" y1="13.3" x2="15.7" y2="17.2"/>
    </svg>
    <div>Present to clients at the US Geological Survey without copying huge files or managing subsets</div>
  </div>

  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <line x1="2.5" y1="20" x2="21.5" y2="20"/>
      <path d="M6 20 V14 M12 20 V9.5 M18 20 V4.5"/>
    </svg>
    <div>Simple scaling for hands-off data pipelines with lots of spatial joins</div>
  </div>

</div>

---
layout: center
class: text-center
---

# What if the file *is* the API?

<figure class="mt-8">
<svg viewBox="0 0 900 175" role="img" aria-label="A single file laid out as a row of byte blocks on object storage. The client reads the index at the head of the file, then issues range requests for only the two blocks of matching features; the rest of the file is never downloaded." style="width:100%;max-width:800px;height:auto;margin:0 auto;display:block">
  <defs>
    <marker id="fetch-arrow-index" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 z" fill="var(--cgs-green-light)"/>
    </marker>
    <marker id="fetch-arrow-feature" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 z" fill="var(--cgs-yellow)"/>
    </marker>
  </defs>

  <text x="50" y="18" style="font-size:14px;fill:var(--cgs-green-light)">1 · read the index</text>
  <text x="570" y="18" text-anchor="middle" style="font-size:14px;fill:var(--cgs-yellow)">2 · fetch only the matching byte ranges</text>

  <g stroke-width="1.8" fill="none">
    <line x1="70" y1="28" x2="70" y2="58" stroke="var(--cgs-green-light)" marker-end="url(#fetch-arrow-index)"/>
    <!-- One bracket forking from the label down to the two shaded blocks, so the
         callout reads as covering exactly those and nothing in between. -->
    <path d="M570 26 V38" stroke="var(--cgs-yellow)"/>
    <path d="M450 38 H690" stroke="var(--cgs-yellow)"/>
    <line x1="450" y1="38" x2="450" y2="58" stroke="var(--cgs-yellow)" marker-end="url(#fetch-arrow-feature)"/>
    <line x1="690" y1="38" x2="690" y2="58" stroke="var(--cgs-yellow)" marker-end="url(#fetch-arrow-feature)"/>
  </g>

  <!-- The file: 20 equal byte blocks. Only the filled ones cross the wire. -->
  <g>
    <rect x="50" y="64" width="800" height="56" rx="9" fill="rgba(22,76,107,0.55)" stroke="rgba(201,222,232,0.3)"/>
    <rect x="50" y="64" width="40" height="56" rx="9" fill="rgba(70,171,157,0.8)"/>
    <rect x="410" y="64" width="80" height="56" fill="rgba(249,198,9,0.85)"/>
    <rect x="650" y="64" width="80" height="56" fill="rgba(249,198,9,0.85)"/>
    <g stroke="rgba(201,222,232,0.22)" stroke-width="1">
      <path d="M90 64V120 M130 64V120 M170 64V120 M210 64V120 M250 64V120 M290 64V120
               M330 64V120 M370 64V120 M410 64V120 M450 64V120 M490 64V120 M530 64V120
               M570 64V120 M610 64V120 M650 64V120 M690 64V120 M730 64V120 M770 64V120 M810 64V120"/>
    </g>
    <rect x="50" y="64" width="800" height="56" rx="9" fill="none" stroke="rgba(201,222,232,0.4)" stroke-width="1.4"/>
  </g>

  <g stroke="rgba(201,222,232,0.35)" stroke-width="1">
    <path d="M50 130 V138 M850 130 V138 M50 134 H850"/>
  </g>
  <text x="450" y="158" text-anchor="middle" style="font-size:14.5px;fill:var(--wtr-muted)">one file on object storage — everything unshaded is skipped</text>
</svg>
</figure>

<div v-click class="mt-6 grid grid-cols-3 gap-4 max-w-4xl mx-auto text-left text-base">
  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <path d="M6.8 18.5 A4.6 4.6 0 0 1 7.2 9.3 A6.1 6.1 0 0 1 18.4 10.4 A4.1 4.1 0 0 1 17.7 18.5 Z"/>
    </svg>
    <div>Object storage already speaks <strong>HTTP range requests</strong></div>
  </div>
  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <rect x="3" y="4" width="18" height="16" rx="2"/>
      <rect x="6" y="7" width="6.5" height="5" rx="1"/>
      <rect x="15" y="7" width="3.5" height="3.5" rx="1"/>
      <rect x="6" y="14.5" width="9.5" height="3" rx="1"/>
    </svg>
    <div>The format puts a <strong>spatial index in the file</strong></div>
  </div>
  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <path d="M12 3 V13.5"/>
      <path d="M8 9.8 L12 13.8 L16 9.8"/>
      <path d="M4 17 v2.4 a1.6 1.6 0 0 0 1.6 1.6 h12.8 a1.6 1.6 0 0 0 1.6-1.6 V17"/>
    </svg>
    <div>So the client fetches <strong>only the bytes it needs</strong></div>
  </div>
</div>


---
class: text-center
---

# Two options for vector geometries with lossless encoding

<div class="text-lg muted mt-1">
Both just a file on object storage.
</div>

<div class="grid grid-cols-2 gap-8 mt-10">

<div>
  <div class="flex items-center justify-center gap-3">
    <img src="/fgb.svg" alt="" class="format-logo" />
    <h2 class="!m-0">FlatGeobuf</h2>
  </div>
  <div class="card card-accent mt-5 text-left">
    <div class="eyebrow mb-2">Packed Hilbert R-tree</div>
    A packed hilbert R-tree index in the head of the file - optimized for fast bbox queries
  </div>
</div>

<div>
  <div class="flex items-center justify-center gap-3">
    <img src="/parquet.svg" alt="" class="format-logo" />
    <h2 class="!m-0">GeoParquet</h2>
  </div>
  <div class="card card-accent mt-5 text-left">
    <div class="eyebrow mb-2">Row group metadata</div>
    Min/max statistics on <em>any</em> column. No R-Tree index for geometry, but not always needed.
  </div>
  <div class="card card-warn mt-4 mx-auto text-sm text-left" style="width: fit-content">
    Harder to optimize, but more flexible for analytics
  </div>
</div>

</div>

---
layout: center
class: text-center
---

<div class="flex items-center justify-center gap-5">
  <h1 class="!m-0 !text-3xl">What worked for us:</h1>
  <span class="punchline">FlatGeobuf</span>
  <img src="/fgb.svg" alt="" class="format-logo" style="width: 4.3rem; height: 4.3rem" />
</div>

<div class="grid grid-cols-2 gap-5 mt-12 max-w-4xl mx-auto text-left">
  <div class="card">
    <div class="eyebrow mb-1">Fewer dependencies for readers</div>
    A small JS library or GDAL — far less to pull in than the Parquet stack
  </div>
  <div class="card">
    <div class="eyebrow mb-1">Less to tune</div>
    No row group sizing or compression settings to get right, and the packed Hilbert
    R-tree is hard to beat
  </div>
</div>

---

# Using FlatGeobuf with two very different consumers

<div class="grid grid-cols-2 gap-6 mt-6">

```ts
// Browser-based visualization
import { deserialize } from 'flatgeobuf/lib/mjs/geojson.js'

const url = 'https://storage.googleapis.com/national-hydrologic-geospatial-fabric-reference-hydrofabric/reference_catchments_and_flowlines.fgb'
const bbox = map.getBounds().toArray().flat()

for await (const feature of deserialize(url, bbox)) {
  source.addFeature(feature) // renders as it streams
}
```

```python
# Notebook-based data analysis
import geopandas as gpd

gdf = gpd.read_file(
    "https://storage.googleapis.com/national-hydrologic-geospatial-fabric-reference-hydrofabric/reference_catchments_and_flowlines.fgb",
    bbox=(-105.1, 39.6, -104.7, 39.9),
)
```

</div>

<div v-click class="mt-6 card card-accent text-center">
  We can serve 2.7 million catchment polygons from the National Hydrologic Geospatial Fabric Reference Hydrofabric dataset using the same file.
</div>


---
layout: full
class: p-0 overflow-hidden
hideLogo: true
---

<div class="w-full h-full flex flex-col">
  <div class="px-6 pt-2 pb-2 flex items-baseline gap-4">
    <a href="https://colton.place/flatgeobuf-viewer/" target="_blank" class="text-sm muted !border-0">
      colton.place/flatgeobuf-viewer
    </a>
  </div>
  <iframe
    src="https://colton.place/flatgeobuf-viewer/?remote_url=https%3A%2F%2Fstorage.googleapis.com%2Fnational-hydrologic-geospatial-fabric-reference-hydrofabric%2Freference_catchments_and_flowlines.fgb&bbox=-102.192304%2C+40.820678%2C+-101.926099%2C+41.022213"
    class="flex-1 w-full border-0"
    title="FlatGeobuf viewer"
    loading="lazy"
    allow="fullscreen"
  />
</div>
---

# What Changed

<div class="mt-8 max-w-5xl mx-auto grid gap-x-5 gap-y-4 items-center text-lg"
     style="grid-template-columns: 1fr 2.75rem 1fr">

  <div class="eyebrow">Before</div>
  <div></div>
  <div class="eyebrow">After</div>

  <div class="card muted">Database connection issues during high-throughput pipelines</div>
  <div class="pipe-arrow text-center text-3xl">→</div>
  <div class="card card-accent">Easy scaling on S3</div>

  <div class="card muted">Manual exports that easily became out of sync</div>
  <div class="pipe-arrow text-center text-3xl">→</div>
  <div class="card card-accent">One URL providing the dataset's source of truth</div>

  <div class="card muted">Opaque versioning in the database, or added on the fly into the API response</div>
  <div class="pipe-arrow text-center text-3xl">→</div>
  <div class="card card-accent">Provenance metadata contained directly in the FlatGeobuf header</div>

</div>
---

# Next Step: utilizing OCI artifacts

<div class="text-lg mt-2 max-w-4xl">
OCI Artifacts are already utilized in the ML space for tracking models. 
<br>
We can do the same for geospatial reference data.
</div>

<div class="mt-8 flex items-center justify-center gap-6">

  <div class="card flex items-center gap-3 text-left">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <path d="M12 2.6 L20.5 7 v10 L12 21.4 L3.5 17 V7 Z"/>
      <path d="M3.5 7 L12 11.4 L20.5 7"/>
      <path d="M12 11.4 V21.4"/>
    </svg>
    <div>
      <div>catchments.fgb</div>
      <div class="muted text-xs">+ tags, digest, annotations</div>
    </div>
  </div>

  <div class="flex flex-col items-center gap-2">
    <span class="arrow-label">oras push</span>
    <svg viewBox="0 0 64 12" style="width:4.5rem;height:auto" aria-hidden="true">
      <path d="M0 6 H55" stroke="var(--wtr-accent-text)" stroke-width="1.7" fill="none"/>
      <path d="M49 1.5 L57 6 L49 10.5 Z" fill="var(--wtr-accent-text)"/>
    </svg>
  </div>

  <div class="flex flex-col gap-2 text-left">
    <div class="chip">
      <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3 L21 7.5 L12 12 L3 7.5 Z"/><path d="M3 12 L12 16.5 L21 12"/><path d="M3 16.5 L12 21 L21 16.5"/></svg>
      <span>Docker Hub</span>
    </div>
    <div class="chip">
      <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3 L21 7.5 L12 12 L3 7.5 Z"/><path d="M3 12 L12 16.5 L21 12"/><path d="M3 16.5 L12 21 L21 16.5"/></svg>
      <span>GitHub Container Registry</span>
    </div>
    <div class="chip">
      <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3 L21 7.5 L12 12 L3 7.5 Z"/><path d="M3 12 L12 16.5 L21 12"/><path d="M3 16.5 L12 21 L21 16.5"/></svg>
      <span>Google Artifact Registry</span>
    </div>
  </div>

</div>

<div class="mt-8 grid grid-cols-2 gap-5 max-w-4xl mx-auto text-sm text-left">
  <div class="card flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <path d="M3 12.6 V4.6 A1.6 1.6 0 0 1 4.6 3 h8 L21 11.4 a1.6 1.6 0 0 1 0 2.2 l-7.4 7.4 a1.6 1.6 0 0 1-2.2 0 Z"/>
      <circle cx="7.6" cy="7.6" r="1.4"/>
    </svg>
    <div>
      <div class="eyebrow mb-1">Versioning comes for free</div>
      Tags, digests, and annotations live in the manifest — provenance travels with the data
    </div>
  </div>
  <div class="card card-accent flex items-start gap-3">
    <svg class="card-icon" viewBox="0 0 24 24" aria-hidden="true">
      <circle cx="12" cy="5" r="2.4"/>
      <circle cx="5" cy="18" r="2.4"/>
      <circle cx="19" cy="18" r="2.4"/>
      <path d="M10.5 7.1 L6.5 15.9 M13.5 7.1 L17.5 15.9 M7.4 18 H16.6"/>
    </svg>
    <div>
      <div class="eyebrow mb-1">No single service to depend on</div>
      The same artifact pushes to any registry, so no one provider owns the dataset
    </div>
  </div>
</div>



---
layout: center
class: text-center
---

# Thank you! Any Questions?

<div class="mt-6 text-xl">
  Colton Loftus · Center for Geospatial Solutions
</div>

<div class="mt-2 text-lg">
  <a href="mailto:cloftus@lincolninst.edu">cloftus@lincolninst.edu</a>
</div>

<div class="mt-12 grid grid-cols-3 gap-4 text-sm max-w-4xl mx-auto">
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

<div class="mt-12 muted">
  Thanks to the U.S. Geological Survey for funding this work.
</div>
