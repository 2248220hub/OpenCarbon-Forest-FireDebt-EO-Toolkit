# 02 · Run it on your own forest fire

The toolkit is built so that **only Block 0 changes** between fires. This page walks through each choice, in the order you should make it.

## Checklist

- [ ] **The fire** — dates and a rough location (EFFIS, Copernicus EMS, NASA FIRMS or news reports)
- [ ] **`AOI_BBOX`** — the burn scar plus a 1–2 km margin, `[west, south, east, north]` in degrees
- [ ] **`CTRL_BBOX`** — an unburnt forest of the same type, elevation and climate, ideally within 20–30 km
- [ ] **`PRE`, `POST`** — the clearest Sentinel-2 overpasses just before and just after
- [ ] **`FIRE_START`, `FIRE_END`** — burning period, end date *exclusive*
- [ ] **EMS reference** — activation code, vector file, layer (or leave empty)
- [ ] **`AGB_YEAR`** — the last ESA CCI biomass year before the fire
- [ ] **Fuel classes and emission factors** — Block 1 `CODES` and Block 5A `EF` if your forest differs
- [ ] **`LAST_YEAR`** — the most recent complete summer for the recovery series

## 1. Study box and control forest

`AOI_BBOX` should contain the whole scar with a margin, and little else — Earth Engine cost and time scale with area.

`CTRL_BBOX` is what makes the recovery analysis honest: if the control forest's NBR drifts, the "recovery" you measure is climate or sensor, not forest regrowth. Pick it so that:

- it has **the same dominant CORINE forest class** as the burned stand,
- it sits at a **similar elevation and aspect**,
- it had **no fire** in the whole analysis period — check on [FIRMS](https://firms.modaps.eosdis.nasa.gov/map/) for active-fire detections.

## 2. Dates

- **`PRE` / `POST`** — Block 3 composites every Sentinel-2 scene within ±3 days with cloud cover below 40 %. Browse candidate dates in the [Copernicus Browser](https://browser.dataspace.copernicus.eu/) and choose clear ones as close to the fire as possible. `POST` should be after the fire is out.
- **`FIRE_START` / `FIRE_END`** — used by the MODIS energy block and the Sentinel-5P window. Earth Engine date filters exclude the end date, so use the day *after* the fire ended.
- **`MOISTURE_WINDOW`** — month–day range in the weeks before ignition, compared with the same range in the four previous years.

## 3. Reference perimeter from Copernicus EMS

1. Open the [Copernicus EMS rapid-mapping portal](https://rapidmapping.emergency.copernicus.eu/) and search your country and date for a *Wildfire* activation (codes look like `EMSR449`).
2. In the activation, open the most recent **delineation** (`DEL`) or **monitoring** (`MONIT`) product for your area of interest.
3. Copy the name of the **vector package** — it ends in `_vector.zip` — into `EMS_ZIP`, and the activation code into `EMS_ACTIVATION`.
4. `EMS_LAYER` is the burnt-area polygon layer inside the zip; it contains `observedEventA`. For a monitoring product it is prefixed, e.g. `MONIT03_observedEventA`.

**No activation for your fire?** Leave `EMS_ZIP = ''`. The AOI box becomes the reference geometry; burned area then comes entirely from the Sentinel-2 dNBR threshold, so check the severity map visually in Block 8.

## 4. Fuel classes and emission factors

Block 1 keeps these CORINE classes as fuel:

| CORINE | Class | Kept by default |
|:--:|---|:--:|
| 311 | Broad-leaved forest | ✓ |
| 312 | Coniferous forest | ✓ |
| 313 | Mixed forest | ✓ |
| 324 | Transitional woodland-shrub | ✓ |
| 323 | Sclerophyllous vegetation (maquis, garrigue) | add for coastal Mediterranean fires |
| 321 / 322 | Natural grassland / moors and heathland | add for grass or heath fires |

**Every class in `CODES` needs a row in the Block 5A `EF` table** — emission factors for CO₂, CO, CH₄ and PM₂.₅ in g kg⁻¹, and a burn efficiency `BE`. The defaults are temperate/Mediterranean values from Laneve & Pampanoni (2021, after Prichard et al. 2020 and Li et al. 2017). For other biomes, the standard compilations are Andreae (2019) and Akagi et al. (2011).

## 5. Thresholds

- **`BURN_MIN`** (default 0.16) — the dNBR above which a pixel counts as burned. Calibrate it: run Block 3, compare the burned mask with the EMS perimeter in the Block 8 map, and adjust in steps of 0.02.
- **`SEV_BREAKS`** (0.27 / 0.44 / 0.66) — Key & Benson (2006) class edges. Change only with a field-based reason.
- **Block 5B presets** — combustion completeness by severity. Keep the *mean-preserving spread* unless you have field data.

## 6. What to expect

| Fire size | Sentinel-2 (20 m) | MODIS (1 km) | Sentinel-5P (5.5 × 7 km) |
|---|---|---|---|
| < 500 ha | thousands of pixels — reliable | a handful of pixels — use as upper bound | not detectable |
| 500 – 5,000 ha | reliable | FRE meaningful | usually below the floor — Block 6B tells you by how much |
| > 10,000 ha | reliable | reliable | may cross the floor on the peak days |

## 7. Sanity checks before you trust the numbers

1. **Block 1** — fuel area is smaller than the perimeter, and the class mix looks like the forest you know.
2. **Block 3** — the severity map sits inside the EMS outline in the Block 8 map.
3. **Block 5A** — implied emission factors (gas ÷ biomass) stay inside the range of the EF table.
4. **Block 7A** — the control forest is flat: drift within a few percent, σ around 0.01 NBR.
5. **Register** — every number you quote appears in `session1_results.csv`.
