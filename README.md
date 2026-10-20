# hhrlegger

## Introduction

This project recreates the original Esri map of Hoogheemraadschap van Rijnland (HHR) with open tooling (Portolan catalog, PMTiles, MapLibre).

The approach is cloud native geo: there are no servers or databases to run. All data (GeoParquet, PMTiles, styles, STAC catalog) is static files deployed on an S3-compatible blob store (Azure Blob Storage) and read directly by clients over HTTP range requests.

- Original Esri map: https://rijnland.maps.arcgis.com/apps/instant/sidebar/index.html?appid=f0fd839278474fe7841a12800fcc3e39
- HHR legger - oppervlaktewateren (recreated): https://bertt.github.io/hhrlegger/oppervlaktewateren/

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

## Conclusion

With Portolan, a map with your own data (sourced from WFS, WMS or ArcGIS) can be run without the hosting, maintenance and license costs of databases and application servers. The data is converted to cloud native formats and deployed to S3-compatible blob storage, which serves it as static files. The only remaining cost is cheap blob storage.
