# india-nightlights-district-1992-2025
A district-wise nighttime lights dataset for India (1992–present), with DMSP-OLS and VIIRS sensors calibrated into a consistent time series using a CNN-based model.
# India Nighttime Lights — District Level (1992–Present)

District-level nighttime lights raster data for India (1992–present), harmonizing **DMSP-OLS** and **VIIRS-DNB** imagery into a single consistent time series using a **CNN-based pixel-level calibration model**.

---

## Overview

This repository compiles, processes, and calibrates satellite-derived nighttime lights (NTL) data for India at the **district level**, spanning two generations of sensors:

- **DMSP-OLS**: 1992–2013 (~1 km resolution, 6-bit digital number, known for blooming/saturation and lack of inter-annual calibration)
- **VIIRS-DNB / VNL**: 2012–present (~500 m resolution, higher radiometric and spatial fidelity)

Because DMSP-OLS and VIIRS are not directly comparable (different sensors, resolutions, and radiometric scales), this repository applies a **convolutional neural network (CNN)** to learn a pixel-level mapping between the two, producing a harmonized long-term series suitable for time-series analysis across districts.

---

## Data Sources

| Sensor | Years | Resolution | Source | Notes |
|---|---|---|---|---|
| DMSP-OLS | 1992–2013 | ~1 km | NOAA NGDC | Annual stable lights composites |
| VIIRS-DNB (VNL) | 2012–present | ~500 m | NOAA / Earth Observation Group (Colorado School of Mines) | Monthly/annual composites |

> TODO: Add exact download links/version numbers for each source once finalized (e.g., VNL v2.1, DMSP v4).

**Overlap period (2012–2013):** Both sensors have data in this window. This overlap is used as the training/validation period for the CNN calibration model, since it provides paired DMSP–VIIRS observations for the same locations and time.

---

## District Boundaries

- **Boundary vintage**: 2011 Census of India district boundaries.
- District boundaries in India have changed over time (new districts have been carved out of older ones). Using a single fixed vintage (2011) ensures consistent zonal statistics across all years, but means:
  - Pre-2011 years are aggregated to 2011 district boundaries (not the boundaries that existed at that time).
  - Post-2011 newly created districts are merged back into their 2011 parent district for consistency.
- Source: [Add shapefile source/link here, e.g., Census of India / data.gov.in]

---

## Calibration Methodology (DMSP–VIIRS via CNN)

The CNN performs **pixel-level calibration**, learning to translate DMSP-OLS digital number values into VIIRS-equivalent radiance values (or vice versa, depending on chosen target sensor), correcting for:

- Blooming and saturation artifacts in DMSP-OLS
- Sensor-specific scale/resolution differences
- Inter-annual inconsistency in DMSP composites (no onboard calibration)

**Pipeline:**
1. Extract overlapping pixels from DMSP-OLS and VIIRS for 2012–2013.
2. Train CNN on paired patches (input: DMSP pixel + spatial neighborhood; target: VIIRS-equivalent value).
3. Apply trained model to all DMSP-OLS years (1992–2011) to produce calibrated, VIIRS-consistent rasters.
4. Validate calibrated output against held-out overlap data and/or known light-stable regions.

> TODO: Add specifics once finalized — CNN architecture, patch size, loss function, train/val split, validation metrics (e.g., RMSE, correlation with VIIRS).

---

## Repository Structure

```
india-nightlights-cnn-calibration/
│
├── raw/                      # Original, unmodified downloaded rasters
│   ├── dmsp/                 # DMSP-OLS annual composites (1992–2013)
│   └── viirs/                # VIIRS-DNB annual/monthly composites (2012–present)
│
├── calibrated/                # CNN-calibrated, harmonized rasters (1992–present)
│
├── processed/                  # District-clipped / zonal-extracted rasters
│
├── district_stats/             # Zonal statistics (CSV) — district x year nightlight values
│
├── boundaries/                 # District boundary shapefiles (2011 Census vintage)
│
├── model/                      # CNN calibration model
│   ├── architecture.py
│   ├── train.py
│   ├── weights/
│   └── validation/             # Calibration validation results, plots, metrics
│
├── scripts/                    # Data download, preprocessing, extraction scripts
│
└── README.md
```

---

## File Naming Convention

```
india_ntl_<sensor>_<year>.tif          # Raw raster, e.g. india_ntl_dmsp_2005.tif
india_ntl_calibrated_<year>.tif        # CNN-calibrated raster, e.g. india_ntl_calibrated_1998.tif
district_stats_<year>.csv              # Zonal statistics per district, e.g. district_stats_2005.csv
```

---

## How to Use

```bash
# Example: load a calibrated raster and extract district-level mean nightlight value
pip install rasterio geopandas rasterstats

python scripts/extract_district_stats.py \
  --raster calibrated/india_ntl_calibrated_2005.tif \
  --boundaries boundaries/districts_2011.shp \
  --output district_stats/district_stats_2005.csv
```

> TODO: Add actual script names/usage once written.

---

## Requirements

- Python 3.x
- `rasterio`, `gdal`, `geopandas`, `rasterstats`, `numpy`
- `tensorflow` / `pytorch` (for CNN model — specify once finalized)

---

## Citation / Attribution

If you use this dataset, please cite the original sensor data providers:

- DMSP-OLS: NOAA National Geophysical Data Center (NGDC)
- VIIRS-DNB: NOAA / Earth Observation Group, Colorado School of Mines

> TODO: Add your own citation format here once the dataset/paper is finalized.

---

## License

> TODO: Specify license (e.g., MIT, CC-BY-4.0) for the code and/or derived data in this repository. Note that original satellite data may carry its own usage terms separate from your derived/calibrated outputs.

---

## Status / Roadmap

- [ ] Finalize data download scripts (DMSP + VIIRS)
- [ ] Finalize district boundary source link
- [ ] Build and train CNN calibration model
- [ ] Validate calibrated output (1992–2011) against ground truth / literature benchmarks
- [ ] Generate full district-level time series (1992–present)
- [ ] Add example analysis notebook
