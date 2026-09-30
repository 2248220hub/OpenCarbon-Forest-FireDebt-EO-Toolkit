# 01 · Setup — from zero to a first run

You need a browser and a Google account. Nothing is installed on your computer.

## 1. Get access to Google Earth Engine

Earth Engine is free for research, education and non-profit work, but every user needs a **Google Cloud project** registered for it.

1. Go to **[code.earthengine.google.com/register](https://code.earthengine.google.com/register)** and sign in.
2. Choose **Use with a noncommercial Cloud project** (research, education, non-profit, journalism) and answer the short eligibility questions.
3. Create a new Cloud project (or pick an existing one). Give it any name.
4. Confirm the **Earth Engine API** is enabled for that project — the registration flow normally does it; you can check under *APIs & Services* in [console.cloud.google.com](https://console.cloud.google.com/).
5. Copy the **project ID** — the lowercase identifier such as `my-fire-study-123456`, not the display name.

> Earth Engine's registration pages change from time to time. If a step looks different, the official guide is [developers.google.com/earth-engine/guides/access](https://developers.google.com/earth-engine/guides/access).

## 2. Open the notebook in Colab

Click the badge on the repository front page, or in Colab choose *File → Open notebook → GitHub*, paste the repository URL and pick `notebooks/OpenCarbon_FireDebt_Toolkit.ipynb`.

Save your own copy (*File → Save a copy in Drive*) so your edits persist.

## 3. Configure Block 0

In the form on the right of Block 0:

| Field | Put |
|---|---|
| `GEE_PROJECT` | your project ID from step 1 |
| `DRIVE_DIR` | the folder name you want in your Google Drive |

Leave everything else as it is for a first run — it reproduces the Arischia showcase.

## 4. Run

*Runtime → Run all.* Two pop-ups appear:

1. **Earth Engine** — sign in with the account you registered and allow access.
2. **Google Drive** — allow Colab to mount your Drive; results are written to `MyDrive/<DRIVE_DIR>/`.

A full run takes about 10–20 minutes, almost all of it waiting for Earth Engine. Your printed register lines should match the **Arischia result** line in each block's card.

## 5. Where everything lands

```
MyDrive/<DRIVE_DIR>/
├── data/
│   ├── reference/     Copernicus EMS perimeter (downloaded once)
│   └── outputs/       CSV tables: register, β-by-severity, geometry sweep, recovery
├── figures/           PNG figures + dashboard.html, dashboard_novelty.html, recovery_dashboard.html
└── docs/
    └── prediction.txt pre-registered methane prediction, time-stamped
```

## Running locally instead of Colab (optional)

```bash
git clone https://github.com/2248220hub/OpenCarbon-Forest-FireDebt-EO-Toolkit.git
cd OpenCarbon-Forest-FireDebt-EO-Toolkit
conda env create -f environment.yml && conda activate opencarbon-firedebt   # or: pip install -r requirements.txt
earthengine authenticate
jupyter lab notebooks/OpenCarbon_FireDebt_Toolkit.ipynb
```

Outside Colab, replace the `drive.mount(...)` line in Block 0 with a local folder, e.g. `PROJ = './outputs'`.

## Troubleshooting

| Message | Meaning | Fix |
|---|---|---|
| `Please set GEE_PROJECT …` | the placeholder is still in the form | paste your project ID |
| `EEException: … not registered to use Earth Engine` | the Cloud project isn't registered | finish step 1, then re-run Block 0 |
| `Collection asset … not found` | an asset or geometry ID is wrong | check `AOI_BBOX`, `EMS_*`, `AGB_YEAR` |
| `Computation timed out` | the request is too big for an interactive call | shrink `AOI_BBOX` to the fire plus 1–2 km |
| `HTTP Error 404` in Block 1 | the EMS file name doesn't exist | copy the exact `…_vector.zip` name (see [02_ADAPT](02_ADAPT_YOUR_FOREST.md)), or set `EMS_ZIP = ''` |
| map layers blank in a saved HTML | Earth Engine tile links expire after a while | re-run Block 8 / 8B to regenerate the dashboard |
| sliders missing in Block 5B | widgets only run live | open the notebook in Colab, not on GitHub |
