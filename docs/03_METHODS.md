# 03 · Methods, formulas and assumptions

A compact reference for every calculation in the notebook, in block order. Numbers in *italics* are the Arischia values from the saved run.

## Block 1 · Fuel map

| Step | Formula / rule | Notes |
|---|---|---|
| Fuel type | CORINE 2018 ∈ {311, 312, 313, 324} | 100 m, photo-interpreted |
| Fuel extent | ESA WorldCover 2020 ∈ {10 tree, 20 shrub} | 10 m, automatic classification |
| Fuel mask | `FUEL = type ∧ extent` | no tuned parameter — *removes 28.3 % of the EMS perimeter* |
| Area | Σ pixel area inside the EMS geometry, 20 m | `ee.Image.pixelArea()` |

## Block 2 · Pre-fire state

| Quantity | Formula | Source |
|---|---|---|
| Biomass density | mean and σ of CCI AGB per fuel class | ESA CCI Biomass v6, 100 m — *117.3 t ha⁻¹* |
| Moisture index | `NDMI = (B8 − B11)/(B8 + B11)` | Sentinel-2 L2A, SCL-masked median |
| Dryness anomaly | `(NDMI_fire − μ_baseline)/σ_baseline`, same calendar window | *−0.66 σ: not anomalously dry* |
| Weather | mean 2 m temperature, total precipitation, 1 Jun → ignition | ERA5-Land — *18.1 °C, 180.6 mm* |

## Block 3 · Burn severity

| Quantity | Formula |
|---|---|
| Normalised burn ratio | `NBR = (B8 − B12)/(B8 + B12)` |
| Difference | `dNBR = NBR_pre − NBR_post` |
| Relativised | `RBR = dNBR/(NBR_pre + 1.001)` (Parks et al. 2014) |
| Burned | `dNBR ≥ 0.16` inside the fuel mask |
| Classes | Low 0.16–0.27 · Moderate 0.27–0.44 · High 0.44–0.66 · Very high ≥ 0.66 (Key & Benson 2006) |

Compositing: median of Sentinel-2 L2A scenes within ±3 days of `PRE` / `POST`, cloud < 40 %, SCL classes 3, 8, 9, 10, 11 masked. *309.0 ha burned.*

## Block 4 · Fire radiative energy

| Quantity | Formula | Notes |
|---|---|---|
| FRP | Σ `MaxFRP × 0.1` over fire pixels (FireMask ≥ 7) within the perimeter + 2 km | MOD14A1 (Terra) + MYD14A1 (Aqua), 1 km |
| FRE | trapezoid `Σ ½(FRPᵢ + FRPᵢ₊₁)·Δt`, gaps > 30 h not bridged | daily *maximum* treated as sustained → **upper bound** |
| Biomass | `BB = Cr · FRE`, `Cr = 0.368 kg MJ⁻¹` (Wooster et al. 2005) | *279.8 TJ → 102,976 t* |

## Block 5A · Emission budget

**Seiler & Crutzen (1980)**, per fuel class *i*:

```
BB   = Σᵢ Aᵢ · AGBᵢ · BEᵢ
Eₓ   = Σᵢ BBᵢ · EFₓ,ᵢ
```

| Symbol | Meaning | Value / source |
|---|---|---|
| Aᵢ | burned area of class *i* | Block 3 ∩ CORINE |
| AGBᵢ | biomass density | Block 2 |
| BEᵢ | burn efficiency | 0.40 forest, 0.50 woodland-shrub |
| EFₓ,ᵢ | emission factor, g kg⁻¹ | Laneve & Pampanoni (2021), after Prichard et al. (2020) and Li et al. (2017) |

**Derived quantities**

| Quantity | Formula | Arischia |
|---|---|---|
| Diurnal duty cycle | `BB_optical / BB_FRE-upper` | *14.5 % ≈ 3.5 full-power h/day* |
| CO₂-equivalent | `CH₄ × GWP₁₀₀`, GWP₁₀₀ = 27.2 (IPCC AR6) | *1,770 t = 7.3 % of CO₂* |
| Methane debt | `CH₄ / (A_burned × soil sink)`, sink 1.5–4.5 kg ha⁻¹ yr⁻¹ | *47–140 years* |

**Monte-Carlo uncertainty** — 200,000 draws, seed 42, independent normal perturbations:

| Term | Spread |
|---|---|
| Area | ±5 % |
| AGB | ± class standard deviation |
| BE | ±30 % |
| EF | ±40 % |

Reported as the 16th–84th percentile (the ±1σ-equivalent for a skewed distribution): *65.07 t CH₄, 42.6–87.6 t*. The product of several uncertain terms is right-skewed, which is why the band is asymmetric.

**Carbon closure** (Block 8B): `C_emitted / C_available`, with `C_emitted = CO₂·12/44 + CO·12/28 + CH₄·12/16 + 0.60·PM₂.₅` and `C_available = 0.47·BB`. The 0.60 carbon fraction of PM₂.₅ is an assumption; 0.47 is the IPCC default. *104.6 %.*

## Block 5B · β by severity

`BB = Σ(class × severity) A · AGB · β(severity)` with the mean-preserving spread β = 0.20 / 0.35 / 0.50 / 0.65 (Low → Very high). Effective β is kept at 0.408 so the comparison isolates the *distribution* of burning. *63.12 t CH₄, −3.0 % vs uniform — the budget is robust to it.*

## Block 6A/6B · Methane detectability

Integrated-mass-enhancement (IME) mass balance, run **forwards** (Varon et al. 2018):

```
τ      = L / U                                   residence time of gas in view
IME    = Q · τ                                   instantaneous plume mass
ΔXCH₄  = (IME / M_CH₄) · N_A / (A · N_air) · 10⁹  column enhancement [ppb]
N_air  = P₀ / (g₀ · M_air) · N_A                 molecules of air per m²
```

| Parameter | Value |
|---|---|
| Q (acute phase) | 60 % of CH₄ over the first 6 days — *6.5 t day⁻¹* |
| L, U, A (nominal) | 20 km, 3 m s⁻¹, 100 km² |
| σ_TROPOMI | 0.6 % × 1850 ppb ≈ 11.1 ppb (single sounding) |
| detection | 2σ |
| sweep (6B) | L ∈ {10, 20, 40} km × U ∈ {2, 3, 5} m s⁻¹ × A ∈ {50, 100, 200} km² |

The prediction is written to `docs/prediction.txt` **before** TROPOMI data are loaded. *Predicted 0.88 ppb = 0.08 σ; 2σ floor 165 t day⁻¹ (27–1,098 across the sweep); the fire is 25× below it. No valid CH₄ retrieval existed over the site in the window (33 for CO), so the conclusion rests on the forward model.*

## Block 7A/7B · Recovery

| Step | Formula |
|---|---|
| Annual composite | median NBR, 15 Jun – 15 Sep, cloud < 30 %, per severity class and control |
| Baseline | mean of the pre-fire years with a valid composite (*2016, 2018, 2019*) |
| Recovery | `NBR(year) / NBR_baseline × 100` |
| Model | `NBR(t) = NBR_base − A·e^(−t/k)` ⇔ `ln(NBR_base − NBR) = ln A − t/k` (linear fit) |
| 90 % canopy | year when the deficit equals 10 % of baseline |
| Full canopy | year when the deficit falls below the control forest's σ (*0.0108 NBR*) |
| Methane sink | canopy years × soil-microbial lag 1.3–2.0 — **modelled**, tagged `[MODELLED]` |

The fire year is a mixed composite and is excluded from baseline and fit.

## Honest limits

1. **Emission factors dominate the uncertainty** (±40 %). Better imagery cannot narrow the methane estimate; field-measured emission factors could.
2. **MODIS FRE is an upper bound** here, because daily `MaxFRP` is a peak value. It is used to derive the duty cycle, not as a second biomass estimate. The independent thermal comparison in the README uses the published polar-orbit FRP budget of Laneve & Pampanoni (2021).
3. **Sentinel-5P cannot see fires this size.** The toolkit quantifies by how much, rather than claiming a detection.
4. **NBR is a spectral proxy**, not canopy height or biomass. Structural recovery would need lidar (GEDI) or radar coherence.
5. **The methane-sink recovery is modelled**, not measured: methanotrophs live in the soil organic horizon, which no satellite band reaches.
