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

# PostGIS is great, but often isn't the right fit for small organizations or ad-hoc projects

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
  <line x1="342" y1="135" x2="342" y2="158" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="559" y1="135" x2="559" y2="158" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <line x1="776" y1="135" x2="776" y2="158" stroke="currentColor" stroke-opacity="0.5" stroke-dasharray="3 3"/>
  <rect x="210" y="158" width="641" height="8" rx="4" fill-opacity="0.55" style="fill: var(--wtr-accent-text)"/>
  <polygon points="861,155 872,162 861,169" opacity="0.55" style="fill: var(--wtr-accent-text)"/>
  <text x="210" y="185" text-anchor="start" style="font-size:14px;fill: var(--wtr-accent-text)">today</text>
  <text x="861" y="185" text-anchor="end" style="font-size:14px;fill: var(--wtr-accent-text)">10 years later</text>
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

<div class="mt-10 flex items-center justify-center gap-4">
  <div class="pipe-box" style="min-width: 9rem">Browser</div>
  <div class="pipe-arrow text-sm">Range: bytes=<br>4096-65535 →</div>
  <div class="pipe-box card-accent" style="min-width: 9rem">
    One file<br><span class="muted text-xs">S3 / R2 / GCS / Azure Blob</span>
  </div>
</div>

<div v-click class="mt-12 grid grid-cols-3 gap-4 text-md max-w-4xl mx-auto">
  <div class="card">Object storage already speaks <strong>HTTP range requests</strong></div>
  <div class="card">The format puts a <strong>spatial index in the file</strong></div>
  <div class="card">So the client fetches <strong>only the bytes it needs</strong></div>
</div>

<div v-click class="mt-10 text-xl muted">
  No database. No API process. Just a bucket and a CDN.
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
    <img src="/fgb.svg" alt="" class="format-logo" style="height: 3.1rem" />
    <h2 class="!m-0">FlatGeobuf</h2>
  </div>
  <div class="card card-accent mt-5 text-left">
    <div class="eyebrow mb-2">Packed Hilbert R-tree</div>
    A packed hilbert r-tree index in the head of the file - optimized for fast bbox queries
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
class: text-center
---

# What worked for us

<div class="flex items-center justify-center gap-4 mt-6">
  <img src="/fgb.svg" alt="" class="format-logo" style="height: 3.6rem" />
  <span class="punchline">FlatGeobuf</span>
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
  <div class="pipe-arrow text-center text-2xl">→</div>
  <div class="card card-accent">Easy scaling on S3</div>

  <div class="card muted">Manual exports that easily became out of sync</div>
  <div class="pipe-arrow text-center text-2xl">→</div>
  <div class="card card-accent">One URL providing the dataset's source of truth</div>

  <div class="card muted">Opaque versioning in the database, or added on the fly into the API response</div>
  <div class="pipe-arrow text-center text-2xl">→</div>
  <div class="card card-accent">Provenance metadata contained directly in the FlatGeobuf header</div>

</div>
---

# Next Step: utilizing OCI artifacts

<div class="text-lg mt-2 max-w-4xl">
Container registries already store, version, and mirror arbitrary blobs — so a dataset can be
pushed as an OCI artifact and inherit all of it.
</div>

<div class="mt-8 flex items-center justify-center gap-5">
  <div class="pipe-box card-accent" style="min-width: 11rem">
    catchments.fgb<br><span class="muted text-xs"></span>
  </div>
  <div class="pipe-arrow text-sm">oras push →</div>
  <div class="flex flex-col gap-2">
    <div class="pipe-box" style="min-width: 15rem">Docker Hub</div>
    <div class="pipe-box" style="min-width: 15rem">GitHub Container Registry</div>
    <div class="pipe-box" style="min-width: 15rem">Google Artifact Registry</div>
  </div>
</div>

<div class="mt-8 grid grid-cols-2 gap-5 max-w-4xl mx-auto text-sm">
  <div class="card">
    <div class="eyebrow mb-1">Versioning comes for free</div>
    Tags, digests, and annotations live in the manifest — provenance travels with the data
  </div>
  <div class="card card-accent">
    <div class="eyebrow mb-1">No single service to depend on</div>
    The same artifact pushes to any registry, so no one provider owns the dataset
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
