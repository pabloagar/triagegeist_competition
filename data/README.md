# Data acquisition

No data is committed to this repository. There are two ways to obtain it.

## Option A — Kaggle (recommended, exact reproduction)

Attach the public companion dataset **`nhamcs-ed-2015-2022-raw`** to the notebook on Kaggle. It mirrors the raw CDC files used for the submission so the notebook runs unchanged.

## Option B — Run locally from the CDC source

1. Download the **NHAMCS Emergency Department Public Use Files** for survey years **2015, 2016, 2017, 2018, 2019, and 2022** from the CDC/NCHS NHAMCS data page (public domain, CC0).
   `<CDC_NHAMCS_DATA_URL — verify the exact page and file names before scripting>`
2. Place the raw fixed-width files here, matching the exact case CDC ships each year in
   (the notebook's parser depends on this case):
   ```
   data/raw/
     2015/ED2015/ED2015
     2016/ed2016/ed2016
     2017/ED2017/ED2017
     2018/ED2018/ED2018
     2019/ED2019/ed2019
     2022/ed2022/ed2022
   ```
3. Point the notebook's data path at `data/raw/` if it isn't already.

> The notebook parses the raw fixed-width layout directly. Do not commit anything under `data/raw/` — it is git-ignored on purpose.

## Provenance

NHAMCS ED Public Use Files are produced by CDC/NCHS and are US-government public domain (CC0). 2020–2021 are excluded from this project (COVID-era ED utilization and triage practice changed enough to confound temporal modeling). The competition's synthetic dataset is **not** stored or redistributed here.
