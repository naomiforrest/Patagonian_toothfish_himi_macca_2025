# Patagonian_toothfish_himi_macca_2025
Modelling the spatial distribution of Patagonian toothfish in two sub-Antarctic regions

**Author:** Naomi Forrest  
** Master's project - Institute of Marine and Antarctic Science (SZ7)
**Project Duration:** 2024-2025  
**Regions:** Heard Island & McDonald Islands (HIMI) and Macquarie Island (MI)

## Project Overview

This repository contains the complete analytical workflow for modelling Patagonian toothfish (*Dissostichus eleginoides*) age and length distributions across two critical Subantarctic fishing regions. The research applies Generalised Additive Models (GAMs) using age and lenght data from the Heard Island McDonald Island and Macquarie Island fisheries to understand spatial, temporal, and environmental drivers of toothfish population structure, providing crucial insights for fisheries management and conservation.

### Research Objectives

1. **Spatial Distribution Analysis**: Model age and length distributions across HIMI and MI regions
2. **Environmental Drivers**: Identify oceanographic and bathymetric factors influencing toothfish demographics
3. **Temporal Patterns**: Examine seasonal and inter-annual trends in population structure
4. **Management Applications**: Generate high-resolution prediction maps for fisheries management

### Key Findings

- Successfully modeled median age and length distributions for both regions
- Identified critical environmental drivers: fishing depth, ocean temperature, mixed layer depth, and salinity
- Generated high-resolution prediction maps at BRAN (~10km) and GEBCO (~500m) resolutions
- Corrected methodological issues with gear type classification (GearPurpose vs Gear)
- Applied rigorous collinearity analysis and outlier detection protocols

---

## Repository Structure

```
FINAL_PROJECT_PACKAGE/
├── README.md                          # This file
├── HIMI/                              # Heard Island & McDonald Islands
│   ├── 1_Raw_Data/                    # Original TOP database extracts
│   │   ├── TOP_Data_Ages.csv
│   │   ├── TOP_Data_Biol.csv
│   │   └── TOP_Data_Haul.csv
│   ├── 2_Final_Datasets/              # Cleaned, model-ready datasets
│   │   ├── final_himi_age_data_stage3.rds
│   │   └── final_himi_length_data_stage3.rds
│   ├── 3_Final_Models/                # Final fitted GAM models
│   │   ├── final_himi_age_model_stage3.rds
│   │   └── final_himi_length_model_stage3.rds
│   └── 4_Analysis_Scripts/            # Complete analytical workflow
│       ├── 1_Outlier_Detection.Rmd
│       ├── 2_Collinearity_Analysis.Rmd
│       ├── 3a_Stage1_Age_Model.Rmd
│       ├── 3b_Stage1_Length_Model.Rmd
│       ├── 4_Stage2_3_Correction.Rmd
│       ├── 5a_Age_Prediction_Maps.Rmd
│       └── 5b_Length_Prediction_Maps.Rmd
│
└── MI/                                # Macquarie Island
    ├── 1_Raw_Data/                    # Original TOP database extracts
    │   ├── TOP_Data_Ages.csv
    │   ├── TOP_Data_Biol.csv
    │   └── TOP_Data_Haul.csv
    ├── 2_Final_Datasets/              # Cleaned, model-ready datasets
    │   ├── final_mi_age_data_stage3.rds
    │   └── final_mi_length_data_stage3.rds
    ├── 3_Final_Models/                # Final fitted GAM models
    │   ├── final_mi_age_model_stage3.rds
    │   └── final_mi_length_model_stage3.rds
    └── 4_Analysis_Scripts/            # Complete analytical workflow
        ├── 1_Outlier_Detection.Rmd
        ├── 2_Collinearity_Analysis.Rmd
        ├── 3a_Stage1_Age_Model.Rmd
        ├── 3b_Stage1_Length_Model.Rmd
        ├── 4a_Age_Prediction_Maps.Rmd
        └── 4b_Combined_Prediction_Maps.Rmd
```

---

## Data Description

### 1. Raw Data

The raw data files originate from the Australian Toothfish Observation Program (TOP) database, containing comprehensive biological and operational data from commercial longline fisheries.

#### Files:
- **TOP_Data_Ages.csv**: Age determination data from otolith analysis
- **TOP_Data_Biol.csv**: Individual fish measurements (length, weight, sex, maturity)
- **TOP_Data_Haul.csv**: Fishing operation data (location, depth, gear, environmental conditions)

#### Data Coverage:
- **HIMI**: Commercial and research fisheries (1997-2024)
- **MI**: Commercial and research fisheries (2003-2024)
- **Total Records**: >1.5 million individual fish measurements

### 2. Final Datasets

Cleaned, standardized, and model-ready datasets used for final GAM fitting:

#### HIMI Datasets:
- `final_himi_age_data_stage3.rds`: Haul-level aggregated age data (n ≈ 8,000 hauls)
- `final_himi_length_data_stage3.rds`: Haul-level aggregated length data (n ≈ 52,000 hauls)

#### MI Datasets:
- `final_mi_age_data_stage3.rds`: Haul-level aggregated age data (n ≈ 3,500 hauls)
- `final_mi_length_data_stage3.rds`: Haul-level aggregated length data (n ≈ 18,000 hauls)

#### Variables Included:
**Response Variables:**
- `Median_Age`: Median age per haul (years)
- `Median_TL`: Median total length per haul (mm)

**Environmental Predictors:**
- `FishingDepth`: Actual fishing depth (m)
- `ocean_temp`: Sea surface temperature (°C, BRAN 0.1° resolution)
- `ocean_mld`: Mixed layer depth (m, BRAN)
- `ocean_salt`: Salinity (PSU, BRAN)
- `GEBCO_Slope`: Bathymetric slope class

**Spatial Variables:**
- `Longitude`, `Latitude`: Decimal degrees
- `te(Longitude, Latitude)`: Tensor product smooth for spatial effects

**Temporal Variables:**
- `Year`: Year of capture (categorical)
- `Month`: Month of capture (cyclic smooth)
- `Day_of_year`: Julian day (1-365)
- `Biological_Season`: Spawning/non-spawning

**Operational Variables:**
- `GearPurpose`: Gear type and fishing purpose (4 levels: Commercial Longline, Research Longline, Commercial Trawl, Research Trawl)
- `Gear`: Simplified gear type (2 levels: Longline, Trawl)

### 3. Final Models

GAM objects fitted using `mgcv::gam()` with Gamma family (log link):

#### HIMI Models:
- **Age Model**: `Median_Age ~ s(FishingDepth, k=10) + s(ocean_temp, k=8) + s(ocean_mld, k=8) + s(ocean_salt, k=8) + te(Longitude, Latitude, k=20) + s(Month, bs='cc', k=12) + s(Year, k=5) + GearPurpose + GEBCO_Slope_Class + Biological_Season`
  - AIC: 41,953.39
  - Deviance explained: 61.2%
  - n = 7,892 hauls

- **Length Model**: `Median_TL ~ s(FishingDepth, k=10) + s(ocean_temp, k=8) + s(ocean_mld, k=8) + s(ocean_salt, k=8) + te(Longitude, Latitude, k=20) + s(Month, bs='cc', k=12) + s(Year, k=5) + GearPurpose + GEBCO_Slope_Class + Biological_Season`
  - Deviance explained: 47.8%
  - n = 51,923 hauls

#### MI Models:
- **Age Model**: Mean age GAM with similar structure adapted for MI region
  - n = 3,421 hauls
  
- **Length Model**: Median length GAM adapted for MI spatial extent
  - n = 17,845 hauls

---

## Analysis Scripts

### Workflow Overview

The analytical workflow follows a rigorous, stepwise approach:

```
Raw Data → Outlier Detection → Collinearity Analysis → Stage 1 Modeling → 
Stage 2/3 Optimization → Validation/Correction → Prediction Maps
```

### Script Descriptions

#### 1. Outlier Detection (`1_Outlier_Detection.Rmd`)

**Purpose:** Identify and flag statistical outliers using the IQR method (Tukey, 1977)

**Methods:**
- Interquartile Range (IQR) analysis for continuous variables
- DBSCAN clustering for geographic outliers
- Visualization using box plots and spatial maps

**Key Outputs:**
- Outlier summary tables
- Box plots for all predictor variables
- Outlier percentage by variable and region

**Reference:** Following Zuur et al. (2009) data exploration protocols

---

#### 2. Collinearity Analysis (`2_Collinearity_Analysis.Rmd`)

**Purpose:** Assess multicollinearity among environmental and spatial predictors

**Methods:**
- Pearson correlation matrices
- Variance Inflation Factors (VIF) calculation
- Moran's I spatial autocorrelation tests
- Temporal autocorrelation (ACF) analysis

**Thresholds:**
- High correlation: |r| > 0.7
- VIF concern: VIF > 5
- Spatial autocorrelation: Moran's I significance testing

**Key Findings:**
- HIMI: Strong correlation between ocean_salt and FishingDepth (r > 0.7)
- MI: Moderate correlations among oceanographic variables
- No perfect multicollinearity detected in final predictor sets

---

#### 3. Stage 1 Model Development (`3a_Stage1_Age_Model.Rmd`, `3b_Stage1_Length_Model.Rmd`)

**Purpose:** Ecological stepwise variable selection with proper GAM diagnostics

**Methods:**
- Start with full model including all ecologically relevant variables
- Systematic removal of one variable at a time
- Model comparison using AIC, deviance explained, and cross-validation
- Comprehensive diagnostics: `gam.check()`, residual plots, concurvity analysis

**Model Family:** Gamma with log link (appropriate for continuous positive data)

**K Values (basis dimensions):**
- Spatial: `te(Longitude, Latitude, k=20-25)` → 400-625 basis functions
- Environmental: `k=8-12` for depth and ocean variables
- Temporal: `k=12` for Month (cyclic), `k=5-8` for Year

**Selection Criteria:**
- ΔAIC < 2 threshold for competing models
- 100-fold cross-validation (RMSE, MAE, R²)
- Biological interpretability
- Smooth function EDF inspection

**Key Decisions:**
- HIMI Age: Retained all variables; optimal model = full model
- HIMI Length: Retained all variables; optimal model = full model
- MI Age: Mean age calculation with sample size weighting
- MI Length: Median length aggregation at haul level

---

#### 4. Stage 2/3 Correction (`4_Stage2_3_Correction.Rmd` - HIMI only)

**Purpose:** Fix critical error in HIMI Stage 2-3 parameterization

**Issue Identified:**
- Stage 2-3 models incorrectly used `Gear` (2 levels: Longline, Trawl)
- Should have used `GearPurpose` (4 levels: including Research vs Commercial)
- Missing research fishery distinction led to biased predictions

**Correction Applied:**
- Refit best models with `GearPurpose` instead of `Gear`
- Used optimal k values from Stage 2-3 parameterization
- Validated correction through AIC comparison and diagnostic plots

**Results:**
- **Age Model**: Stage 1 optimal model retained (AIC: 41,953.39 vs corrected: 50,335.29)
  - ΔAIC = 8,381.9 indicated overfitting in corrected model
  - **Final Decision**: Use Stage 1 optimal model (GearPurpose included correctly from start)

- **Length Model**: GearPurpose correction methodologically correct and improved model
  - **Final Decision**: Use Stage 1 optimal model with GearPurpose

**Critical Note:** This correction script documents the methodological verification process. Final models (`final_himi_*_model_stage3.rds`) are the **Stage 1 optimal models**, which had GearPurpose specified correctly from the beginning.

---

#### 5. Prediction Maps (`5a_Age_Prediction_Maps.Rmd`, `5b_Length_Prediction_Maps.Rmd`)

**Purpose:** Generate high-resolution spatial predictions for fisheries management

**Prediction Approaches:**

**A. BRAN Resolution (~10km)**
- Uses real environmental gradients from BRAN oceanographic data (0.1° × 0.1°)
- Environmentally realistic but coarser spatial resolution
- Better for understanding environmental drivers

**B. GEBCO Native Resolution (~500m)**
- Uses GEBCO bathymetry at native fine-scale resolution (0.042°)
- Representative (mean) environmental values applied uniformly
- ~20× more prediction points than BRAN
- Superior visual alignment with bathymetric contours
- Higher spatial precision for management applications

**Map Features:**
- Publication-quality 600 dpi output (16×12 inches)
- Bathymetric contours (200m, 500m, 1000m, 2000m)
- EEZ boundaries and Marine Protected Areas
- Island features and coastlines
- Color scales optimized for colorblind accessibility

**Key Outputs:**
- Age distribution maps (years): Showing spatial patterns of older/younger fish
- Length distribution maps (mm): Showing spatial patterns of size structure
- Size/age class summaries by environmental conditions
- Uncertainty maps (standard errors)

**Management Applications:**
- Identify nursery areas (younger/smaller fish concentrations)
- Locate spawning aggregations (older/larger fish)
- Inform spatial closures and gear restrictions
- Support ecosystem-based fisheries management

---

## Methodological Framework

### GAM Specification

**Family:** Gamma with log link function  
**Reasoning:** Median age and length are continuous positive values; Gamma family handles right-skewed distributions better than Gaussian

**Smooth Specification:**
- `select=TRUE`: Enables automatic smooth selection via additional penalty (Marra & Wood 2011)
- Cyclic smooth for Month: `bs='cc'` to handle circular nature of calendar months
- Tensor product for spatial interaction: `te(Longitude, Latitude)` allows anisotropic spatial patterns

**Model Formula Structure:**
```r
Response ~ s(Depth) + s(Temp) + s(MLD) + s(Salt) +     # Environmental
           te(Lon, Lat) +                                # Spatial
           s(Month, bs='cc') + s(Year) +                # Temporal
           GearPurpose + Slope + BioSeason               # Categorical
```

### Key Methodological Decisions

1. **Aggregation to Haul Level:**
   - Individual fish → haul-level median/mean
   - Reduces pseudo-replication from multiple fish per haul
   - More appropriate for spatial modeling of fishing operations

2. **Length-Stratified Weighting (Age Models):**
   - Age data is size-biased (preferential sampling of larger fish)
   - Applied length-bin weights to correct for size-selection bias
   - Ensures age predictions representative of true population

3. **Environmental Data:**
   - BRAN2020: 0.1° × 0.1° resolution oceanographic reanalysis
   - GEBCO 2023: Global bathymetry at ~15 arc-second resolution
   - Temporal matching: Environmental variables matched to haul date

4. **Cross-Validation:**
   - 100-fold Monte Carlo cross-validation
   - Random 80/20 train/test splits
   - Metrics: RMSE, MAE, R², Spearman correlation

5. **Model Selection:**
   - Ecological stepwise approach (variable removal, not addition)
   - AIC as primary criterion (ΔAIC < 2 threshold)
   - Diagnostic validation (residuals, concurvity, EDF)
   - Cross-validation performance
   - Biological interpretability

---

## Software and Dependencies

### R Version
- R version 4.3.0 or higher

### Key Packages
- **mgcv** (>= 1.9-0): GAM fitting and diagnostics
- **tidyverse** (>= 2.0.0): Data manipulation and visualization
- **sf** (>= 1.0-14): Spatial data handling
- **terra** / **raster**: Raster data processing
- **viridis**: Colorblind-friendly color scales
- **gratia**: GAM visualization and diagnostics
- **ggplot2** (>= 3.4.0): Publication-quality graphics

### Data Processing
- **dplyr**: Data wrangling
- **tidyr**: Data reshaping
- **lubridate**: Date/time handling

### Spatial Analysis
- **sp**: Spatial objects and analysis
- **spdep**: Spatial autocorrelation tests
- **dbscan**: Geographic outlier detection

---

## Reproducibility Notes

### File Paths
All scripts use relative paths from the project root:
```r
knitr::opts_knit$set(root.dir = "C:/Toothfish_Modelling_2024")
```

**For reproduction:** Update the root directory path to your local project location.

### Random Seeds
Where applicable, random seeds are set for reproducibility:
```r
set.seed(123)
```

### Computational Requirements
- **RAM**: Minimum 16GB recommended (32GB for prediction maps)
- **Processing**: Multi-core recommended for cross-validation
- **Storage**: ~5GB for data, models, and outputs

### Expected Runtime
- Outlier Detection: ~5-10 minutes
- Collinearity Analysis: ~10-15 minutes
- Stage 1 Modeling: ~30-60 minutes per model
- Prediction Maps: ~15-30 minutes per map (depending on resolution)

---

## Data Provenance

### Primary Data Source
Australian Antarctic Division (AAD) - Toothfish Observation Program (TOP) Database

### Environmental Data Sources
1. **BRAN2020**: Blue Ocean Research and Advanced National (BRAN) oceanographic reanalysis
   - Provider: CSIRO (Commonwealth Scientific and Industrial Research Organisation)
   - Variables: ocean_temp, ocean_mld, ocean_salt
   - Resolution: 0.1° × 0.1°, daily

2. **GEBCO 2023**: General Bathymetric Chart of the Oceans
   - Provider: GEBCO Compilation Group
   - Variables: Depth, slope
   - Resolution: 15 arc-second (~450m)

### Spatial Boundaries
- EEZ boundaries: Marine Regions / VLIZ
- Marine Protected Areas: Australian Government Department of Agriculture, Water and the Environment

---

## Citation

If you use this code or data, please cite:

**Forrest, N.** (2025). *Spatial and environmental drivers of Patagonian toothfish age and length distributions in Subantarctic waters*. Australian Antarctic Division, Kingston, Tasmania.

### Related Publications
- [Add publication details when available]

---

## Acknowledgments

This research was conducted as part of the Australian Antarctic Division's toothfish research program. Special thanks to:

- **Dale Maschette**: Statistical consultation and GAM methodology guidance
- **AAD Toothfish Team**: Data provision and biological expertise
- **CSIRO**: BRAN oceanographic data
- **GEBCO**: Global bathymetry data

---

## Contact

**Naomi Forrest**  
Australian Antarctic Division  
Email: [Add email if appropriate]

---

## License

[Specify license - e.g., CC BY 4.0, MIT, etc.]

---

## Version History

**v1.0 (October 2025)**
- Initial release for GitHub
- Complete HIMI and MI workflows
- Final Stage 3 models included
- Publication-ready prediction maps

---

## Known Issues and Future Directions

### Current Limitations
1. **MI Age Data**: Mean age calculation rather than median (due to sample size per haul)
2. **Environmental Resolution**: BRAN 0.1° resolution may miss fine-scale features
3. **Temporal Scope**: Analysis limited to historical data (1997-2024)

### Future Extensions
1. **Climate Change Projections**: Incorporate future climate scenarios
2. **Stock Assessment Integration**: Link spatial models to assessment models
3. **Real-Time Updates**: Automated pipeline for new data ingestion
4. **Additional Regions**: Extend to other toothfish fisheries (e.g., Kerguelen Plateau)

---

**Repository Last Updated:** October 27, 2025

