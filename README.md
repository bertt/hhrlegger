# Open and Serverless Waterboard Legger: STAC, GeoParquet, PMTiles, and MapLibre

## Introduction

This project recreates the original Esri map of Hoogheemraadschap van Rijnland (HHR) with open tooling (Portolan catalog, PMTiles, MapLibre).

The approach is cloud native geo: there are no servers or databases to run. All data (GeoParquet, PMTiles, styles, STAC catalog) is static files deployed on an S3-compatible blob store (Azure Blob Storage) and read directly by clients over HTTP range requests.

The catalog metadata follows [STAC](https://stacspec.org/) (SpatioTemporal Asset Catalog), an open standard to describe geospatial data as JSON: a `catalog.json` links to collections, which describe their data files (assets) such as GeoParquet and PMTiles.

- Original Esri map: https://rijnland.maps.arcgis.com/apps/instant/sidebar/index.html?appid=f0fd839278474fe7841a12800fcc3e39
- HHR legger - oppervlaktewateren (recreated): https://bertt.github.io/hhrlegger/oppervlaktewateren/

This document walks through the following steps:

1. Install Portolan CLI and initialize a catalog.
2. Extract the layers from the ArcGIS MapServer (the source data).
3. Add PMTiles (vector tiles) for each layer.
4. Generate human readable documentation for each layer.
5. Fill the `AGENTS.md` files with an AI tool.
6. Push the catalog to Azure Blob Storage.
7. Use the result: the published PMTiles, styles and GeoParquet files, and the MapLibre demo.

## History of Portolan

Portolan is created by Chris Holmes (Cloud-Native Geospatial Foundation) with the idea of an SDI (Spatial Data Infrastructure) as "files in a bucket": cloud native formats plus STAC metadata on object storage, without servers. See [Portolan's origin story](https://latlong.blog/2026/09/portolans-origin-story-as-told-by-creator-chris-holmes.html).

- Current version: Portolan CLI 0.8 (pre-1.0, breaking changes are possible).
- Planned: version 1.0 in December.

Project site: https://www.portolan-sdi.org/

The name comes from the [portolan chart](https://en.wikipedia.org/wiki/Portolan_chart): medieval Mediterranean nautical charts (13th-14th century) that sailors used to navigate between ports. Like those charts, Portolan is decentralized (published by anyone, federated through a registry), practical (static files in a bucket) and built on proven open formats rather than design committees.

<img width="600" alt="Mediterranean portolan chart, 14th century" src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Mediterranean_chart_fourteenth_century2.jpg/960px-Mediterranean_chart_fourteenth_century2.jpg" />

*Mediterranean portolan chart, 14th century (Wikimedia Commons).*

## Catalog preparation

The catalog is built with [Portolan](https://github.com/portolan-sdi/portolan-cli) and published to Azure Blob Storage.

### 1. Install and initialize the catalog

Install Portolan CLI v0.8:

```bash
pip install portolan-cli==0.8.0
```

Then initialize:

```bash
portolan init
```

### 2. Extract from ArcGIS

```bash
portolan extract arcgis https://rijnland.enl-mcs.nl/arcgis/rest/services/Leggers/Legger_Oppervlaktewater_Vigerend/MapServer --license CC-BY-4.0 --auto
```

Extracting from a WFS works the same way:

```bash
portolan extract wfs "https://<wfs-host>/<path>/wfs?request=GetCapabilities&apikey=<YOUR_API_KEY>" administrative --license CC-BY-4.0
```

Query parameters such as an API key are supported through these pull requests:

- [portolan-sdi/portolan-cli#911](https://github.com/portolan-sdi/portolan-cli/pull/911): fix(extract): keep original query params in WMS style/legend URLs
- [geoparquet/geoparquet-io#836](https://github.com/geoparquet/geoparquet-io/pull/836): fix(wfs): keep existing query params (e.g. apikey) when building WFS GetFeature URLs

### 3. Add PMTiles

Requires [tippecanoe](https://github.com/felt/tippecanoe) to be installed and available on the `PATH`.

```bash
cd leggers_legger_oppervlaktewater_vigerend

portolan add watergang_as --pmtiles
portolan add watergang_begroeiing --pmtiles
portolan add watergang_beschermingszone --pmtiles
portolan add watergang_kernzone --pmtiles
portolan add watergang_vlak --pmtiles
```

### 4. Documentation

Run `portolan readme` in the catalog root. It creates human readable docs for each layer.

```
portolan readme
```

### 5. Fill the AGENTS.md files

Use an AI tool (e.g. GitHub Copilot CLI) to fill the `AGENTS.md` files. Each data layer ships with an `AGENTS.md` next to its Parquet file (e.g. `watergang_as/AGENTS.md`), plus one at the catalog root. It is a machine-readable guide written for LLMs and coding agents, and it complements this README. It contains:

- **Overview**: what the layer contains and when to use it
- **Accessing the data**: working DuckDB/Python snippets and the CRS
- **Schema & field notes**: column meanings, coded values and opaque columns
- **Data quality & usage notes**: nulls, quirks and caveats
- **Example queries**: only queries that were actually run, with result excerpts
- **Related collections**: join keys to the other layers

### 6. Push

```bash
cd leggers_legger_oppervlaktewater_vigerend

export AZURE_STORAGE_ACCOUNT_KEY=<your-azure-storage-account-key>
portolan push https://flatgeobuf.blob.core.windows.net/hhrlegger/
```

The catalog is available at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json

- Browse: https://browser.moregeo.it/external/flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json
- Or clone the catalog, and pull to update an existing local catalog:

```bash
portolan clone https://flatgeobuf.blob.core.windows.net/hhrlegger
portolan pull https://flatgeobuf.blob.core.windows.net/hhrlegger/
```

### Result

| Layer | PMTiles | Style | GeoParquet | README | AGENTS |
|-------|---------|-------|------------|--------|--------|
| watergang_as | [watergang_as.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/styles/default.json) | [watergang_as.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet) | [README.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/README.md) | [AGENTS.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/AGENTS.md) |
| watergang_begroeiing | [watergang_begroeiing.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/watergang_begroeiing.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/styles/default.json) | [watergang_begroeiing.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/watergang_begroeiing.parquet) | [README.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/README.md) | [AGENTS.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/AGENTS.md) |
| watergang_beschermingszone | [watergang_beschermingszone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/watergang_beschermingszone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/styles/default.json) | [watergang_beschermingszone.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/watergang_beschermingszone.parquet) | [README.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/README.md) | [AGENTS.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/AGENTS.md) |
| watergang_kernzone | [watergang_kernzone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/watergang_kernzone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/styles/default.json) | [watergang_kernzone.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/watergang_kernzone.parquet) | [README.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/README.md) | [AGENTS.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/AGENTS.md) |
| watergang_vlak | [watergang_vlak.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/watergang_vlak.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/styles/default.json) | [watergang_vlak.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/watergang_vlak.parquet) | [README.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/README.md) | [AGENTS.md](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/AGENTS.md) |

## Demo

MapLibre demo: https://bertt.github.io/hhrlegger/oppervlaktewateren/

<img width="800" height="469" alt="image" src="https://github.com/user-attachments/assets/5dd45ef5-6db2-441f-b3f7-4c269c522e00" />

The demo contains links to the PMTiles files and the style JSON files (`styles/default.json`) published in the catalog (see the Result table above).

The basemap is provided by [OpenFreeMap](https://openfreemap.org/).

Portolan catalogs can also be used in desktop GIS:

- QGIS: [Portolan Registry plugin](https://github.com/portolan-sdi/portolan-registry-qgis) (browse catalogs, add PMTiles and GeoParquet layers)
- GeoLibre: [GeoLibre](https://geolibre.app/) (version 3 and up) has a built-in Portolan catalog browser, no plugin needed

## OGC status of the formats

| Format | Status | Specification | Note |
|--------|--------|---------------|------|
| STAC | OGC Community Standard (published 2025) | [STAC spec](https://stacspec.org/en/about/stac-spec/) / [OGC 25-004](https://docs.ogc.org/cs/25-004/25-004.html) | Developed outside OGC, then endorsed as a Community Standard |
| GeoParquet | In progress: OGC GeoParquet SWG is working towards an OGC Encoding Standard | [GeoParquet spec](https://github.com/opengeospatial/geoparquet) | Version 1.0.0 released by the community in 2023; no adoption announcement verified yet |
| PMTiles | Not an OGC standard | [PMTiles v3 spec](https://github.com/protomaps/PMTiles/blob/main/spec/v3/spec.md) | Open community specification (Protomaps) |
| Portolan | Not an OGC standard | [Portolan spec](https://github.com/portolan-sdi/portolan-spec) | Specification and best practices for Portolan SDI, built on STAC, GeoParquet and PMTiles |

Status as of October 2026; check ogc.org for the latest.

## Querying the catalog with agents

The catalog is designed to be queried by AI agents as well.

- Each collection publishes GeoParquet plus an `AGENTS.md` (e.g. `watergang_as/AGENTS.md`), and there is one at catalog level describing layers, attributes, CRS, license and example queries.
- An agent starts at the catalog URL (`https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json`), follows the STAC links, reads the relevant `AGENTS.md` and queries the GeoParquet directly over HTTP with a tool such as DuckDB.

Point your agent at the `catalog.json` before asking it to query the data.

### Example

**Q:** I have a Portolan catalog of the water board at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json and I want to know the water depth of the watergang at 93900, 463500

**A:** The nearest watergang to RD (93900, 463500) is the Nieuwe Rijn (code 448-058-00086, in Leiden), about 27 m away.

- Water depth: 2.5 m
- Width: 17.27 m
- Bottom width: 9.77 m
- Status: Vastgesteld

The point does not lie exactly on the watergang. The next candidate is the Steenschuur at 65 m, with a water depth of 0.75 m. If your point is actually on that canal, 0.75 m applies. The values come from the legger (`watergang_as`). This is the required depth according to the legger, not a measured depth.

Query used:

```sql
SELECT CODE, NAAM, WATERDIEPTE, BREEDTE, BODEMBREEDTE,
       STATUSLEGGERWATERSYSTEEM,
       round(ST_Distance(geometry, ST_Point(93900, 463500)), 1) AS afstand_m
FROM read_parquet('https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet')
ORDER BY afstand_m
LIMIT 5;
```

### Federated query

An agent can combine data from several catalogs in a single query. The example below combines the water board catalog (Rijnland legger) with the [Portolan NL](https://source.coop/cholmes/portolan-nl) catalog (PDOK data).

**Q:** I have two Portolan catalogs: the water board legger at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json and Portolan NL (PDOK) at https://source.coop/cholmes/portolan-nl/catalog.json. Give the deepest watergangen (top 10) within the municipality of Leiden.

**A:** The municipality boundary of Leiden comes from Portolan NL (`kadaster/bestuurlijke_gebieden/gemeentegebied.parquet`), the water depth (`WATERDIEPTE`) from the legger (`watergang_as`). The three deepest are stretches of the Rijn (6.0 m), followed by Galgewater and Korte Vlietkanaal (3.5 m). These are the required depths according to the legger, not measured depths.

Query used:

```sql
INSTALL spatial; LOAD spatial;

-- PDOK catalog: boundary of Leiden
CREATE TEMP TABLE gem AS
SELECT geom g
FROM read_parquet('https://data.source.coop/cholmes/portolan-nl/kadaster/bestuurlijke_gebieden/gemeentegebied.parquet')
WHERE naam = 'Leiden';

-- Water board catalog: deepest watergangen intersecting Leiden
SELECT a.CODE, a.NAAM, a.WATERDIEPTE, round(a.LENGTE) AS lengte_m
FROM read_parquet('https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet') a, gem
WHERE ST_Intersects(a.geometry, gem.g)
ORDER BY a.WATERDIEPTE DESC
LIMIT 10;
```

Result:

| CODE | NAAM | WATERDIEPTE (m) | lengte_m |
|------|------|----------------:|---------:|
| 448-058-00135 | Rijn (traject Korte Vlietkanaal - Additionele kanaal) | 6.0 | 757 |
| 448-058-00643 | Rijn (traject Korte Vlietkanaal - Additionele kanaal) | 6.0 | 770 |
| 448-058-00162 | Rijn (traject Korte Vlietkanaal - Additionele kanaal) | 6.0 | 127 |
| 448-058-00200 | Galgewater | 3.5 | 224 |
| 448-058-00051 | Korte Vlietkanaal | 3.5 | 114 |
| 448-058-00022 | Galgewater | 3.5 | 190 |
| 449-058-02200 | Korte Vlietkanaal | 3.5 | 450 |
| 449-058-00054 | Korte Vlietkanaal | 3.5 | 1511 |
| 448-058-00024 | Rijn (traject Galgewater - Korte Vlietkanaal) | 3.5 | 1620 |
| 448-058-00198 | Rijn (traject Galgewater - Korte Vlietkanaal) | 3.5 | 58 |

`ST_Intersects` also returns segments that cross the municipal boundary. Use `ST_Intersection` or `ST_Within` to keep only the parts inside Leiden.

## Portolan skills

The [portolan-skills](https://github.com/portolan-sdi/portolan-skills) provide reusable workflows for working with Portolan catalogs. Install them for GitHub Copilot CLI (requires [uv](https://docs.astral.sh/uv/)):

```
copilot plugin marketplace add portolan-sdi/portolan-skills
copilot plugin install portolan@portolan-skills
```

Other agents: see the [installation instructions](https://github.com/portolan-sdi/portolan-skills#installation).

| Skill | Purpose |
|-------|---------|
| `portolan-cli` | Publish and manage catalogs (init, add, check, push, pull, sync, partitioning, format conversion) |
| `portolan-bootstrap` | Build a complete, documented catalog from a data source (open data portal, ArcGIS/WFS service, folder of files) |
| `reading-portolan` | Explore, query, and visualize catalog data (DuckDB, GeoParquet, COGs, PMTiles/MapLibre maps) |
| `portolan-migrate` | Bring an existing non-compliant catalog up to spec |
| `git-backed-catalog` | Maintain a catalog in git with CI validation |
| `enrich-metadata` | Fill `metadata.yaml` from an open data portal record |
| `portolan-thumbnails` | Generate framed collection thumbnails (requires Node.js 24.12+) |
| `catalog-grader` | Score a published catalog from A (best) to F (worst) with evidence gathered by `catalog-grader/scripts/collect_facts.py` |
| `register-catalog` | Submit a catalog to the registry |
| `report-catalog-issue` | Open a feedback issue for a registered catalog |
| `sourcecoop` | Upload data to Source Cooperative |
| `customize-portolan-browser` | Build a publisher-branded catalog browser |

### Usage examples

- "Use `portolan-bootstrap` to publish the GeoPackages in `data/` as a Portolan catalog."
- "Mirror this ArcGIS FeatureServer as a Portolan catalog: `<URL>`"
- "Use `reading-portolan` to list the collections in `https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json`."

## Other Dutch Portolan catalogs

Source: [Portolan registry](https://www.portolan-sdi.org/registry)

| Catalog | Description | Link |
|---------|-------------|------|
| Portolan NL | Cloud-Native Dutch Geodata | https://source.coop/cholmes/portolan-nl |
| Studio Bereikbaar Data Lab | NWB | https://browser.portolan-sdi.org/#/external/s3.hidrive.strato.com/sb-portolan-lab/catalog.json |
| Gemeente Utrecht | By Bert Temme | https://browser.portolan-sdi.org/#/external/utrecht.blob.core.windows.net/catalog/catalog.json |

## Further listening

[Cloud Native Geospatial for Small Teams](https://mapscaping.com/podcast/cloud-native-geospatial-for-small-teams/) (Mapscaping podcast): the Utah Geological Survey moved from a traditional ArcGIS Server setup to Postgres and GeoServer, and then to a cloud-native pipeline built on GeoParquet, PMTiles, and a STAC catalog. Turning off two servers saved around $25K/yr.

## Community

Questions about Portolan or cloud-native geospatial? Join the discussion in the [Cloud Native Geo Slack channel](https://cloudnativegeo.slack.com/archives/C0A1JBH9529).

## Conclusion

With Portolan, a map with your own data (sourced from WFS, WMS or ArcGIS) can be run without the hosting, maintenance and license costs of databases and application servers. The data is converted to cloud native formats and deployed to S3-compatible blob storage, which serves it as static files. The only remaining cost is cheap blob storage.

The result serves both humans (maps in the browser via PMTiles and MapLibre) and AI agents (GeoParquet, STAC metadata and `AGENTS.md`).
