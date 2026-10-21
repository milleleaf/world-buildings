# world-buildings

Every building in the world that has a known height, with every field its source publishes, as GeoParquet and as 3D-ready
vector tiles, rebuilt for each [Overture Maps](https://overturemaps.org) release. About 206 million buildings and building parts.
Official building data is published beside Overture's, never merged: Hong Kong from
[milleleaf/hk-buildings](https://huggingface.co/datasets/milleleaf/hk-buildings), the Netherlands from [3DBAG](https://3dbag.nl).

Part of [Milleleaf](https://github.com/milleleaf): open, re-runnable pipelines over public data.

| Output | Where | Format |
|---|---|---|
| Every building with a height, every field, per region | [huggingface.co/datasets/milleleaf/world-buildings](https://huggingface.co/datasets/milleleaf/world-buildings) `v/<release>/<region>/*.parquet` | GeoParquet (WGS84) |
| Vector tiles per region, layer `buildings` (every field; height `height`, base `min_height`) | [huggingface.co/buckets/milleleaf/world-buildings](https://huggingface.co/buckets/milleleaf/world-buildings) `v/<release>/<region>.pmtiles` | PMTiles |
| The release and every region (source, bbox, rows, tile URL); start here | both, `index.json` | JSON |
| Row counts per source, sha256 per file, sources | both, `manifest.json` | JSON |
| The same files, mirrored per region | [kaggle.com/datasets/helenkwok/world-buildings-&lt;region&gt;](https://www.kaggle.com/datasets/helenkwok/world-buildings-r001) | uploaded after each region's publish |

Both Hugging Face locations answer CORS and HTTP Range requests (MapLibre with the `pmtiles` protocol; DuckDB for the Parquet).
The dataset card is `hf/world-buildings.md`.

## Run it

```bash
pip install -r requirements.txt               # plus tippecanoe 2.79 (CI builds it from source)
python buildings.py plan                      # data/out/plan.json: regions sized by Overture's own height counts
python buildings.py region <id>               # data/out/<id>/*.parquet + data/out/<id>.pmtiles
python publish_buildings.py region <id>       # -> Hugging Face under v/<release>/ (needs HF_TOKEN); nothing live changes
python publish_buildings.py finalize          # gate (manifest.py), then index.json + manifest.json, last
```

`--files 00058` limits `plan` to some of Overture's files (a probe); `--regions nl` adds regions to the release already
published; `SKY_DATA` moves the data folder.

The GitHub Action `.github/workflows/buildings.yml` checks every Monday (20:00 UTC) whether Overture has a release newer than
the one in the published `index.json`, and if so rebuilds and publishes the world (regions in parallel, a rerun skips regions
already published). By hand it runs a probe, some regions, or the world. Overture deletes each release about 60 days after
publishing it, so the release is always looked up in its STAC catalogue (`skylines.py`), never fixed.

## Height checks (Overture only): flagged, never removed

OpenStreetMap heights include errors (an 800 m chapel, an 816 m villa). Such a building keeps its row and gets a `check`
value: `floors` when its height divided by its floor count is over 12 m, `wikidata` when a whole building is 300 m or taller
without a plausible 2.5-6 m storey, unless a Wikidata building or tower within 300 m has a height within 15 % of it. Flagged
buildings are tiled from zoom 13 only, so a wrong spike does not stand in the skyline. Official sources are taken as published.

## Sources and terms

- Overture Maps Foundation buildings and building parts (which include OpenStreetMap data),
  [ODbL-1.0](https://opendatacommons.org/licenses/odbl/1-0/). Attribution: Overture Maps Foundation; © OpenStreetMap
  contributors. The Overture regions are a derived database, so they are ODbL-1.0 too.
- Region `hk`: Lands Department via the CSDI Portal, [CSDI Terms of Use](https://portal.csdi.gov.hk/csdi-webpage/doc/TNC).
- Region `nl`: © 3DBAG by tudelft3d and 3DGI, [CC BY 4.0](https://docs.3dbag.nl/en/copyright/), every layer and field.
- Height check only (not published): Wikidata (CC0). Coverage cut only: Eurostat GISCO, © EuroGeographics.
  Declared sources with licences: `sources.json`; CI checks them (`check.py`).

Outputs are provided as is, with no warranty, and are not endorsed by any source.

## History

This repo began on 2026-10-10 as **world-skylines**: only buildings of 30 m or more, zooms 6-13, published as
`world-skylines` by `publish.py` and a weekly `bake.yml`. world-buildings (every building with a height, every field) replaced
it the same day; the skylines bake was switched off, its Hugging Face outputs no longer exist, and its files were removed on
2026-10-22. `skylines.py` stays: `buildings.py` uses its release lookup, Wikidata and coverage helpers.

Code: Apache-2.0.
