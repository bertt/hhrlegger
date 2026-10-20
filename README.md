# hhrlegger

## Introduction

This project recreates the original Esri map of Hoogheemraadschap van Rijnland (HHR) with open tooling (Portolan catalog, PMTiles, MapLibre).

The approach is cloud native geo: there are no servers or databases to run. All data (GeoParquet, PMTiles, styles, STAC catalog) is static files deployed on an S3-compatible blob store (Azure Blob Storage) and read directly by clients over HTTP range requests.

- Original Esri map: https://rijnland.maps.arcgis.com/apps/instant/sidebar/index.html?appid=f0fd839278474fe7841a12800fcc3e39
- HHR legger - oppervlaktewateren (recreated): https://bertt.github.io/hhrlegger/oppervlaktewateren/

This document walks through the following steps:

1. Install Portolan CLI and initialize a catalog.
2. Extract the layers from the ArcGIS MapServer (the source data).
3. Add PMTiles (vector tiles) for each layer.
4. Fill in the metadata per layer and for the catalog.
5. Push the catalog to Azure Blob Storage.
6. Use the result: the published PMTiles, styles and GeoParquet files, and the MapLibre demo.

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

## Querying the catalog with agents

Note: the catalog is designed to be queried by AI agents as well. This is planned and not fully in place yet (see Todo).

- Each collection publishes its data as GeoParquet and an `AGENTS.md` next to it (e.g. `watergang_as/AGENTS.md`), plus one at catalog level.
- `AGENTS.md` describes, in plain language, what the catalog and each dataset contain: layers, attributes and their meaning, coordinate system, license and example queries.
- An agent starts at `catalog.json`, reads the relevant `AGENTS.md` to understand the data, and picks the right collection.
- It then queries the GeoParquet files directly over HTTP with a tool such as DuckDB (range requests, no server or download of the full file needed), for example `SELECT ... FROM 'https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.parquet'`.
- The agent answers the user's question from the query result.

How does an agent know about a Portolan catalog and its format?

- The user (or the agent's instructions) gives the catalog URL, e.g. `https://flatgeobuf.blob.core.windows.net/hhrlegger/catalog.json`.
- The catalog follows the open STAC standard (`catalog.json` links to a `collection.json` per layer, whose assets point to the `.parquet`, `.pmtiles` and style files). Agents already know STAC, so they can navigate it by following the links.
- The root `AGENTS.md` explains the catalog layout and how to query it (formats, URL patterns, tools to use), so no prior knowledge of Portolan is needed.
- Optionally, the catalog URL and a pointer to `AGENTS.md` can be added to the agent's own instructions or skill, so it knows where to look.

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
