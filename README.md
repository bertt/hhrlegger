# hhrlegger

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
4. Fill in the metadata per layer and for the catalog.
5. Push the catalog to Azure Blob Storage.
6. Use the result: the published PMTiles, styles and GeoParquet files, and the MapLibre demo.

## History of Portolan

Portolan is created by Chris Holmes (Cloud-Native Geospatial Foundation) with the idea of an SDI (Spatial Data Infrastructure) as "files in a bucket": cloud native formats plus STAC metadata on object storage, without servers. See [Portolan's origin story](https://latlong.blog/2026/09/portolans-origin-story-as-told-by-creator-chris-holmes.html).

- Current version: Portolan CLI 0.8 (pre-1.0, breaking changes are possible).
- Planned: version 1.0 in December.

Project site: https://www.portolan-sdi.org/

The name comes from the [portolan chart](https://en.wikipedia.org/wiki/Portolan_chart): the earliest known type of nautical charts, made in the late 13th and early 14th centuries in the Mediterranean region. Sailors used them to navigate between ports, a fitting name for a project that helps people find and navigate data.

Why this is a good name for the project:

- **Decentralized:** portolan charts were drawn and copied by many individual chartmakers across the Mediterranean, not issued by one central authority. In the same way, Portolan catalogs are published by anyone, on their own storage, and federated through a registry.
- **Practical:** the charts were made to be used on board, not to be perfect. Portolan likewise favors what works: static files in a bucket instead of heavy infrastructure.
- **No design committees:** the charts grew out of practice and sharing among users. Portolan builds on existing open formats and tools that are proven in use, instead of waiting for a big standardization process.

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

### 4. Metadata

Fill in `./portolan/metadata.yaml` for each layer, and in the catalog root.

### 5. Push

```bash
cd leggers_legger_oppervlaktewater_vigerend

export AZURE_STORAGE_ACCOUNT_KEY=<your-azure-storage-account-key>
portolan push https://flatgeobuf.blob.core.windows.net/hhrlegger/
```

The catalog is available at https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json

- Browse: https://browser.moregeo.it/external/flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json
- Or clone the catalog:

```bash
portolan clone https://flatgeobuf.blob.core.windows.net/hhrlegger
```

### Result

| Layer | PMTiles | Style | GeoParquet |
|-------|---------|-------|------------|
| watergang_as | [watergang_as.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/styles/default.json) | [watergang_as.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet) |
| watergang_begroeiing | [watergang_begroeiing.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/watergang_begroeiing.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/styles/default.json) | [watergang_begroeiing.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/watergang_begroeiing.parquet) |
| watergang_beschermingszone | [watergang_beschermingszone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/watergang_beschermingszone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/styles/default.json) | [watergang_beschermingszone.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/watergang_beschermingszone.parquet) |
| watergang_kernzone | [watergang_kernzone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/watergang_kernzone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/styles/default.json) | [watergang_kernzone.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/watergang_kernzone.parquet) |
| watergang_vlak | [watergang_vlak.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/watergang_vlak.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/styles/default.json) | [watergang_vlak.parquet](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/watergang_vlak.parquet) |

## Todo

- Fill AGENTS.md for catalog and datasets

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

The catalog is designed to be queried by AI agents as well (planned, not fully in place yet, see Todo).

- Each collection publishes GeoParquet plus an `AGENTS.md` (e.g. `watergang_as/AGENTS.md`), and there is one at catalog level describing layers, attributes, CRS, license and example queries.
- An agent starts at the catalog URL (`https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json`), follows the STAC links, reads the relevant `AGENTS.md` and queries the GeoParquet directly over HTTP with a tool such as DuckDB.
- Ready-made agent skills for Portolan catalogs: [portolan-skills](https://github.com/portolan-sdi/portolan-skills).

## Other Dutch Portolan catalogs

Source: [Portolan registry](https://www.portolan-sdi.org/registry)

| Catalog | Description | Link |
|---------|-------------|------|
| Portolan NL | Cloud-Native Dutch Geodata | https://source.coop/cholmes/portolan-nl |
| Studio Bereikbaar Data Lab | NWB | https://browser.portolan-sdi.org/#/external/s3.hidrive.strato.com/sb-portolan-lab/catalog.json |
| Gemeente Utrecht | By Bert Temme | https://browser.portolan-sdi.org/#/external/utrecht.blob.core.windows.net/catalog/catalog.json |

## Conclusion

With Portolan, a map with your own data (sourced from WFS, WMS or ArcGIS) can be run without the hosting, maintenance and license costs of databases and application servers. The data is converted to cloud native formats and deployed to S3-compatible blob storage, which serves it as static files. The only remaining cost is cheap blob storage.

The result serves both humans (maps in the browser via PMTiles and MapLibre) and AI agents (GeoParquet, STAC metadata and `AGENTS.md`).
