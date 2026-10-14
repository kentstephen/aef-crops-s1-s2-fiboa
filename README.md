# aef-crops-s1-s2-fiboa

Crop and soil condition over time for a few tomato fields in the San Marzano DOP area (Sarno plain, Campania),
read from AlphaEarth Foundations (AEF) embeddings and checked against Sentinel-1 and Sentinel-2 time series under
the same field mask. Field boundaries come from Fields of the World (fiboa GeoParquet on Source Cooperative).

One marimo notebook runs the ETL and serves an interactive dashboard. See [PLAN.md](PLAN.md).

## Setup

```sh
uv venv
uv run marimo edit <notebook>.py --sandbox
```

## Data

- AEF: Google and Google DeepMind, CC BY 4.0, via Source Cooperative (`tge-labs/aef`).
- Fields of the World global predictions, CC BY 4.0, via Source Cooperative.
- Sentinel-2 L2A via Element 84 Earth Search; Sentinel-1 RTC via ASF HyP3. Contains modified Copernicus Sentinel data.
