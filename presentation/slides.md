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
| COG | Raster | Partial reads via range requests, with overviews |
| COPC | Point clouds | LAZ with an octree index, stream only the points you need |
| Parquet | Tabular data | Columnar, read only the columns and row groups you need |
| GeoParquet | Vector, analysis | Columnar, fast to query with DuckDB |
| FlatGeobuf | Vector, streaming | Spatial index, bbox queries without a server |
| PMTiles | Map display | Single-file vector tiles, straight into MapLibre |
| STAC | Catalog | Discoverability and structure across all assets |

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

<div class="source">rijnland.maps.arcgis.com · MapServer: Leggers/Legger_Oppervlaktewater_Vigerend</div>

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

<div class="grid grid-cols-2 gap-6 mt-4">
<div class="shot" style="min-height:180px">📷 Screenshot: terminal, portolan extract running<br>public/screenshots/extract-terminal.png</div>
<div class="shot" style="min-height:180px">📷 Screenshot: resulting catalog folder (layers, parquet, styles)<br>public/screenshots/catalog-folder.png</div>
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

# Step 2: PMTiles per layer

Requires [tippecanoe](https://github.com/felt/tippecanoe) on the `PATH`.

```bash
portolan add watergang_as --pmtiles
portolan add watergang_begroeiing --pmtiles
portolan add watergang_beschermingszone --pmtiles
portolan add watergang_kernzone --pmtiles
portolan add watergang_vlak --pmtiles
```

<div class="shot mt-4">📷 Screenshot: terminal, portolan add --pmtiles (or file sizes of parquet vs pmtiles)<br>public/screenshots/add-pmtiles.png</div>

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
<div class="shot tall">📷 Layer README.md<br>public/screenshots/readme.png</div>
<div class="shot tall">📷 Layer AGENTS.md<br>public/screenshots/agents-md.png</div>
</div>

---

# Step 4: check

```bash
portolan check
```

<div class="lead">Powered by <b>rashid</b>, a validator and linter for Portolan catalogs. Validates STAC, links, assets and cloud-native formats. Fix the errors and warnings, check again, then push.</div>

<div class="shot tall mt-4">📷 portolan check output<br>public/screenshots/check.png</div>

---

# Step 5: push to Azure Blob Storage


```bash
export AZURE_STORAGE_ACCOUNT_KEY=<your-azure-storage-account-key>
portolan push https://flatgeobuf.blob.core.windows.net/hhrlegger/
```

<div class="grid grid-cols-[1fr_1fr] gap-6 mt-2">
<div class="card">

- Uploads `catalog.json`, collections, items and data files as static files
- Optimistic locking: refuses to overwrite a diverged remote without `--force`
- `--dry-run`, `--collection`, `--workers`
- Backends: S3 (and compatible), GCS, Azure Blob

</div>
<div class="shot">📷 Screenshot: Azure portal blob container or push output<br>public/screenshots/push.png</div>
</div>

<div class="source">Catalog: <a href="https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json">flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json</a></div>

---

# The result: a catalog of static files

| Layer | PMTiles | Style | GeoParquet | README | AGENTS |
|---|---|---|---|---|---|
| watergang_as | ✓ | ✓ | ✓ | ✓ | ✓ |
| watergang_begroeiing | ✓ | ✓ | ✓ | ✓ | ✓ |
| watergang_beschermingszone | ✓ | ✓ | ✓ | ✓ | ✓ |
| watergang_kernzone | ✓ | ✓ | ✓ | ✓ | ✓ |
| watergang_vlak | ✓ | ✓ | ✓ | ✓ | ✓ |

<div class="shot mt-4" style="min-height:220px">📷 Screenshot: Portolan browser showing the hhrlegger catalog<br>public/screenshots/portolan-browser.png</div>

<div class="source">browser.moregeo.it · portolan-browser</div>

---

# ArcGIS Online vs MapLibre: side by side

<div class="grid grid-cols-2 gap-4 mt-2">
<div>
<div class="shot tall">📷 Screenshot: ArcGIS Online map<br>public/screenshots/compare-arcgis.png</div>
<div class="caption">Esri map (ArcGIS Server + licences)</div>
</div>
<div>
<div class="shot tall">📷 Screenshot: MapLibre map<br>public/screenshots/compare-maplibre.png</div>
<div class="caption">MapLibre + PMTiles (static files on blob storage)</div>
</div>
</div>

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

<div class="mt-6 text-center" style="font-size:1.6em"><a href="https://bertt.github.io/hhrlegger/oppervlaktewateren/">Demo: bertt.github.io/hhrlegger/oppervlaktewateren</a></div>

<div class="source text-center">Basemap by OpenFreeMap</div>

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

<div class="grid grid-cols-2 gap-6 mt-4">
<div class="shot">📷 Screenshot: QGIS with the legger layers<br>public/screenshots/qgis.png</div>
<div class="shot">📷 Screenshot: GeoLibre catalog browser<br>public/screenshots/geolibre.png</div>
</div>

---

# Ask the data: AI agents

<div class="lead">Point an agent at <code>catalog.json</code>. It follows the STAC links, reads AGENTS.md and queries the GeoParquet over HTTP with DuckDB.</div>

<div class="grid grid-cols-2 gap-6 mt-2">
<div class="card">

**Q:** What is the water depth of the watergang at RD 93900, 463500?

**A:** Nearest is the Nieuwe Rijn (Leiden), 27 m away: depth 2.5 m, width 17.27 m, bottom width 9.77 m. Required depth per the legger, not measured.

</div>
<div class="shot">📷 Screenshot: Copilot CLI session answering the question<br>public/screenshots/agent-query.png</div>
</div>

---

# Federated query across two catalogs

<div class="grid grid-cols-2 gap-6 mt-2">
<div>

**Q:** Deepest 10 watergangen in the municipality of Leiden?

```sql
CREATE TEMP TABLE gem AS
SELECT geom g FROM read_parquet(
 'https://data.source.coop/cholmes/portolan-nl/kadaster/bestuurlijke_gebieden/gemeentegebied.parquet')
WHERE naam = 'Leiden';

SELECT a.CODE, a.NAAM, a.WATERDIEPTE
FROM read_parquet(
 'https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet') a, gem
WHERE ST_Intersects(a.geometry, gem.g)
ORDER BY a.WATERDIEPTE DESC LIMIT 10;
```

</div>
<div class="shot tall">📷 Screenshot: result table (Rijn 6.0 m, Galgewater 3.5 m, ...)<br>public/screenshots/federated.png</div>
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
<div class="shot tall">📷 Screenshot: portolan version list output / versions.json<br>public/screenshots/versioning.png</div>
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
<div class="shot">📷 Screenshot: Copilot CLI using portolan-bootstrap<br>public/screenshots/skills.png</div>
</div>

<div class="source">More: portolan-migrate, git-backed-catalog, enrich-metadata, portolan-thumbnails, report-catalog-issue, sourcecoop, customize-portolan-browser</div>

---

# Skills: example prompts

<div class="cards c3">
<div class="card">

“Use `portolan-bootstrap` to publish the GeoPackages in `data/` as a Portolan catalog.”

</div>
<div class="card">

“Mirror this ArcGIS FeatureServer as a Portolan catalog: `<URL>`”

</div>
<div class="card">

“Use `reading-portolan` to list the collections in `.../hhrlegger/catalog.json`.”

</div>
</div>

<div class="shot mt-4">📷 Screenshot: catalog-grader report for the hhrlegger catalog<br>public/screenshots/catalog-grader.png</div>

---

# The Portolan registry

<div class="grid grid-cols-[1fr_2fr] gap-8 items-center mt-2">
<div>
  <div class="big">85</div>
  <div class="lead">catalogs tracked, last checked 15 Oct 2026</div>
</div>
<div>

### A catalog of independently hosted catalogs
Catalogs from many publishers become searchable in one place, while the data stays distributed and is queried directly from the source. Anyone can submit a catalog by its URL with `register-catalog`.

</div>
</div>

<div class="shot mt-4">📷 Screenshot: Portolan registry<br>public/screenshots/registry.png</div>

<div class="source"><a href="https://www.portolan-sdi.org/registry">portolan-sdi.org/registry</a></div>

---

# Dutch Portolan catalogs

| Catalog | Description |
|---|---|
| Portolan NL | Cloud-native Dutch geodata (PDOK-inspired, Chris Holmes) |
| Studio Bereikbaar Data Lab | NWB |
| Gemeente Utrecht | By Bert Temme |

<div class="grid grid-cols-2 gap-6 mt-2">
<div class="shot">📷 Screenshot: Portolan NL on source.coop<br>public/screenshots/portolan-nl.png</div>
<div class="shot">📷 Screenshot: Utrecht catalog in the browser<br>public/screenshots/utrecht.png</div>
</div>

<div class="source">Source: <a href="https://www.portolan-sdi.org/registry">portolan-sdi.org/registry</a></div>

---

# OGC status of the formats

| Format | Status |
|---|---|
| STAC | OGC Community Standard (2025) |
| GeoParquet | OGC GeoParquet SWG working towards an Encoding Standard |
| PMTiles | Not an OGC standard, open community spec (Protomaps) |
| Portolan | Not an OGC standard, built on STAC, GeoParquet and PMTiles |

<div class="source">Status as of October 2026; check ogc.org for the latest.</div>

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
