# AQI & Weather Analysis — India

> **Exploratory and statistical analysis of air quality and meteorological data across Indian cities, examining how weather conditions drive pollutant concentrations and AQI levels.**

---

## Table of Contents

1. [Overview](#overview)
2. [Dataset](#dataset)
3. [Repository Structure](#repository-structure)
4. [Methodology](#methodology)
5. [Key Findings](#key-findings)
6. [Visualizations](#visualizations)
7. [Installation & Usage](#installation--usage)
8. [Notebooks Guide](#notebooks-guide)
9. [Variable Reference](#variable-reference)
10. [Limitations & Future Work](#limitations--future-work)

---

## Overview

Air quality in India is a public health crisis of national scale. This project systematically examines the relationship between meteorological variables (temperature, humidity, wind speed, UV index) and pollutant concentrations (PM2.5, PM10, NO2, CO, SO2) across multiple Indian cities using statistical analysis and data visualization.

The central question is: **do weather conditions drive pollutant levels uniformly across cities, or is the weather–pollution relationship city-dependent?**

The analysis finds that the answer is emphatically the latter — the influence of weather on AQI is highly heterogeneous across cities, driven by differences in local emission sources, topography, and seasonal climate patterns.

**Key computational techniques used:**

| Technique | Purpose |
|---|---|
| Missing-value imputation & outlier detection | Data cleaning and validation |
| Pearson correlation matrix + heatmap | Quantify inter-pollutant and weather–pollutant relationships |
| City-stratified group statistics | Identify city-level divergence from national trends |
| Distribution analysis (KDE, box plots) | Characterise AQI and pollutant spread per city |
| Scatter plots with regression overlays | Visualise directional weather–pollutant effects |

---

## Dataset

- **Source:** Kaggle — Indian Cities Air Quality & Weather Dataset
- **Coverage:** Multiple Indian cities (metro and non-metro)
- **Temporal scope:** Multi-year daily observations
- **Variables:** 20+ features spanning AQI components, meteorology, and wind

### Pollutant Variables

| Variable | Description | WHO Annual Guideline |
|---|---|---|
| `PM2.5` | Fine particulate matter (μg/m³) | 5 μg/m³ |
| `PM10` | Coarse particulate matter (μg/m³) | 15 μg/m³ |
| `NO2` | Nitrogen dioxide (μg/m³) | 10 μg/m³ |
| `CO` | Carbon monoxide (mg/m³) | 4 mg/m³ |
| `SO2` | Sulphur dioxide (μg/m³) | 40 μg/m³ |
| `O3` | Ozone (μg/m³) | 60 μg/m³ |
| `AQI` | Composite Air Quality Index | — |

### Meteorological Variables

| Variable | Description |
|---|---|
| `Temperature` | Ambient air temperature (°C) |
| `Humidity` | Relative humidity (%) |
| `Wind Speed` | Surface wind speed (km/h) |
| `Wind Direction` | Predominant wind direction (degrees) |
| `UV Index` | Ultraviolet radiation index |
| `Pressure` | Atmospheric pressure (hPa) |
| `Precipitation` | Daily rainfall (mm) |

---

## Repository Structure

```
air-quality-weather-analysis/
│
├── data/                          # Raw and cleaned datasets
│   ├── raw/                       # Original Kaggle CSV files
│   └── processed/                 # Cleaned and feature-engineered CSVs (Here the raw data is already filtered so both are same) 
│
├── images/                        # Saved output figures
│   ├── correlation_heatmap.png
│   ├── aqi_distribution_by_city.png
│   ├── pm25_vs_weather.png
│   └── ...
│
├── notebooks/                     # Jupyter analysis notebooks
│   └── AQI_Weather_Analysis.ipynb # Main analysis notebook
│
├── README.md
└── requirements.txt
```

---

## Methodology

### 1. Data Cleaning & Validation

Raw data from Kaggle contained missing values, inconsistent city name encodings, and occasional physical impossibilities (negative PM2.5 values from sensor noise). The cleaning pipeline:

- Identified columns with >30% missing values and flagged them for exclusion
- Applied **median imputation** for numerical columns with <30% missingness (robust to outliers versus mean imputation)
- Removed rows where AQI was null, as AQI is the primary target variable
- Detected outliers using the **IQR method** (1.5× fence) and confirmed against domain knowledge (PM2.5 > 500 μg/m³ is physically plausible during severe pollution events and was retained; negative values were dropped)
- Standardised city names to a canonical list

### 2. Exploratory Data Analysis

**National-level distributions:** Examined the AQI distribution across the full dataset. Indian cities sit predominantly in the "Poor" to "Very Poor" AQI bands (AQI 200–400), with the distribution being right-skewed — extreme pollution events pull the tail significantly above the bulk.

**City-level stratification:** Computed per-city summary statistics (mean, median, IQR, 95th percentile) for all pollutants and AQI. Cities in the Indo-Gangetic Plain (Delhi, Lucknow, Patna) show substantially higher median AQI than coastal cities (Mumbai, Chennai), consistent with the topographic trapping effect of the Himalayan barrier on northerly cold-season winds.

**Temporal patterns:** Box plots of AQI by month reveal a strong seasonal signal — winter months (November–February) show elevated pollution across all cities, driven by reduced planetary boundary layer height, increased biomass burning, and stagnant anticyclonic weather.

### 3. Correlation Analysis

The full Pearson correlation matrix was computed across all pollutants and weather variables. Key methodological choices:

- Correlation computed separately for the **full dataset** and for each **city group** to capture heterogeneity
- Heatmap visualised with diverging colormap (blue–white–red, centered at 0) using Seaborn
- Statistical significance of correlations tested at α = 0.05

### 4. Weather–Pollution Regression

For the meteorological variables with the strongest correlations to PM2.5, scatter plots with linear regression overlays (via `scipy.stats.linregress`) were produced. This quantified the direction and strength of individual weather–pollutant relationships.

---

## Key Findings

### Inter-Pollutant Correlations

> **PM2.5 shows moderate positive correlation with both NO2 and PM10 nationally.**

This is physically expected: PM2.5 and PM10 share emission sources (vehicular exhaust, construction dust, industrial emissions), and NO2 co-occurs in traffic and combustion exhaust. The correlation is not unity because PM2.5 also receives significant secondary aerosol contribution (atmospheric chemistry) while PM10 is more dominated by mechanical resuspension.

**Approximate correlation strengths (national):**

| Pair | Pearson r | Interpretation |
|---|---|---|
| PM2.5 – PM10 | ~0.65–0.75 | Moderate–strong; shared combustion sources |
| PM2.5 – NO2 | ~0.50–0.65 | Moderate; co-emitted in traffic exhaust |
| PM10 – NO2 | ~0.45–0.60 | Moderate; weaker, different size modes |
| AQI – PM2.5 | ~0.80–0.90 | Strong; PM2.5 is the dominant AQI driver |

### Weather–Pollution Relationships

> **Weather variables show weaker but city-dependent correlations with pollutant levels.**

The national-level correlations between weather and pollutants are modest, but city-level analysis reveals meaningful structure:

- **Wind speed** shows the most consistent negative correlation with PM2.5 and PM10 across cities: higher wind speed disperses surface-level particulates. The effect is stronger in cities with open topography.
- **Humidity** shows a non-monotonic relationship with PM2.5 — moderate humidity promotes hygroscopic growth of particles (raising measured PM2.5), but very high humidity (monsoon) correlates with lower PM2.5 due to wet deposition. This creates a near-zero national average that masks opposing trends at different humidity ranges.
- **Temperature** has a weak negative correlation with PM2.5 overall, consistent with the seasonal effect: higher temperatures in summer co-occur with monsoon-driven washout and improved vertical mixing.
- **UV Index** shows weak negative correlation with PM2.5, partly because high UV occurs in summer (cleaner air) and partly because photochemical reactions reduce primary aerosol loadings at the surface.

### City-Level Divergence

The most important finding is the **heterogeneity of weather sensitivity across cities**. Splitting the correlation matrix by city shows that the wind speed–PM2.5 relationship ranges from strongly negative in flat, open cities to near-zero in cities surrounded by hills (where local wind channeling dominates). This means a single national model for predicting AQI from weather would perform poorly; city-specific models are necessary.

---

## Visualizations

All figures are saved in `images/`. Key outputs:

| Figure | Description |
|---|---|
| `correlation_heatmap.png` | Full Pearson correlation matrix across all pollutants and weather variables |
| `aqi_distribution_by_city.png` | Box plots of AQI distribution per city — reveals Indo-Gangetic vs coastal divergence |
| `pm25_no2_scatter.png` | PM2.5 vs NO2 scatter with regression line — moderate positive correlation |
| `pm25_windspeed_scatter.png` | PM2.5 vs wind speed — negative relationship, city-stratified |
| `monthly_aqi_boxplot.png` | Monthly AQI seasonality — winter peak clearly visible |
| `pm25_humidity_scatter.png` | PM2.5 vs humidity — non-monotonic relationship visible at city level |

---

## Installation & Usage

### Requirements

```
pandas
numpy
matplotlib
seaborn
scipy
jupyter
```

### Setup

```bash
# Clone the repository
git clone https://github.com/vajadiye-gif/air-quality-weather-analysis.git
cd air-quality-weather-analysis

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook notebooks/AQI_Weather_Analysis.ipynb
```

### Google Colab

Open the notebook directly in Colab by navigating to the notebook file and clicking the "Open in Colab" badge, or manually:

```python
# Mount Drive and clone
from google.colab import drive
drive.mount('/content/drive')

!git clone https://github.com/vajadiye-gif/air-quality-weather-analysis.git
%cd air-quality-weather-analysis
!pip install -q seaborn scipy
```

---

## Notebooks Guide

### `notebooks/AQI_Weather_Analysis.ipynb`

The notebook is structured as a linear analysis pipeline:

**Section 1 — Data Loading & Inspection**
Load raw CSV, inspect dtypes, check shape, identify missing values per column.

**Section 2 — Data Cleaning**
Imputation, outlier handling, city name standardisation, final cleaned dataset export.

**Section 3 — National EDA**
AQI distribution (histogram + KDE), pollutant summary statistics table, pairplot of pollutants.

**Section 4 — City-Level Analysis**
Grouped statistics, AQI box plots by city, identification of highest and lowest pollution cities in the dataset.

**Section 5 — Correlation Analysis**
Full Pearson matrix heatmap. Separate heatmaps per city for the top-5 cities by data volume.

**Section 6 — Weather–Pollution Relationships**
Scatter plots with regression overlays for PM2.5 vs each weather variable. Annotated with Pearson r and p-value.

**Section 7 — Seasonal Analysis**
Monthly aggregation, box plots by month, identification of peak pollution periods.

---

## Variable Reference

### AQI Breakpoints (India — CPCB Standard)

| AQI Range | Category | Health Implication |
|---|---|---|
| 0–50 | Good | Minimal impact |
| 51–100 | Satisfactory | Minor breathing discomfort for sensitive people |
| 101–200 | Moderate | Discomfort to people with lung/heart disease |
| 201–300 | Poor | Breathing discomfort to most on prolonged exposure |
| 301–400 | Very Poor | Respiratory illness on prolonged exposure |
| 401–500 | Severe | Affects healthy people; serious risk to sensitive groups |

---

## Limitations & Future Work

**Current limitations:**

- Correlations are observational — no causal inference has been performed. Confounders (e.g., weekday vs weekend traffic, festival dates) are not controlled for.
- The dataset is static (no live API feed); findings reflect the historical period covered by the Kaggle source.
- Linear (Pearson) correlation may miss non-linear relationships; the humidity–PM2.5 non-monotonicity noted above is one example.

**Natural extensions:**

- **Time-series forecasting:** SARIMA or LSTM models to predict next-day AQI from lagged weather and pollutant data
- **Causal deweathering:** Use Random Forest to partial out meteorological variability and isolate the trend in emissions-driven pollution (standard methodology in atmospheric science)
- **Spatial interpolation:** Kriging or IDW to produce pollution maps between station locations
- **Classification:** Train a multi-class classifier to predict AQI category (Good / Moderate / Poor / Very Poor) from weather features alone — tests the predictive value of meteorology

---

## Data Source & Citation

Dataset sourced from Kaggle. If using this analysis, please cite the original dataset accordingly.

Built with **NumPy · Pandas · Matplotlib · Seaborn · SciPy** | Google Colab
