# Satellite quantification of the methane budget and post-fire recovery of a Mediterranean forest fire

**Vinoth Emberumal** — MSc Space & Astronautical Engineering, Sapienza University of Rome
*Technical note accompanying the OpenCarbon · Forest Fire-Debt EO Toolkit · October 2026* · [PDF version](Arischia_Technical_Note.pdf)

> **Abstract.** The Arischia wildfire (L'Aquila, Italy, 30 July – 13 August 2020) is re-analysed with an open, reproducible Google Earth Engine pipeline built from free optical, thermal and atmospheric satellite data. Intersecting a thematic land-cover map with a 10 m tree-cover map removes 28.3 % of the Copernicus EMS perimeter as non-combustible and leaves 458.0 ha of fuel, of which 309.0 ha burned. A Seiler–Crutzen budget per fuel class gives 14,982 t of biomass consumed and **65.07 t of methane** (16–84 % Monte-Carlo band 42.6–87.6 t), with 24,244 t CO₂, 1,416 t CO and 167 t PM₂.₅. The carbon mass balance closes at 104.6 %. Against the published analysis of the same fire the methane estimate differs by +9.2 %, obtained without calibration to it, and the optical biomass lies within 11.8 % of that study's independent thermal estimate. MODIS fire radiative energy integrated from daily maxima is an upper bound six to seven times larger than the optical biomass; the ratio implies the fire burned at full intensity for only ~3.5 hours a day. A forward plume model, written down before any Sentinel-5P data were opened, predicts a 0.88 ppb column enhancement — 0.08 σ of TROPOMI's single-sounding precision — so this fire was 25 times below the 2σ detection floor across all 27 plume geometries tested. Eleven summers of Sentinel-2 resolve exponential canopy recovery with time constants of 8–11 years by severity class. The fire released 47–140 years of the stand's own soil methane uptake; because soil microbial recovery cannot be observed from orbit, the return of that sink is bracketed only as a modelled five-decade window.

## 1 · Why methane

Carbon dioxide dominates fire emission inventories because it dominates the emitted mass: 93.6 % at Arischia. Methane is 0.25 % of the mass and 0.66 % of the carbon — but one kilogram traps roughly eighty times more heat than a kilogram of CO₂ over twenty years. Weighted over that horizon, the fire's 65 t of methane carries **about 18 %** of its near-term warming. A CO₂-only inventory misses roughly a fifth of the impact.

A second asymmetry makes methane more than a footnote. Temperate forest soils are the main biological land sink for atmospheric methane: methanotrophic bacteria in the organic horizon oxidise 1.5–4.5 kg CH₄ ha⁻¹ yr⁻¹ (Dutaur & Verchot 2007). A fire therefore acts twice — it emits a pulse and it removes the mechanism that would have taken methane back out of the air. Over 309 ha, the lost uptake is 0.46–1.39 t CH₄ yr⁻¹, so the two-week pulse equals **47–140 years** of this forest's own removal capacity: a *methane debt*.

This note asks three questions. **(i)** How much methane was released, with what honest uncertainty? **(ii)** Could a satellite have seen it? **(iii)** When does the forest — and its methane sink — recover?

## 2 · Study area and reference

The fire burned in the central Apennines above Arischia (42.4° N, 13.3° E), on slopes averaging 23°, in a stand dominated by coniferous forest (60 % of the fuel area). Copernicus EMS activation **EMSR449** provides an operational burn delineation of 638.99 ha, used here only as reference geometry.

Laneve & Pampanoni (2021) analysed the same event with Sentinel-2, PlanetScope and thermal fire-radiative-power data. Their Sentinel-2 budget (59.60 t CH₄) and their polar-orbit thermal budget (16,751 t biomass) serve as **independent anchors**: they are compared against at the end, never used to tune any parameter.

## 3 · Data

| Domain | Sensor / product | Resolution | Role |
|---|---|---|---|
| Optical | Sentinel-2 MSI L2A | 10–20 m, 5-day | burn severity, moisture, eleven-year recovery |
| Thermal | MODIS Terra + Aqua, MOD14A1/MYD14A1 | 1 km, ~4 looks/day | fire radiative power and energy |
| Atmospheric chemistry | Sentinel-5P TROPOMI CH₄ and CO | ~5.5 × 7 km, daily | detectability test |
| Land cover | CORINE 2018, ESA WorldCover 2020 | 100 m, 10 m | fuel type ∩ fuel extent |
| Biomass | ESA CCI AGB v6 (2019) | 100 m | fuel load per class |
| Weather | ERA5-Land | ~9 km | pre-fire temperature and rainfall |
| Reference | Copernicus EMS EMSR449 | vector | perimeter |

All processing runs server-side in Google Earth Engine from a single Colab notebook; every number is written to one results register at the moment it is computed.

## 4 · Methods

Full formulas and constants are in [`03_METHODS.md`](../03_METHODS.md); the essentials:

- **Fuel.** CORINE forest classes (311, 312, 313, 324) intersected with WorldCover tree and shrub cover. No tuned parameter.
- **Severity.** `NBR = (B8 − B12)/(B8 + B12)`, `dNBR = NBR_pre − NBR_post`; burned where dNBR ≥ 0.16; four classes at 0.27, 0.44, 0.66 (Key & Benson 2006).
- **Budget.** `BB = Σ A·AGB·BE` per fuel class; `E_x = Σ BB·EF_x` with the class-specific emission factors of Laneve & Pampanoni (2021). Uncertainty by 200,000 Monte-Carlo draws (area ±5 %, AGB ± class σ, BE ±30 %, EF ±40 %). A severity-dependent combustion completeness (β = 0.20 / 0.35 / 0.50 / 0.65) is run as a sensitivity test.
- **Thermal energy.** `FRE = ∫FRP dt` from daily MODIS maxima, `BB = 0.368·FRE` (Wooster et al. 2005). Because daily maxima are treated as sustained, this is an upper bound; its ratio to the optical biomass is interpreted as a **diurnal duty cycle**.
- **Detectability.** Integrated-mass-enhancement balance run forwards: `τ = L/U`, `IME = Q·τ`, `ΔXCH₄ = (IME/M)·N_A/(A·N_air)`. The prediction was saved with a timestamp before TROPOMI data were queried, then swept over 27 plume geometries.
- **Recovery.** One peak-season NBR composite per summer, 2016–2026, per severity class and for an unburnt control forest; `ln(NBR_base − NBR)` fitted linearly in time (exponential recovery). Full recovery is declared when the deficit falls below the control forest's interannual σ; the methane-sink window applies an assumed soil-microbial lag of 1.3–2.0× to the canopy timeline.

<p align="center"><img src="../../assets/results/pipeline_flow.png" width="100%" alt="Figure 1 · Sensor → method → measured result"><br><sub>Figure 1 · Sensor → method → measured result</sub></p>

## 5 · Results

### 5.1 Areas and severity

| Quantity | ha | % of perimeter |
|---|--:|--:|
| EMS perimeter | 638.99 | 100.0 |
| removed as non-fuel | 181.0 | 28.3 |
| **burnable forest** | **458.0** | 71.7 |
| **burned forest (dNBR ≥ 0.16)** | **309.0** | 48.4 |
| — Low / Moderate / High / Very high | 95.7 / 79.1 / 52.3 / 81.9 | |

### 5.2 Emission budget

| Species | Mass [t] | t ha⁻¹ | % of mass | Implied EF [g kg⁻¹] |
|---|--:|--:|--:|--:|
| CO₂ | 24,244 | 78.5 | 93.6 | 1,618 |
| CO | 1,416 | 4.58 | 5.47 | 94.5 |
| PM₂.₅ | 167 | 0.54 | 0.65 | 11.2 |
| **CH₄** | **65.07** (42.6–87.6) | **0.211** | **0.25** | **4.34** |
| Biomass consumed | 14,982 | 48.5 | — | — |

The asymmetric band is the signature of a product of uncertain terms. The emission factor dominates the variance; burned area contributes least — so better imagery would not narrow the result, but a field-measured emission factor would. Distributing combustion completeness by severity instead of by fuel class changes methane by only −3.0 % (63.12 t).

<p align="center"><img src="../../assets/results/monte_carlo_ch4.png" width="100%" alt="Figure 2 · Monte-Carlo distribution of methane released"><br><sub>Figure 2 · Monte-Carlo distribution of methane released</sub></p>

### 5.3 Consistency checks

| Check | This work | Reference / closure | Agreement |
|---|---|---|---|
| Methane vs Laneve & Pampanoni (2021), Sentinel-2 route | 65.07 t | 59.60 t | +9.2 %, inside the 16–84 % band |
| Optical biomass vs their thermal (FRP, polar-orbit) route | 14,982 t | 16,751 t | 11.8 % |
| Carbon emitted vs carbon available (BB × 0.47) | 7,368 tC | 7,042 tC | 104.6 % |

The comparison also exposes an internal inconsistency in the reference: its Sentinel-2 budget implies an emission factor for CO₂ of 1,390 g kg⁻¹ (23,329 t ÷ 16,783 t), below every class value in its own emission-factor table (1,592–1,639 g kg⁻¹). The present budget implies 1,618 g kg⁻¹, inside that range — a ratio of 1.164 that follows directly from the table.

<p align="center"><img src="../../assets/results/cross_method_consistency.png" width="100%" alt="Figure 3 · Optical vs reference thermal biomass, methane vs reference, carbon balance"><br><sub>Figure 3 · Optical vs reference thermal biomass, methane vs reference, carbon balance</sub></p>

### 5.4 How long the fire actually burned

MODIS saw the fire on five days, peaking at 4,115 MW. Integrating the daily maxima as if sustained gives 279.8 TJ and an upper-bound biomass of 102,976 t — 6.9 times the optical estimate. Read the other way round, the optical budget is 14.5 % of that bound: the fire released its energy as if it had burned at peak intensity for about **3.5 hours a day**, consistent with a strongly diurnal Mediterranean fire.

### 5.5 Was the methane visible from orbit?

The acute-phase emission rate is 6.5 t CH₄ day⁻¹ (60 % of the methane in the first six days). At the nominal geometry (L = 20 km, U = 3 m s⁻¹, A = 100 km²) this gives **ΔXCH₄ = 0.88 ppb — 0.08 σ** of TROPOMI's ~11 ppb single-sounding precision. Detection at 2σ would need 165 t day⁻¹; across the 27-geometry sweep the floor ranges from 27 to 1,098 t day⁻¹ and the best achievable significance is 0.47 σ. The fire was **25 times below the floor**. In the event, TROPOMI returned no valid methane retrieval over the site during the fire window (33 valid days for CO), so the conclusion rests on the forward model — which is exactly why the model was written down first.

<p align="center"><img src="../../assets/results/nb_detection_floor_sweep.png" width="100%" alt="Figure 5 · 2σ TROPOMI detection floor across 27 plume geometries"><br><sub>Figure 5 · 2σ TROPOMI detection floor across 27 plume geometries</sub></p>

### 5.6 Recovery

| Severity | Recovered by 2026 | k [yr] | 90 % canopy | Full canopy | CH₄ sink (modelled) |
|---|--:|--:|--:|--:|--:|
| Low | 66.7 % | 10.9 | 2039 | 2057 | 2045 – 2094 |
| Moderate | 59.4 % | 8.9 | 2038 | 2053 | 2043 – 2085 |
| High | 51.9 % | 8.3 | 2038 | 2052 | 2044 – 2083 |
| Very high | 35.8 % | 8.3 | 2041 | 2055 | 2047 – 2089 |

The unburnt control drifted by −1.4 % over the decade with an interannual σ of 0.0108 NBR, so the recovery signal is not climate. Severity sets how deep the hole is, not how fast the forest climbs out: the time constants are similar, the starting depths are not. The pre-fire moisture test (NDMI −0.66 σ, 180.6 mm of rain in June–July) indicates the stand was not anomalously dry — an ignition-limited rather than a drought-primed fire.

<p align="center"><img src="../../assets/results/nb_recovery_trajectory.png" width="100%" alt="Figure 4 · Spectral recovery by severity class and the modelled methane-sink window"><br><sub>Figure 4 · Spectral recovery by severity class and the modelled methane-sink window</sub></p>

## 6 · Limits

1. Emission factors, not imagery, limit the methane estimate (±40 %).
2. The MODIS energy integral is an upper bound here and is not used as a second biomass estimate.
3. NBR is a spectral proxy for canopy, not a measure of structure or carbon.
4. The methane-sink window is modelled; methanotroph recovery is invisible to every satellite band, and closing that range needs soil flux chambers or eddy-covariance measurements along a severity gradient.

## 7 · Conclusions

1. **Quantification** — 65.07 t CH₄ (42.6–87.6 t) from 309 ha, within 9.2 % of an independent study and with a closing carbon balance.
2. **Detectability** — a fire of this size sits 25× below what a daily global methane satellite can see, predicted before looking.
3. **Recovery** — the canopy returns within two to four decades; the methane sink is bracketed only as a modelled 2043–2094 window, and 47–140 years of uptake are owed back.

## 8 · Next steps

- **Structure, not only colour** — GEDI lidar canopy height matched to severity classes, and Sentinel-1 SLC interferometric coherence processed in ESA SNAP.
- **Data fusion** — optical, radar and lidar in one recovery model; VIIRS 375 m fire radiative power for a sharper thermal route.
- **Prediction** — Gaussian-process regression in log-deficit space for recovery with credible intervals.
- **Sensors that could see it** — EnMAP, PRISMA and GHGSat now; Copernicus CO2M next.
- **More fires** — the same notebook run on larger Mediterranean events to map where the detection floor is crossed.

## Acknowledgements

This work was developed for the *Earth Observation* course of **Prof. Ferdinando Nunziata**, Sapienza University of Rome. It builds on, and is compared against, the analysis of **G. Laneve and V. Pampanoni**. Data from Copernicus, ESA, NASA and ECMWF, processed in Google Earth Engine.

## References

- Dutaur, L. & Verchot, L. V. (2007). A global inventory of the soil CH₄ sink. *Global Biogeochemical Cycles* 21, GB4013.
- Key, C. H. & Benson, N. C. (2006). Landscape assessment. *FIREMON*, USDA RMRS-GTR-164-CD.
- Laneve, G. & Pampanoni, V. (2021). Review of satellite based support to forest fire environmental impact assessment: the example of Arischia (Italy) forest fire. *Geografia, Riscos e Protecção Civil* 2, 163–178. [doi:10.34037/978-989-9053-06-9_1.2_12](https://doi.org/10.34037/978-989-9053-06-9_1.2_12)
- Li, F. et al. (2017) and Prichard, S. J. et al. (2020) — emission-factor sources, via Laneve & Pampanoni (2021).
- Parks, S. A., Dillon, G. K. & Miller, C. (2014). A new metric for quantifying burn severity: the relativized burn ratio. *Remote Sensing* 6, 1827–1844.
- Seiler, W. & Crutzen, P. J. (1980). Estimates of gross and net fluxes of carbon between the biosphere and the atmosphere from biomass burning. *Climatic Change* 2, 207–247.
- Varon, D. J. et al. (2018). Quantifying methane point sources from fine-scale satellite observations of atmospheric methane plumes. *Atmospheric Measurement Techniques* 11, 5673–5686.
- Wooster, M. J. et al. (2005). Retrieval of biomass combustion rates and totals from fire radiative power observations. *Journal of Geophysical Research* 110, D24311.
