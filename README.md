# 🌧️ Rainfall Climatology of India — Interactive 3D Explorer

**An interactive, browser-based view of 124 years of Indian rainfall (1901–2024), built from IMD 0.25° gridded daily data.**

[![Live site](https://img.shields.io/badge/Live%20site-GitHub%20Pages-1f4e79?style=for-the-badge&logo=github)](https://USERNAME.github.io/REPO/)
![Data](https://img.shields.io/badge/Data-IMD%200.25%C2%B0%20gridded-2b7bb9?style=flat-square)
![Period](https://img.shields.io/badge/Period-1901%E2%80%932024-5aae61?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-WebGL-3F4F75?style=flat-square&logo=plotly&logoColor=white)

### 👉 **[Open the interactive site](https://USERNAME.github.io/REPO/)**

<p align="center">
  <img src="assets/preview.png" alt="3D surface of mean daily rainfall over India" width="90%">
</p>

---

## Overview

This site turns the India Meteorological Department's (IMD) high-resolution gridded rainfall dataset into an explorable 3D landscape. The height and colour of the surface both show mean daily rainfall. You can switch between seasons, rotate the view from the Bay of Bengal or the Arabian Sea, and hover over any grid point to read its value.

Alongside the 3D view, the page includes a set of evaluation panels on the seasonal cycle, long-term trends, interannual variability, and monsoon excess/deficit years. These give a quick quantitative picture of how rainfall is distributed across India and how it has changed.

Everything runs in the browser. There is no server and no installation, and it works on desktop and mobile.

---

## ✨ What's on the site

### 1. Interactive 3D rainfall surface
| Control | What it does |
|---|---|
| **Season selector** | Annual · SW Monsoon (JJAS) · Post-monsoon (OND) · Winter (JF) · Pre-monsoon (MAM), following IMD's seasonal convention |
| **Camera presets** | SW oblique · from the Bay of Bengal · from the Arabian Sea · from the Himalaya · top-down |
| **Relief ×0.5 – ×2** | Vertical exaggeration |
| **Base map on/off** | Projected 2D rainfall map with coastlines and the India outline, drawn beneath the surface |
| **Hover** | Latitude, longitude and mean rainfall (mm day⁻¹) |
| **📷 toolbar icon** | Saves a PNG at 3× resolution for slides or papers |

### 2. Key statistics
- All-India mean annual rainfall
- Share of annual rainfall falling in the SW monsoon (JJAS)
- Interannual coefficient of variation
- Linear trends of annual and JJAS rainfall, in mm per decade, with p-values
- Number of excess (≥ +10 %) and deficient (≤ −10 %) monsoon years
- Wettest and driest grid cells

### 3. Evaluation panels
| Panel | Description |
|---|---|
| **Seasonal cycle** | Area-weighted all-India monthly rainfall, coloured by season |
| **Rainfall through time** | Annual all-India totals with an 11-year running mean and trend line, plus JJAS % departure bars highlighting excess and deficient years |
| **Grid-point trends** | Maps of annual and JJAS trends (mm/decade), with dots where p < 0.05 |
| **Interannual variability** | Coefficient-of-variation maps for JJAS and annual totals |

---

## 📊 Data

| | |
|---|---|
| **Dataset** | IMD gridded daily rainfall, 0.25° × 0.25° |
| **Period** | 1 January 1901 – 31 December 2024 |
| **Coverage** | Indian mainland (land points only) |
| **Units** | mm day⁻¹ (daily), aggregated to monthly, seasonal and annual totals |
| **Source** | [IMD Pune — Climate Data Service Portal](https://imdpune.gov.in/) |

> **Note:** The raw NetCDF data is **not** redistributed in this repository. Please obtain it directly from IMD, subject to their terms of use.

---

## 🔬 Methods

1. **Aggregation.** Daily fields are summed to monthly totals in a single pass using dask, and cached. IMD fill values (−999) are treated as missing.
2. **Climatology.** Monthly totals are divided by the number of days in each month to get mm day⁻¹. Seasonal means are **day-weighted** averages of the monthly climatology.
3. **3D surface (display only).** Missing cells along the coast are filled from the nearest valid neighbour, the field is Gaussian-smoothed (σ = 1 grid cell), and no-data cells are then masked again. Values in the statistics and evaluation panels are computed from the **unsmoothed** data.
4. **All-India series.** Means are weighted by cos(latitude) over valid land cells.
5. **Departures.** Seasonal totals are expressed as a percentage departure from the **1901–2024 mean**. Years at or above +10 % are labelled excess and years at or below −10 % are labelled deficient.
6. **Trends.** Ordinary least-squares regression on yearly totals, with two-sided t-tests for significance.

### ⚠️ Limitations
- Departures are measured from the period mean, **not** from IMD's official Long Period Average (LPA), so excess/deficient counts may differ slightly from IMD bulletins.
- The trend tests do not account for serial autocorrelation or field significance. Treat the stippling as indicative rather than conclusive.
- Changes in the rain-gauge network over 124 years affect the homogeneity of the dataset, especially in the early decades.
- The default boundary comes from **Natural Earth** and is for illustration only. It is **not** the official Survey of India boundary. Use `--boundary` to supply an official shapefile for publication.

---

## 🛠️ Reproduce or update the site

### Requirements
```bash
pip install xarray netCDF4 dask scipy pandas plotly cartopy shapely matplotlib
```

### Build
```bash
python build_rainfall_site.py "/path/to/IMD_Rainfall_1901-2024_merged.nc" \
    --out-dir rainfall-site \
    --author "Your Name, Affiliation"
```
Or from Jupyter:
```python
from build_rainfall_site import build_site
build_site("/path/to/IMD_Rainfall_1901-2024_merged.nc", out_dir="rainfall-site")
```

The first run aggregates the full daily record, which takes a few minutes, and writes a cache (`*_monthly_totals.nc`) next to the input file. Later runs finish in seconds.

### Options
| Flag | Default | Description |
|---|---|---|
| `--out-dir` | `rainfall-site` | Output folder |
| `--var` / `--lat` / `--lon` | `rainfall` / `latitude` / `longitude` | Variable and dimension names in the NetCDF |
| `--sigma` | `1.0` | Gaussian smoothing of the 3D surface, in grid cells (0 = off) |
| `--upsample` | `1` | Mesh refinement. Use `2` for a smoother surface; the page becomes about 4× larger |
| `--offset` | `0.35` | Depth of the optional base map below the surface, as a fraction of the maximum |
| `--boundary` | *Natural Earth* | Path to an official India boundary shapefile |
| `--author` | — | Text shown in the page header |
| `--cache` | *auto* | Custom path for the monthly-totals cache |

---

## 🚀 Deploy on GitHub Pages

1. Push `index.html`, `.nojekyll` and this `README.md` to the root of a repository.
2. Go to **Settings → Pages → Build and deployment**.
3. Set **Source** to *Deploy from a branch*, choose `main` and `/ (root)`, then click **Save**.
4. After a minute or two the site is live at `https://USERNAME.github.io/REPO/`.

To host it inside an existing Pages site, put the files in a subfolder such as `/rainfall/`. It will then be served at `https://USERNAME.github.io/REPO/rainfall/`.

---

## 📁 Repository structure
```
.
├── index.html               # The website (self-contained; Plotly loaded from CDN)
├── .nojekyll                # Serve files as-is on GitHub Pages
├── build_rainfall_site.py   # Generates index.html from IMD NetCDF data
├── assets/
│   └── preview.png          # Screenshot used in this README
└── README.md
```

---

## 📚 Citation

If you use this site or its figures, please cite the underlying dataset:

> Pai, D. S., Sridhar, L., Rajeevan, M., Sreejith, O. P., Satbhai, N. S., & Mukhopadhyay, B. (2014). Development of a new high spatial resolution (0.25° × 0.25°) long period (1901–2010) daily gridded rainfall data set over India and its comparison with existing data sets over the region. *MAUSAM*, 65(1), 1–18.

And, optionally, this repository:
```bibtex
@misc{rainfall_india_3d,
  author       = {Devashish},
  title        = {Rainfall Climatology of India: Interactive 3D Explorer (IMD 1901--2024)},
  year         = {2026},
  howpublished = {\url{https://github.com/USERNAME/REPO}}
}
```

---

## 🙏 Acknowledgements

- **India Meteorological Department (IMD)**, Pune, for the gridded rainfall dataset
- [Plotly](https://plotly.com/python/), [xarray](https://xarray.dev/), [Cartopy](https://scitools.org.uk/cartopy/) and [SciPy](https://scipy.org/)
- [Natural Earth](https://www.naturalearthdata.com/) for the coastline and boundary data

---

## 👤 Author

**Devashish**, Research Scholar, MEGHA Lab, Indian Institute of Technology Hyderabad
Tropical meteorology · Rapid intensification of Bay of Bengal cyclones
Lifetime Member, Indian Meteorological Society

*Questions, suggestions or bugs? Please [open an issue](https://github.com/USERNAME/REPO/issues).*

---

## 📄 License

The code is released under the [MIT License](LICENSE). IMD data remains subject to IMD's own terms of use and is not covered by this license.
