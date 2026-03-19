# Agroclimatic Pipeline — Haute Matsiatra, Madagascar

> Pipeline for computing agroclimatic indicators (SPI, SPEI, ET₀, GDD, WRSI, NDVI…)  
> for the Haute Matsiatra region, Madagascar — Agricultural seasons 2011/12 to 2017/18  
> Climatological baseline: 1991–2020 (WMO standard)  
>  
> **Objective**: build a district-scale agroclimatic database (7 seasons: 2011/12 – 2017/18)  
> for the analysis of **agricultural exposure to drought** in Haute Matsiatra.

---

## Overview

This project is part of a doctoral thesis on the **agroclimatic analysis of Haute Matsiatra** (Madagascar). The two scripts aim to **build a multivariate agroclimatic database** at the district level over **7 agricultural seasons (2011/12 – 2017/18)** for the analysis of agricultural exposure to drought.

> These scripts are the **data constitution phase** of the thesis.  
> The extracted variables serve as inputs for subsequent correlation analysis  
> and drought exposure modelling.

The variables extracted cover:
- **Drought indices** (SPI, SPEI at different time scales)
- **Meteorological variables** per crop phase (precipitation, temperature, ET₀, GDD…)
- **Water balance indicators** (WRSI, water deficit, soil moisture)
- **Vegetation indicators** (NDVI, NDWI, VCI from Landsat)
- **Phenological indicators** (onset date, cessation date, season length)

The pipeline consists of **two scripts** to be run in order:

```
1. pretraitement_spi_spei_final_v2.py   →  SPI/SPEI computation (1991–2020 baseline)
2. pipeline_agroclimatique_v2.py        →  Full agroclimatic pipeline
```

---

## Project Structure

```
📦 agroclimatic-pipeline-haute-matsiatra/
├── pretraitement_spi_spei_final_v2.py   # Script 1 — SPI/SPEI
├── pipeline_agroclimatique_v2.py        # Script 2 — Full pipeline
├── README.md
│
├── 📁 inputs/  (to be provided — not included in repository)
│   ├── region_district_HM.shp           # District shapefile
│   ├── Soil_map_HM_VF.shp               # FAO soil map
│   └── rendemenrt_haute_matsiatra.xlsx  # Crop yield data
│
└── 📁 outputs/  (auto-generated in Google Drive)
    ├── SPI_SPEI_FINAL/
    │   ├── all_indices_saison.pkl
    │   ├── indices_saison_tous_districts.csv
    │   ├── validation/rapport_validation.csv
    │   └── figures/
    └── DONNEES_COMPLETES_VF/
        ├── 02_PROCESSED/ANNUAL/
        └── 03_FINAL/dataset_complet.csv
```

---

## Requirements

### Environment
- Python ≥ 3.10 (Google Colab recommended)
- Google Drive mounted in Colab

### Python Packages
```bash
pip install climate-indices earthengine-api geopandas rasterio \
            xarray netCDF4 scipy matplotlib cdsapi requests
```

### API Access
| API | Usage | Registration |
|-----|-------|--------------|
| **Google Earth Engine** | CHIRPS, ERA5-Land, SRTM, Landsat | [earthengine.google.com](https://earthengine.google.com) |
| **Copernicus CDS (ECMWF)** | Daily ERA5-Land | [cds.climate.copernicus.eu](https://cds.climate.copernicus.eu) |

---

## Usage

### Step 1 — SPI & SPEI Computation (1991–2020 baseline)

```python
# pretraitement_spi_spei_final_v2.py
# Update paths at the top of the script:
SHAPEFILE_DISTRICTS = "/content/region_district_HM.shp"
OUTPUT_DIR = "/content/drive/MyDrive/DATA_THESES_MADATLAS/SPI_SPEI_FINAL"
```

**Output files:**
- `all_indices_saison.pkl` — SPEI indices per district and agricultural season
- `indices_saison_tous_districts.csv` — readable CSV version
- `validation/rapport_validation.csv` — validation metrics
- `figures/` — time series and scatter plots

---

### Step 2 — Full Agroclimatic Pipeline

> **Prerequisite**: Step 1 must be completed first.

```python
# pipeline_agroclimatique_v2.py
# Update paths at the top of the script:
SHAPEFILE_DISTRICTS = "/content/region_district_HM.shp"
SHAPEFILE_SOLS      = "/content/Soil_map_HM_VF.shp"
EXCEL_RENDEMENTS    = "/content/rendemenrt_haute_matsiatra.xlsx"
ECMWF_API_KEY       = "your-cds-api-key"
OUTPUT_DIR          = "/content/drive/MyDrive/DATA_THESES_MADATLAS/DONNEES_COMPLETES_VF"
```

**Output files:**
- `dataset_complet.csv` / `.xlsx` — final dataset merged with crop yields

---

##  Scientific Methods

### Drought Indices

| Index | Method | Reference |
|-------|--------|-----------|
| SPI | Gamma distribution | McKee et al. (1993) |
| SPEI | Log-logistic distribution | Vicente-Serrano et al. (2010) |
| WRSI | Water Requirement Satisfaction Index | FAO-33 |

### SPEI Indices Retained

| Index | Extraction month | Rolling window | Meaning |
|-------|-----------------|----------------|---------|
| **SPEI-3** | February | Dec → Jan → Feb | Critical phase (peak rainfall) |
| **SPEI-6** | May | Dec → … → May | Full season water balance |
| **SPEI-12** | May | Jun(Y-1) → … → May | Background hydrological drought |

### Drought Categories (McKee et al. 1993)

| SPEI value | Category |
|------------|----------|
| ≥ −1.0 | Normal |
| −1.0 to −1.5 | Moderate |
| −1.5 to −2.0 | Severe |
| < −2.0 | Extreme |

### ET₀ — Penman-Monteith FAO-56

```
Sources : ERA5-Land (temperature, humidity, wind, radiation)
Altitude : SRTM 30m via GEE — variable per district
Reference: Allen et al. (1998)
```

### Other Computed Indicators

| Variable | Method / Source | Reference |
|----------|----------------|-----------|
| GDD | Growing Degree Days | McMaster & Wilhelm (1997) |
| Kc / ETc | Crop coefficients | FAO-33 |
| Season onset | 20 mm / 7-day dry spell criterion | Dunning et al. (2016) |
| Season cessation | Soil water balance | Araya et al. (2010) |
| CDD | Consecutive Dry Days | WMO (2018) |
| NDVI / NDWI | Landsat 5/7/8/9 | — |
| VCI | Vegetation Condition Index | Kogan (1995) |

---

##  Data Sources

| Data | Source | Resolution |
|------|--------|------------|
| Precipitation | [CHIRPS v2](https://www.chc.ucsb.edu/data/chirps) | ~5 km, daily |
| Weather / ET₀ | [ERA5-Land CDS](https://cds.climate.copernicus.eu) | ~9 km, daily |
| Elevation | [SRTM 30m (GEE)](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003) | 30 m |
| Vegetation | [Landsat 5/7/8/9 (GEE)](https://developers.google.com/earth-engine/datasets) | 30 m |
| SPEI reference | [SPEIbase v2.9 CSIC](https://digital.csic.es/handle/10261/288226) | ~55 km |

---

##  Crop Phases

The agricultural season in Haute Matsiatra runs from **November to May**:

```
Season Y = November(Y) → May(Y+1)

  P1 : November – December   (season establishment)
  P2 : January  – February   (peak rainfall — critical phase)
  P3 : March    – May        (maturation and harvest)
```

---

##  SPI/SPEI Validation

Script 1 performs automatic **cross-validation**:

- **Reference**: ERA5-Land native SPEI (via GEE) — or SPEIbase CSIC if NetCDF file available locally
- **Metrics**: Spearman correlation (ρ), RMSE, bias, detection rate of known drought years
- **Known drought years**: 1991, 1992, 2001, 2009, 2016, 2019, 2020, 2021

| Quality | Criterion |
|---------|-----------|
| GOOD | ρ ≥ 0.70 |
| ACCEPTABLE | ρ ≥ 0.50 |
| POOR | ρ < 0.50 |

---

##  Place of These Scripts in the Thesis

These two scripts constitute the **data constitution phase** of the thesis.

```
┌──────────────────────────────────────────────────────────────┐
│  THIS REPOSITORY — Database construction                     │
│                                                              │
│  • SPI / SPEI computation (1991–2020 baseline, WMO)         │
│  • Agroclimatic variable extraction per district             │
│    (precipitation, ET₀, GDD, WRSI, NDVI, CDD…)              │
│  • 7 agricultural seasons: 2011/12 → 2017/18                │
│  • Crops: Rice | Maize | Groundnut                          │
│                                                              │
│  OUTPUT: dataset_complet.csv                                 │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼  subsequent analyses in the thesis
              Agricultural Exposure to Drought
              Haute Matsiatra, Madagascar
```

---

## Dataset Variables

### X — Input Features (111 variables)

The dataset contains **111 numerical variables (X)** usable as model features,
and **3 target variables (Y)** — one per crop.

```
X  →  111 agroclimatic variables    (model input features)
Y  →  yield_rice_t_ha               (target — Rice)
       yield_maize_t_ha              (target — Maize)
       yield_groundnut_t_ha          (target — Groundnut)
```

| # | Variable | Category | Phase |
|---|----------|----------|-------|
| 1 | `onset_day` | Phenology | Season |
| 2 | `cessation_day` | Phenology | Season |
| 3 | `season_length_days` | Phenology | Season |
| 4 | `spei3_fev` | Drought | Season |
| 5 | `spei6_mai` | Drought | Season |
| 6 | `spei12_mai` | Drought | Season |
| 7 | `wrsi_rice` | WRSI | Season |
| 8 | `wrsi_maize` | WRSI | Season |
| 9 | `wrsi_groundnut` | WRSI | Season |
| 10 | `P1_precip_total_mm` | Precipitation | P1 |
| 11 | `P1_rainy_days` | Precipitation | P1 |
| 12 | `P1_precip_intensity_mm` | Precipitation | P1 |
| 13 | `P1_cdd` | Precipitation | P1 |
| 14 | `P1_dry_spell_mean` | Precipitation | P1 |
| 15 | `P1_extreme_days` | Precipitation | P1 |
| 16 | `P1_extreme_total_mm` | Precipitation | P1 |
| 17 | `P1_extreme_pct` | Precipitation | P1 |
| 18 | `P1_tmax_C` | Temperature | P1 |
| 19 | `P1_tmin_C` | Temperature | P1 |
| 20 | `P1_tmean_C` | Temperature | P1 |
| 21 | `P1_dtr_C` | Temperature | P1 |
| 22 | `P1_thermal_stress_days` | Temperature | P1 |
| 23 | `P1_hot_nights` | Temperature | P1 |
| 24 | `P1_gdd_rice` | GDD | P1 |
| 25 | `P1_gdd_maize` | GDD | P1 |
| 26 | `P1_gdd_groundnut` | GDD | P1 |
| 27 | `P1_rh_pct` | Atmospheric | P1 |
| 28 | `P1_vpd_kPa` | Atmospheric | P1 |
| 29 | `P1_soil_moisture` | Atmospheric | P1 |
| 30 | `P1_wind_speed_2m` | Atmospheric | P1 |
| 31 | `P1_wind_dir_deg` | Atmospheric | P1 |
| 32 | `P1_et0_mm` | ET₀ / Water | P1 |
| 33 | `P1_etc_rice_mm` | ET₀ / Water | P1 |
| 34 | `P1_etc_maize_mm` | ET₀ / Water | P1 |
| 35 | `P1_etc_groundnut_mm` | ET₀ / Water | P1 |
| 36 | `P1_water_deficit_rice_mm` | ET₀ / Water | P1 |
| 37 | `P1_water_deficit_maize_mm` | ET₀ / Water | P1 |
| 38 | `P1_water_deficit_groundnut_mm` | ET₀ / Water | P1 |
| 39 | `P1_stress_days_rice` | ET₀ / Water | P1 |
| 40 | `P1_stress_days_maize` | ET₀ / Water | P1 |
| 41 | `P1_stress_days_groundnut` | ET₀ / Water | P1 |
| 42 | `P2_precip_total_mm` | Precipitation | P2 |
| 43 | `P2_rainy_days` | Precipitation | P2 |
| 44 | `P2_precip_intensity_mm` | Precipitation | P2 |
| 45 | `P2_cdd` | Precipitation | P2 |
| 46 | `P2_dry_spell_mean` | Precipitation | P2 |
| 47 | `P2_extreme_days` | Precipitation | P2 |
| 48 | `P2_extreme_total_mm` | Precipitation | P2 |
| 49 | `P2_extreme_pct` | Precipitation | P2 |
| 50 | `P2_tmax_C` | Temperature | P2 |
| 51 | `P2_tmin_C` | Temperature | P2 |
| 52 | `P2_tmean_C` | Temperature | P2 |
| 53 | `P2_dtr_C` | Temperature | P2 |
| 54 | `P2_thermal_stress_days` | Temperature | P2 |
| 55 | `P2_hot_nights` | Temperature | P2 |
| 56 | `P2_gdd_rice` | GDD | P2 |
| 57 | `P2_gdd_maize` | GDD | P2 |
| 58 | `P2_gdd_groundnut` | GDD | P2 |
| 59 | `P2_rh_pct` | Atmospheric | P2 |
| 60 | `P2_vpd_kPa` | Atmospheric | P2 |
| 61 | `P2_soil_moisture` | Atmospheric | P2 |
| 62 | `P2_wind_speed_2m` | Atmospheric | P2 |
| 63 | `P2_wind_dir_deg` | Atmospheric | P2 |
| 64 | `P2_et0_mm` | ET₀ / Water | P2 |
| 65 | `P2_etc_rice_mm` | ET₀ / Water | P2 |
| 66 | `P2_etc_maize_mm` | ET₀ / Water | P2 |
| 67 | `P2_etc_groundnut_mm` | ET₀ / Water | P2 |
| 68 | `P2_water_deficit_rice_mm` | ET₀ / Water | P2 |
| 69 | `P2_water_deficit_maize_mm` | ET₀ / Water | P2 |
| 70 | `P2_water_deficit_groundnut_mm` | ET₀ / Water | P2 |
| 71 | `P2_stress_days_rice` | ET₀ / Water | P2 |
| 72 | `P2_stress_days_maize` | ET₀ / Water | P2 |
| 73 | `P2_stress_days_groundnut` | ET₀ / Water | P2 |
| 74 | `P2_ndvi` | Vegetation | P2 |
| 75 | `P2_ndwi` | Vegetation | P2 |
| 76 | `P2_vci` | Vegetation | P2 |
| 77 | `P3_precip_total_mm` | Precipitation | P3 |
| 78 | `P3_rainy_days` | Precipitation | P3 |
| 79 | `P3_precip_intensity_mm` | Precipitation | P3 |
| 80 | `P3_cdd` | Precipitation | P3 |
| 81 | `P3_dry_spell_mean` | Precipitation | P3 |
| 82 | `P3_extreme_days` | Precipitation | P3 |
| 83 | `P3_extreme_total_mm` | Precipitation | P3 |
| 84 | `P3_extreme_pct` | Precipitation | P3 |
| 85 | `P3_tmax_C` | Temperature | P3 |
| 86 | `P3_tmin_C` | Temperature | P3 |
| 87 | `P3_tmean_C` | Temperature | P3 |
| 88 | `P3_dtr_C` | Temperature | P3 |
| 89 | `P3_thermal_stress_days` | Temperature | P3 |
| 90 | `P3_hot_nights` | Temperature | P3 |
| 91 | `P3_gdd_rice` | GDD | P3 |
| 92 | `P3_gdd_maize` | GDD | P3 |
| 93 | `P3_gdd_groundnut` | GDD | P3 |
| 94 | `P3_rh_pct` | Atmospheric | P3 |
| 95 | `P3_vpd_kPa` | Atmospheric | P3 |
| 96 | `P3_soil_moisture` | Atmospheric | P3 |
| 97 | `P3_wind_speed_2m` | Atmospheric | P3 |
| 98 | `P3_wind_dir_deg` | Atmospheric | P3 |
| 99 | `P3_et0_mm` | ET₀ / Water | P3 |
| 100 | `P3_etc_rice_mm` | ET₀ / Water | P3 |
| 101 | `P3_etc_maize_mm` | ET₀ / Water | P3 |
| 102 | `P3_etc_groundnut_mm` | ET₀ / Water | P3 |
| 103 | `P3_water_deficit_rice_mm` | ET₀ / Water | P3 |
| 104 | `P3_water_deficit_maize_mm` | ET₀ / Water | P3 |
| 105 | `P3_water_deficit_groundnut_mm` | ET₀ / Water | P3 |
| 106 | `P3_stress_days_rice` | ET₀ / Water | P3 |
| 107 | `P3_stress_days_maize` | ET₀ / Water | P3 |
| 108 | `P3_stress_days_groundnut` | ET₀ / Water | P3 |
| 109 | `P3_ndvi` | Vegetation | P3 |
| 110 | `P3_ndwi` | Vegetation | P3 |
| 111 | `P3_vci` | Vegetation | P3 |

**Summary by category**

| Category | Count |
|----------|-------|
| Phenology | 3 |
| Drought (SPEI) | 3 |
| WRSI | 3 |
| Precipitation × 3 phases | 24 |
| Temperature × 3 phases | 18 |
| GDD × 3 phases | 9 |
| Atmospheric × 3 phases | 15 |
| ET₀ / Water balance × 3 phases | 30 |
| Vegetation + VCI (P2 & P3) | 6 |
| **TOTAL** | **111** |

### Y — Target Variables (3 variables, one per crop)

| Variable | Crop | Unit |
|----------|------|------|
| `yield_rice_t_ha` | Rice | t/ha |
| `yield_maize_t_ha` | Maize | t/ha |
| `yield_groundnut_t_ha` | Groundnut | t/ha |

---

## References

```
Allen, R.G. et al. (1998). Crop evapotranspiration. FAO Irrigation and Drainage Paper 56.
Araya, A. et al. (2010). New approaches to model the onset and cessation of seasonal rain. Agric. For. Meteorol.
Dunning, C.M. et al. (2016). Short rains variability in East Africa. Clim. Dyn.
Kogan, F.N. (1995). Application of vegetation index and brightness temperature for drought detection. Adv. Space Res.
McKee, T.B. et al. (1993). The relationship of drought frequency and duration to time scales. 8th AMS Conf.
McMaster, G.S. & Wilhelm, W.W. (1997). Growing degree-days. Agric. For. Meteorol.
Vicente-Serrano, S.M. et al. (2010). A multiscalar drought index sensitive to global warming. J. Climate.
WMO (2018). Guide to Climatological Practices. WMO-No. 100.
```

---

## Author & Supervision

**Larissa RAMIANDRISOA**  
PhD Candidate — Université de Fianarantsoa / Université Gustave Eiffel  
Year: 2025

| Role | Name |
|------|------|
| Thesis Director | **A.R. HAJALALAINA** |
| Supervisor | **Bob E. SAINT FLEUR** |
| Supervisor | **Andry RAZAKAMANANTSOA** |
| Supervisor | **Marie Lucia FANJANIAINA** |

---

## License

This project is licensed under the [MIT License](LICENSE) — free to use with attribution.

---

*Data sourced from public repositories (CHIRPS, ERA5, GEE, CSIC).  
Yield data and administrative shapefiles are not included in this repository.*
