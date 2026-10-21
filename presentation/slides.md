---
theme: default
title: 'Cloud-Native Geodata for People and AI Agents'
colorSchema: dark
aspectRatio: 16/9
canvasWidth: 1280
transition: slide-left
mdc: true
css: unocss
---

<div class="eyebrow">CLOUD-NATIVE GEOSPATIAL · PORTOLAN · MAPLIBRE</div>

# Cloud-Native Geodata for People and AI Agents

<div class="lead">From an ArcGIS Online waterschap legger to a serverless Portolan catalog and a MapLibre client</div>

---

# Today: database, OGC services, clients

<div class="diagram">
  <img src="/slide2-1.svg" alt="Diagram: PostGIS database connects via SQL to OGC services (WFS, WMS), which connect via HTTP to clients" />
</div>

<div class="card mt-4">

### Situation at Hoogheemraadschap van Rijnland / Schieland & Krimpenerwaard
Currently: **ArcGIS Server / GeoNode**

</div>

---

# Today: four pain points

<div class="cards c2">
<div class="card">

### Scalability
Server-side services must scale with every user and every query. Peak load means more servers.

</div>
<div class="card">

### Complex OGC APIs
WMS, WFS and OGC APIs need heavy server stacks, lots of configuration and client-specific knowledge.

</div>
<div class="card">

### High costs
Always-on infrastructure, maintenance and licences, even when the data rarely changes.

</div>
<div class="card">

### Not AI-ready
Capabilities documents and service-specific parameters are hard for agents to use reliably.

</div>
</div>

---

# The alternative: static files

<div class="lead">Data lives as cloud-native files in your own storage. No server runs queries: clients read only the bytes they need.</div>

<div class="cards c3">
<div class="card">

### Storage
Object storage (S3, GCS, Azure) with HTTP range requests and CORS.

</div>
<div class="card">

### Catalog
STAC describes what exists. README and AGENTS.md explain it to people and agents.

</div>
<div class="card">

### Clients
MapLibre, DuckDB, QGIS and AI agents read the same files.

</div>
</div>

---

# HTTP range requests: read part of a file

<div class="lead">A client asks for a byte range instead of the whole file. The server answers with only those bytes, so a 50 GB file is usable without downloading it.</div>

<div class="grid grid-cols-2 gap-6 mt-4">
<div class="card">

### Request and response

```http
GET /city.pmtiles
Range: bytes=0-16383

206 Partial Content
Content-Range: bytes 0-16383/48211904
```

</div>

<div class="card">

### How cloud-native formats use it
1. Read the header or footer: metadata and index.
2. Look up the byte offsets of the tiles, row groups or blocks you need.
3. Fetch only those ranges.

</div>
</div>

<div class="grid grid-cols-[2fr_1fr] gap-6 items-center mt-4">
<div class="lead">Part of HTTP since 1997 (RFC 2068); today in <a href="https://www.rfc-editor.org/rfc/rfc9110#name-range-requests">RFC 9110</a>.</div>
<img src="/secret-sauce.png" alt="The secret sauce" style="max-height:200px;margin-left:auto;border-radius:8px" />
</div>

---

# Building blocks

| Format | Used for | Why |
|---|---|---|
| <span class="hl">GeoParquet</span> | <span class="hl">Vector, analysis</span> | <span class="hl">Columnar, fast to query with DuckDB</span> |
| <span class="hl">PMTiles</span> | <span class="hl">Map display</span> | <span class="hl">Single-file vector tiles, straight into MapLibre</span> |
| <span class="hl">STAC</span> | <span class="hl">Catalog</span> | <span class="hl">Discoverability and structure across all assets</span> |
| COG | Raster | Partial reads via range requests, with overviews |
| COPC | Point clouds | LAZ with an octree index, stream only the points you need |
| Parquet | Tabular data | Columnar, read only the columns and row groups you need |
| FlatGeobuf | Vector, streaming | Spatial index, bbox queries without a server |

<div class="source"><span class="hl">Highlighted</span>: the formats used in the use case of this presentation</div>

---|---|---|
| COG | Raster | Partial reads via range requests, with overviews |
| COPC | Point clouds | LAZ with an octree index, stream only the points you need |
| Parquet | Tabular data | Columnar, read only the columns and row groups you need |
| GeoParquet | Vector, analysis | Columnar, fast to query with DuckDB |
| FlatGeobuf | Vector, streaming | Spatial index, bbox queries without a server |
| PMTiles | Map display | Single-file vector tiles, straight into MapLibre |

---

# Portolan: geodata as “files in a bucket”

<div class="grid grid-cols-[3fr_2fr] gap-8 items-center">
<div>

- Created by Chris Holmes (Cloud-Native Geospatial Foundation): development started January 2026
- Cloud-native formats plus STAC metadata on object storage, no servers
- Portolan CLI 0.8 (pre-1.0, breaking changes possible), 1.0 planned for December
- Named after the medieval *portolan charts*: decentralized, practical, built on proven formats

<div class="source"><a href="https://www.portolan-sdi.org/">portolan-sdi.org</a> · Portolan’s origin story (latlong.blog, 2026)</div>

<div class="mt-6">
<div class="source" style="margin-top:0">Supported from the start by</div>
<div class="flex items-center gap-8 mt-2">
<div style="background:var(--cream);border-radius:12px;padding:0.8rem 1.2rem"><img src="/logos/planet.svg" alt="Planet Labs" style="height:48px" /></div>
<div style="background:var(--cream);border-radius:12px;padding:0.8rem 1.2rem"><img src="/logos/carto.png" alt="CARTO" style="height:40px" /></div>
</div>
</div>

</div>
<div>
<img class="shot-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Mediterranean_chart_fourteenth_century2.jpg/960px-Mediterranean_chart_fourteenth_century2.jpg" alt="Mediterranean portolan chart, 14th century" />
<div class="caption">Mediterranean portolan chart, 14th century (Wikimedia Commons)</div>
</div>
</div>

---
layout: center
class: text-center
---

<div class="grid grid-cols-[3fr_2fr] gap-8 items-center text-left">
<div>

<div class="eyebrow">USE CASE</div>

# Waterboard legger: from ArcGIS Online to Portolan

<div class="lead">Hoogheemraadschap van Rijnland (HHR) · legger oppervlaktewateren<br>ArcGIS MapServer → Portolan catalog → PMTiles → MapLibre</div>

<div class="card mt-6">

### What is a legger?
The legal register of a waterschap that records the official shape and size of its waters: location, width, depth and bottom width of each watergang. It sets the rules for maintenance and for what is allowed in and next to the water, such as protection zones.

</div>

</div>
<div>
<img src="/screenshots/waterschappen-kaart.png" alt="Map of the 21 Dutch waterschappen (Unie van Waterschappen)" style="border-radius:12px;background:#fff" />
<div class="caption">The 21 Dutch waterschappen (Unie van Waterschappen)</div>
</div>
</div>
---

# The starting point: the legger in ArcGIS Online

<div class="grid grid-cols-[2fr_1fr] gap-6 mt-2">
<div>
<img class="shot-img" src="/screenshots/arcgis-original.png" alt="Original Esri map: 03. Legger Oppervlaktewater" />
</div>
<div class="card">

### The source
- ArcGIS Online app on top of an ArcGIS MapServer
- 5 layers: watergang as, begroeiing, beschermingszone, kernzone, vlak
- Three other leggers exist besides oppervlaktewater: primaire waterkeringen, regionale waterkeringen and ondersteunende kunstwerken
- Needs an ArcGIS Server and licences
- Digital sovereignty: ArcGIS Online is a US cloud service (Esri), so data, basemaps and platform depend on a non-European vendor and fall under foreign jurisdiction

</div>
</div>

<div class="source"><a href="https://rijnland.maps.arcgis.com/apps/instant/sidebar/index.html?appid=f0fd839278474fe7841a12800fcc3e39">ArcGIS Online map</a> · MapServer: Leggers/Legger_Oppervlaktewater_Vigerend</div>

---

# Creating a Portolan catalog in 5 steps

<div class="cards c5">
<div class="card"><div class="step">STEP 1</div>

### Extract
`portolan init`

`portolan extract arcgis <MapServer URL> --auto`

</div>
<div class="card"><div class="step">STEP 2</div>

### PMTiles
`portolan add watergang_as --pmtiles`

</div>
<div class="card"><div class="step">STEP 3</div>

### README.md
`portolan readme`

AGENTS.md filled by an AI tool

</div>
<div class="card"><div class="step">STEP 4</div>

### Check
`portolan check`

</div>
<div class="card"><div class="step">STEP 5</div>

### Push
`portolan push <blob URL>`

</div>
</div>

<div class="cards c5" style="margin-top:0.3rem;font-size:1.6em;color:var(--amber);text-align:center;line-height:1"><div>↓</div><div>↓</div><div>↓</div><div>↓</div><div>↓</div></div>

<div class="eyebrow mt-2">RESULT</div>

<div class="cards c5" style="margin-top:0.4rem">
<div class="card">

`.parquet` (GeoParquet)

`collection.json`

`catalog.json`

</div>
<div class="card">

`.pmtiles`

</div>
<div class="card">

`README.md` / `AGENTS.md`

</div>
<div class="card">

`report.json`

</div>
<div class="card">

Static files on blob storage

</div>
</div>

---

# Step 1: init and extract

```bash
pip install portolan-cli==0.8.0
portolan init

portolan extract arcgis https://rijnland.enl-mcs.nl/arcgis/rest/services/Leggers/Legger_Oppervlaktewater_Vigerend/MapServer --license CC-BY-4.0 --auto
```

<div class="grid grid-cols-1 gap-6 mt-4">
<div class="card" style="font-size:0.7em">

```text
leggers_legger_oppervlaktewater_vigerend/
├── catalog.json
├── README.md · AGENTS.md
├── .portolan/extraction-report.json
├── watergang_as/
│   ├── watergang_as.parquet
│   ├── collection.json
│   ├── styles/default.json
│   └── README.md · AGENTS.md
├── watergang_begroeiing/
├── watergang_beschermingszone/
├── watergang_kernzone/
└── watergang_vlak/
```

</div>
</div>

<div class="card mt-4">

### WFS with an API key
```bash
portolan extract wfs "https://<wfs-host>/<path>/wfs?request=GetCapabilities&apikey=<YOUR_API_KEY>" administrative --license CC-BY-4.0
```
Query parameters are kept thanks to two pull requests:
[portolan-sdi/portolan-cli#911](https://github.com/portolan-sdi/portolan-cli/pull/911) · [geoparquet/geoparquet-io#836](https://github.com/geoparquet/geoparquet-io/pull/836)

</div>

---

# Step 2: PMTiles with vector tiles per layer

Requires [tippecanoe](https://github.com/felt/tippecanoe) on the `PATH`.

```bash
portolan add watergang_as --pmtiles
portolan add watergang_begroeiing --pmtiles
portolan add watergang_beschermingszone --pmtiles
portolan add watergang_kernzone --pmtiles
portolan add watergang_vlak --pmtiles
```

<div class="card mt-4" style="font-size:0.7em">
<pre style="margin:0;background:none;font-family:monospace;line-height:1.5">watergang_as/
├── watergang_as.parquet     24 MB
├── <span class="hl" style="background:#fde68a;color:#7c2d12;padding:0 6px;border-radius:4px">watergang_as.pmtiles     36 MB   ← vector tiles</span>
├── watergang_as.thumb.jpg
├── collection.json
├── versions.json
├── styles/default.json
└── README.md · AGENTS.md</pre>
</div>

---

# Step 3: README.md and AGENTS.md

<div class="cards c2">
<div class="card">

### `portolan readme`
Human readable docs for each layer.

</div>
<div class="card">

### AGENTS.md
Filled with Copilot CLI: overview, DuckDB snippets, schema notes, quality notes, example queries, related collections.

</div>
</div>

<div class="grid grid-cols-2 gap-4 mt-4">
<img class="shot-img" src="/screenshots/readme.png" alt="Layer README.md" style="height:340px;object-fit:cover;object-position:top" />
<img class="shot-img" src="/screenshots/agents-md.png" alt="Layer AGENTS.md" style="height:340px;object-fit:cover;object-position:top" />
</div>

---

# Step 4: check

```bash
portolan check
```

<div class="lead">Powered by <b>rashid</b>, a validator and linter for Portolan catalogs. Validates STAC, links, assets and cloud-native formats. Fix the errors and warnings, check again, then push.</div>

---

# Step 5: push to Azure Blob Storage


```bash
export AZURE_STORAGE_ACCOUNT_KEY=<your-azure-storage-account-key>
portolan push https://flatgeobuf.blob.core.windows.net/hhrlegger/
```

<div class="grid grid-cols-[1fr_1fr] gap-6 mt-2">
<div class="card">

- Uploads `catalog.json`, collections, items and data files as static files
- Backends: S3 (and compatible), GCS, Azure Blob

</div>
<img class="shot-img" src="/screenshots/azure-blob.png" alt="Azure blob container hhrlegger" />
</div>

<div class="source">Catalog: <a href="https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json">flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json</a></div>

---

# The result: a catalog of static files

<img class="shot-img" src="/screenshots/portolan-browser.png" alt="The hhrlegger catalog in the Portolan browser" style="max-height:380px;width:auto;margin:0 auto;display:block" />

<div class="source"><a href="https://browser.moregeo.it/external/flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json">Browse the catalog in the Portolan browser</a> · <a href="https://github.com/portolan-sdi/portolan-browser">portolan-browser</a></div>

---

# watergang_as

<img class="shot-img" src="/screenshots/portolan-browser-collection.png" alt="The watergang_as collection in the Portolan browser" style="max-height:400px;width:auto;margin:0 auto;display:block" />

<div class="source">GeoParquet, categorical style and PMTiles assets, with README and AGENTS.md for people and agents</div>

---

# Result: MapLibre map

<img class="shot-img" src="/screenshots/maplibre-result.png" alt="MapLibre map with PMTiles from blob storage" style="max-height:400px;width:auto;margin:0 auto;display:block" />
<div class="caption">MapLibre + PMTiles (static files on blob storage). Basemap: <a href="https://openfreemap.org">OpenFreeMap</a> tiles</div>

<div class="mt-6 text-center" style="font-size:1.6em"><a href="https://bertt.github.io/hhrlegger/oppervlaktewateren/">Demo: bertt.github.io/hhrlegger/oppervlaktewateren</a></div>

---

# What changes

| | ArcGIS Online / Server | Portolan + MapLibre |
|---|---|---|
| Hosting | Application server and database | Blob storage, static files |
| Cost | Hosting, maintenance, licences | Cheap storage only |
| Scaling | Scale the servers | Storage and CDN scale out |
| Data access | Service-specific REST API | Open files over HTTP |
| Styles | Esri-specific | MapLibre style JSON in the catalog |
| AI agents | Hard to use reliably | STAC + AGENTS.md + GeoParquet |

---

# Also in desktop GIS

<div class="grid grid-cols-2 gap-6 mt-2">
<div class="card">

### QGIS
[Portolan Registry plugin](https://github.com/portolan-sdi/portolan-registry-qgis): browse catalogs, add PMTiles and GeoParquet layers.

</div>
<div class="card">

### GeoLibre
[GeoLibre](https://geolibre.app/) 3 and up has a built-in Portolan catalog browser, no plugin needed.

</div>
</div>

<img class="shot-img mt-4" src="/screenshots/geolibre.png" alt="GeoLibre catalog browser" style="height:330px;width:auto;margin:1rem auto 0;display:block" />

---

# Ask the data: AI agents

<div class="lead">Point an agent at <code>catalog.json</code>. It follows the STAC links, reads AGENTS.md and queries the GeoParquet over HTTP with DuckDB.</div>

<div style="background:#0d1117;color:#e6edf3;border-radius:12px;padding:16px 22px;font-family:Consolas,monospace;font-size:0.5em;line-height:1.45;text-align:left;margin-top:8px">
<pre style="margin:0;background:none;color:inherit;font:inherit;white-space:pre-wrap"><span style="color:#58a6ff">&gt;</span> I have a Portolan catalog of the water board at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json
  and I want to know the water depth of the watergang at 93900, 463500
&nbsp;
<span style="color:#d2a8ff">skill(reading-portolan)</span>
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Fetch catalog (irm catalog.json, curl AGENTS.md)</span>
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Read collection guide (watergang_as/AGENTS.md)</span>
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Query nearest watergang (DuckDB spatial + httpfs on the parquet)</span>
&nbsp;
<span style="color:#fbbf24">The watergang at 93900, 463500 (RD New) has a legger water depth of 2.5 m.</span>
&nbsp;
It is the "Nieuwe Rijn" (CODE 448-058-00086), the only watergang within 50 m; its centre
line lies about 27 m from your point. Width 17.27 m, bottom width 9.77 m, status "Vastgesteld".
&nbsp;
<span style="color:#8b949e">WATERDIEPTE is the required legger depth, not a measured one.
Source: watergang_as, CC-BY-4.0 (Hoogheemraadschap van Rijnland).</span></pre>
</div>

---

# Federated query across two catalogs

<div style="background:#0d1117;color:#e6edf3;border-radius:12px;padding:12px 20px;font-family:Consolas,monospace;font-size:0.5em;line-height:1.3;text-align:left;margin-top:6px">
<pre style="margin:0;background:none;color:inherit;font:inherit;white-space:pre-wrap"><span style="color:#58a6ff">&gt;</span> I have two Portolan catalogs: the water board legger at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json
  and Portolan NL (PDOK) at https://source.coop/cholmes/portolan-nl/catalog.json.
  Give the deepest watergangen (top 10) within the municipality of Leiden.
&nbsp;
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Fetch PDOK catalog, explore kadaster and cbs</span>
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Read municipality collection guide (bestuurlijke_gebieden/AGENTS.md)</span>
<span style="color:#7ee787">●</span> Shell  <span style="color:#8b949e">Top 10 deepest watergangen in Leiden (DuckDB spatial join over HTTP)</span>
&nbsp;
<span style="color:#fbbf24">Top 10 deepest watergang segments in Leiden, by legger WATERDIEPTE (m):</span>
&nbsp;
 #   CODE            Name                                          Depth  Width  Length
 1   448-058-00135   Rijn (Korte Vlietkanaal–Additionele kanaal)   6.0    59.2   757
 2   448-058-00643   Rijn (same section)                           6.0    44.0   770
 3   448-058-00162   Rijn (same section)                           6.0    32.3   127
 4   448-058-00198   Rijn (Galgewater–Korte Vlietkanaal)           3.5    54.8   58
 5   448-058-00024   Rijn (Galgewater–Korte Vlietkanaal)           3.5    53.4   1620
 6   448-058-00200   Galgewater                                    3.5    40.7   224
 7   449-058-02200   Korte Vlietkanaal                             3.5    34.4   450
 8   449-058-00054   Korte Vlietkanaal                             3.5    32.8   1511
 9   448-058-00022   Galgewater                                    3.5    24.8   190
10   448-058-00051   Korte Vlietkanaal                             3.5    22.0   114
&nbsp;
<span style="color:#8b949e">Leiden boundary from PDOK (bestuurlijke_gebieden/gemeentegebied); watergang_as lines intersecting it,
excluding "Verwijderd" and null depths. Both layers EPSG:28992. Many segments tie at 3.5 m (ranks 4–10: cut by width).</span></pre>
</div>

<div class="source">Boundaries from Portolan NL (PDOK), depths from the HHR legger</div>

---
layout: center
class: text-center
---

<div class="eyebrow">ADVANCED PORTOLAN</div>

# Versioning, skills and more

---

# Versioning

<div class="grid grid-cols-[3fr_2fr] gap-6 mt-2">
<div>

```bash
portolan version bump watergang_as 1.1.0 -m "Updated source data"
portolan version bump watergang_as 2.0.0 --breaking -m "Schema change"
portolan version current watergang_as
portolan version list watergang_as --json
```

- `bump` compares checksums and records a new entry in `versions.json`
- Explicit semver, `--breaking` marks schema changes
- Backends: `file` or `iceberg`
- Iceberg only: `rollback` (instant, no data copied) and `prune --keep N`

</div>
<div style="background:#0d1117;color:#e6edf3;border-radius:12px;padding:14px 18px;font-family:Consolas,monospace;font-size:0.5em;line-height:1.45;text-align:left">
<pre style="margin:0;background:none;color:inherit;font:inherit;white-space:pre-wrap"><span style="color:#58a6ff">$</span> portolan version list watergang_as
&nbsp;
<span style="color:#7ee787">→</span> Versions for 'watergang_as' (2 total):
&nbsp;
<span style="color:#7ee787">→</span>   <span style="color:#fbbf24">1.0.0</span>  2026-10-21 12:43:44
      watergang_as.parquet
<span style="color:#7ee787">→</span>   <span style="color:#fbbf24">1.0.1</span>  2026-10-21 12:53:13 — Generated PMTiles and thumbnail: watergang_as.pmtiles, watergang_as.thumb.jpg
      watergang_as.pmtiles
      watergang_as.thumb.jpg</pre>
</div>
</div>

---

# Portolan skills

<div class="lead">Reusable agent workflows from <a href="https://github.com/portolan-sdi/portolan-skills">portolan-skills</a></div>

```bash
copilot plugin marketplace add portolan-sdi/portolan-skills
copilot plugin install portolan@portolan-skills
```

<div class="grid grid-cols-2 gap-6 mt-2">
<div>

| Skill | Purpose |
|---|---|
| `portolan-cli` | Publish and manage catalogs |
| `portolan-bootstrap` | Build a documented catalog from a source |
| `reading-portolan` | Explore, query, visualize |
| `catalog-grader` | Score a catalog from A to F |
| `register-catalog` | Submit to the registry |

</div>
<div class="card" style="font-size:0.75em">

**Example prompts**

- “Use `portolan-bootstrap` to publish the GeoPackages in `data/` as a Portolan catalog.”
- “Mirror this ArcGIS FeatureServer as a Portolan catalog: `<URL>`”
- “Use `reading-portolan` to list the collections in `.../hhrlegger/catalog.json`.”

</div>
</div>

<div style="background:#0d1117;color:#e6edf3;border-radius:12px;padding:10px 20px;font-family:Consolas,monospace;font-size:0.45em;line-height:1.35;text-align:left;margin-top:8px">
<pre style="margin:0;background:none;color:inherit;font:inherit;white-space:pre-wrap"><span style="color:#58a6ff">&gt;</span> Grade the catalog at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json
<span style="color:#d2a8ff">skill(catalog-grader)</span>
<span style="color:#fbbf24">Catalog grade: D</span>  <span style="color:#8b949e">(rashid reports 8 errors, which caps the grade at D)</span>
 1. No provider with the producer role in all 5 collections
 2. Storage host: Accept-Ranges and CORS headers missing
 3. 5 broken links in the root README (404)
 4. 11 of 20 documented code blocks fail</pre>
</div>

<div class="source">More skills: portolan-migrate, git-backed-catalog, enrich-metadata, portolan-thumbnails, report-catalog-issue, sourcecoop, customize-portolan-browser</div>

---

# The Portolan registry

<div class="grid grid-cols-[1fr_2fr] gap-8 items-center mt-2">
<div>
  <div class="big">85</div>
  <div class="lead">catalogs tracked, last checked 21 Oct 2026</div>
</div>
<div>

### A catalog of independently hosted catalogs
Catalogs from many publishers become searchable in one place, while the data stays distributed and is queried directly from the source. Anyone can submit a catalog by its URL with `register-catalog`.

</div>
</div>

<img class="shot-img mt-4" src="/screenshots/registry.png" alt="Portolan registry" style="height:300px;width:auto;margin:1rem auto 0;display:block" />

<div class="source"><a href="https://www.portolan-sdi.org/registry">portolan-sdi.org/registry</a></div>

---

# Dutch Portolan catalogs

| Catalog | Description |
|---|---|
| Portolan NL | Cloud-native Dutch geodata (PDOK-inspired, Chris Holmes) |
| Studio Bereikbaar Data Lab | NWB |
| Gemeente Utrecht | By Bert Temme |

<img class="shot-img mt-2" src="/screenshots/portolan-nl.png" alt="Portolan NL on source.coop" style="height:300px;width:auto;margin:0.5rem auto 0;display:block" />

<div class="source">Source: <a href="https://www.portolan-sdi.org/registry">portolan-sdi.org/registry</a></div>

---

# Real-world savings

<div class="grid grid-cols-[1fr_2fr] gap-8 items-center mt-4">
<div>
  <div class="big">$25K</div>
  <div class="lead">saved per year</div>
</div>
<div>

The Utah Geological Survey moved from ArcGIS Server, to Postgres and GeoServer, to a cloud-native pipeline with GeoParquet, PMTiles and STAC. Turning off two servers saved around $25K a year.

<div class="source">Mapscaping podcast: <a href="https://mapscaping.com/podcast/cloud-native-geospatial-for-small-teams/">Cloud Native Geospatial for Small Teams</a></div>

</div>
</div>

---

# Portolan on GitHub

| Repository | Description |
|---|---|
| [portolan-cli](https://github.com/portolan-sdi/portolan-cli) | CLI for cloud-native geospatial data |
| [portolan-spec](https://github.com/portolan-sdi/portolan-spec) | Specification and best practices |
| [portolan-skills](https://github.com/portolan-sdi/portolan-skills) | Agent skills |
| [portolan-browser](https://github.com/portolan-sdi/portolan-browser) | Vue UI for browsing STAC catalogs |
| [portolan-registry](https://github.com/portolan-sdi/portolan-registry) | Unified catalog registry |
| [portolan-registry-qgis](https://github.com/portolan-sdi/portolan-registry-qgis) | QGIS plugin |
| [rashid](https://github.com/portolan-sdi/rashid) | Validator and linter |
| [portolan-catalog-template](https://github.com/portolan-sdi/portolan-catalog-template) | Git-backed catalog template |

<div class="source"><a href="https://github.com/portolan-sdi">github.com/portolan-sdi</a> · <a href="https://www.portolan-sdi.org/">portolan-sdi.org</a></div>

---

# Community: join us

<div class="grid grid-cols-[2fr_3fr] gap-6 mt-2 items-center">
<div class="card">

### Slack
Cloud Native Geo Slack: **more than 1,000 members**, with a dedicated #portolan channel. [Join the channel](https://cloudnativegeo.slack.com/archives/C0A1JBH9529)

### Core team
Nissim Lebovits (technical lead), Cayetano Benavent, Youssef Harby, Chris Holmes, Matthias Mohr, Javier de la Torre. Weekly open meetings.

</div>
<img class="shot-img" src="/screenshots/slack.png" alt="The #portolan channel in the Cloud Native Geo Slack" />
</div>

---

# Conclusion

<div class="cards c2">
<div class="card">

### Cheaper
No database, application server or licence costs. Only blob storage.

</div>
<div class="card">

### Simpler
Files instead of servers: fewer moving parts to run.

</div>
<div class="card">

### Scalable
Storage and CDN scale out, with no query server.

</div>
<div class="card">

### AI-ready
STAC plus AGENTS.md makes data understandable to agents.

</div>
</div>

<div class="grid grid-cols-[3fr_1fr] gap-8 items-center mt-8">
<div>

### Get started today
`uv tool install portolan-cli` · [portolan-sdi.org](https://www.portolan-sdi.org/) · [github.com/bertt/hhrlegger](https://github.com/bertt/hhrlegger)

### These slides
[bertt.github.io/hhrlegger/presentation](https://bertt.github.io/hhrlegger/presentation/)

</div>
<img src="/qr-slides.svg" alt="QR code to the slides" style="width:180px;border-radius:8px" />
</div>
