# 04 · Data sources

Every input is free and open. Earth Engine IDs are given so each one can be opened in the [Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets).

| Dataset | Earth Engine ID / source | Resolution | Level | Used in | Provider & licence |
|---|---|---|---|---|---|
| Sentinel-2 MSI surface reflectance | `COPERNICUS/S2_SR_HARMONIZED` | 10–20 m, 5-day | L2A | Blocks 2, 3, 7 | Copernicus / ESA — free and open |
| MODIS thermal anomalies & fire, daily | `MODIS/061/MOD14A1`, `MODIS/061/MYD14A1` | 1 km, daily | L3 | Block 4 | NASA LP DAAC — open |
| Sentinel-5P TROPOMI methane | `COPERNICUS/S5P/OFFL/L3_CH4` | ~5.5 × 7 km, daily | L2 regridded | Block 6A | Copernicus / ESA — free and open |
| Sentinel-5P TROPOMI carbon monoxide | `COPERNICUS/S5P/OFFL/L3_CO` | ~5.5 × 7 km, daily | L2 regridded | Block 6A | Copernicus / ESA — free and open |
| CORINE Land Cover 2018 | `COPERNICUS/CORINE/V20/100m/2018` | 100 m | thematic | Block 1 | EEA / Copernicus Land — free and open |
| ESA WorldCover 2020 | `ESA/WorldCover/v100` | 10 m | thematic | Block 1 | ESA — CC BY 4.0 |
| ESA CCI Above-Ground Biomass v6 | `ESA/CCI/Above_Ground_Biomass/V6_0/<year>` | 100 m | L4 | Block 2 | ESA Climate Change Initiative — open |
| Copernicus DEM GLO-30 | `COPERNICUS/DEM/GLO30_2024_1` | 30 m | — | Block 1 | Copernicus — free and open |
| ERA5-Land daily aggregates | `ECMWF/ERA5_LAND/DAILY_AGGR` | ~9 km, daily | reanalysis | Block 2 | ECMWF / Copernicus C3S — open |
| Copernicus EMS rapid mapping | vector packages from the CEMS portal | vector | — | Block 1 | European Commission — free with attribution |

**Attribution line for reuse:** *Contains modified Copernicus Sentinel data (2016–2026) processed in Google Earth Engine; MODIS data courtesy NASA LP DAAC; ESA WorldCover and ESA CCI Biomass © ESA; ERA5-Land © ECMWF/C3S; Copernicus EMS © European Union.*

## Processing levels in one line each

| Level | Meaning |
|---|---|
| L0 | raw instrument packets |
| L1B / L1C | calibrated radiance or top-of-atmosphere reflectance, geolocated / orthorectified |
| **L2** | geophysical variable — surface reflectance, XCH₄, FRP (the working level here) |
| L3 | gridded and/or composited in time |
| L4 | modelled or classified product (land cover, biomass) |

A fuller table of missions, levels and SNAP operators is in [`reference/Missions_DataLevels_SNAP_Operators.pdf`](reference/Missions_DataLevels_SNAP_Operators.pdf).
