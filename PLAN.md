# aef-crops-s1-s2-fiboa: plan

Pick a few fields, use their polygons as a mask, and stack everything under that mask: AEF embeddings per year,
Sentinel-2 and Sentinel-1 time series, and whatever ground truth exists. One marimo notebook runs the ETL and
serves an interactive dashboard to drill into fields, years and individual scenes.

## What the research says (2026-10)

- AEF is annual (2017-2025, v1.1, 64 dims, 10 m). It cannot see within-season dynamics. A Jan 2026 benchmark on US
  yield, tillage and cover crops found AEF competitive but with "limited sensitivity to temporal variation"
  ([arxiv 2601.00857](https://arxiv.org/abs/2601.00857v1)), so year-to-year crop health is the open question here,
  not a solved one.
- AEF as soil covariates: R2 0.21 to 0.69 against LUCAS soil properties across Europe
  ([EGU26-18609](https://meetingorganizer.copernicus.org/EGU26/EGU26-18609.html)). No AEF soil moisture study found.
- TESSERA v1.1 is also annual (128 dims). Italy covered 2017-2025.
- Field boundaries: FTW global predictions cover Italy for 2024 and 2025 (no crop type). Crop-labeled parcels exist
  openly only for Tuscany (`ftw/harmonized-field-data/it_1`, EuroCrops, 2016-2022) and Emilia-Romagna (AGREA
  "Piani Colturali Grafici", crop code and variety). Nothing open for Campania.
- Ground truth for performance: tomato yield is public only at province level (ISTAT, ANICAV, OI Nord). No open
  field-level yield in Italy.
- Soil moisture references: CLMS SSM 1 km (S1 based, daily, 2014 onward), ERA5-Land 0.1 deg, ESA CCI 0.25 deg. All
  coarser than a field.

## Candidate sites

| Site | Field size | Labels | Yield truth | Notes |
|---|---|---|---|---|
| San Marzano DOP, Sarno plain (14.47, 40.70, 14.72, 40.84) | small, often staked or netted | hand-picked only | Salerno/Napoli province (ISTAT) | the original idea, DOP plots are hand harvested on stakes |
| Piacenza or Parma, Emilia-Romagna | large open fields | AGREA parcels with tomato as a crop | province yield every year | largest northern processing-tomato area |
| Foggia, Tavoliere | large open fields | none open | province yield, drought years 2024-2025 | largest tomato province, strong drought signal |

## Data sources (no Planetary Computer dependency)

- AEF: Source Coop zarr `tge-labs/aef-mosaic` or UTM COGs `tge-labs/aef/v1/annual/` (flip rows), anonymous.
- TESSERA: Source Coop zarr `tessera/tessera/zarr/v1.1-dclimate`, anonymous.
- Sentinel-2: Element 84 Earth Search STAC (`sentinel-2-c1-l2a`, COGs on AWS, anonymous). SCL band for cloud masking.
- Sentinel-1: ASF. Find GRD granules with `asf_search`, then order radiometrically terrain-corrected (RTC) backscatter
  from HyP3 (free NASA Earthdata login, 8,000 credits a month, 60 credits per 10 m RTC job, so about 133 scenes a
  month). Products are full scenes, so download, clip to the field mask, keep only the clip. OPERA RTC-S1 would be
  ready-made but covers North America only.
- Fields: FTW global predictions (`admin:country_code=IT`, read over S3), plus AGREA or hand-drawn polygons.

## Dashboard sketch (one marimo notebook)

- Opens full screen: map of the area with field polygons; click a field (or several) to set the mask.
- Year strip, 2017 to 2025: AEF field-mean change to the previous year, and a year by year similarity matrix
  (viridis or cividis).
- Inside the field: per-pixel AEF clusters per year as small multiples, to see whether the same zones recur across
  crops (a soil signal) or move every year (a crop signal).
- Season panel: S2 NDVI/NDRE/NDMI and S1 VV/VH medians per field, one line per year, overlaid by day of year.
  Clicking a date shows that scene's chip at native 10 m pixels with the mask outline, nearest-neighbor, no smoothing.
- Truth panel: province yield per year next to the field's season summary (peak NDVI, season length, early VV).
- ETL caches to `data/` (gitignored) so the dashboard reloads instantly.

## Readable for non-specialists

Main labels say what a number means; index names stay available in tooltips and a technical toggle.

| Technical | Dashboard label |
|---|---|
| NDVI (S2) | Plant vigor (how green and leafy) |
| NDMI (S2) | Plant water (water held in leaves) |
| VV / VH backscatter (S1) | Ground wetness, plus crop structure as plants grow |
| AEF cosine to previous year | How different this year looked from last year |
| Within-field pixel clusters | Zones inside the field that behave alike |

- A "normal" band from the other years behind the selected year, so the read is above or below normal.
- One generated sentence per year (green-up timing, peak vs usual, wet or dry spells).
- Season events (green-up, peak, harvest drop) detected and labeled on the timeline.
- Above and below normal in blue and orange, never red and green.

## Open questions

- Site: San Marzano DOP, Sarno plain (decided).
- Sentinel-1: ASF HyP3 RTC (decided). One track or both, 10 m or coarser, full archive or a few years.
- How many fields: two or three hand-picked contrasts, or one field plus its neighbors.
