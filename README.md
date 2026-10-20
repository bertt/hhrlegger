# hhrlegger

- HHR legger - oppervlaktewateren: https://bertt.github.io/hhrlegger/oppervlaktewateren/
## Catalog preparation

The catalog is built with [Portolan](https://github.com/portolan-sdi/portolan-cli) and published to Azure Blob Storage.

### 1. Initialize the catalog

```bash
portolan init
```

### 2. Extract from ArcGIS

```bash
portolan extract arcgis https://rijnland.enl-mcs.nl/arcgis/rest/services/Leggers/Legger_Oppervlaktewater_Vigerend/MapServer --license CC-BY-4.0 --auto
```

### 3. Add PMTiles

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

- Browse: https://browser.moregeo.it/external/flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/collection.json
- Or clone the catalog:

```bash
portolan clone https://flatgeobuf.blob.core.windows.net/hhrlegger
```

### Result

| Layer | PMTiles | Style |
|-------|---------|-------|| watergang_as | [watergang_as.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/watergang_as.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_as/styles/default.json) |
| watergang_begroeiing | [watergang_begroeiing.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/watergang_begroeiing.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_begroeiing/styles/default.json) |
| watergang_beschermingszone | [watergang_beschermingszone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/watergang_beschermingszone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_beschermingszone/styles/default.json) |
| watergang_kernzone | [watergang_kernzone.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/watergang_kernzone.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_kernzone/styles/default.json) |
| watergang_vlak | [watergang_vlak.pmtiles](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/watergang_vlak.pmtiles) | [default.json](https://flatgeobuf.blob.core.windows.net/hhrlegger/watergang_vlak/styles/default.json) |

## Todo

- Fill AGENTS.md for catalog and datasets

## Demo

MapLibre demo: https://bertt.github.io/hhrlegger/oppervlaktewateren/

