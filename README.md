# Kenya Health Infrastructure Analysis

## A Human-Centered AI Research Project

### Project Overview

This is a multi-part HCAI (Human-Centered AI) research project investigating whether geographic accessibility to health facilities in Kenya reflects population needs or historical infrastructure bias.

**Research Questions:**

1. What is the current state of Kenya's health facility data quality?
2. How can disparate datasets be integrated into a unified county-level view?
3. What patterns emerge in the geographic distribution of health infrastructure?
4. Can we accurately predict infrastructure scores from facility metrics?
5. Do predictive models exhibit systematic bias across regions or infrastructure tiers?
6. What are the risks of using such models for resource allocation?

**Project Phases:**

| Phase | Notebook | Focus |
|-------|----------|-------|
| 1 | `data_cleaning_and_exploration.ipynb` | Data quality assessment and cleaning |
| 2 | `merging_and_enrichment.ipynb` | County aggregation and feature engineering |
| 3 | `modeling.ipynb` | Predictive model development and evaluation |
| 4 | `fairness_and_explainability.ipynb` | Fairness audit and SHAP explainability |


## Repository Structure

```
kenya-health-infrastructure/
│
├── data/
│   ├── raw/                    # Original datasets (not committed to git)
│   │   ├── health_units.csv
│   │   ├── hospital_beds.csv
│   │   ├── hospitals_and_beds.csv
│   │   └── pharmacies.csv
│   │
│   └── processed/              # Cleaned and merged outputs
│       ├── health_units_clean.csv
│       ├── pharmacies_clean.csv
│       ├── hospitals_and_beds_clean.csv
│       ├── county_beds_clean.csv
│       ├── county_master.csv           # Final integrated dataset
│       ├── model_results.csv           # Predictions and residuals
│       ├── shap_values.csv             # SHAP values per county/feature
│       └── fairness_audit.csv          # County-level fairness data
│
├── notebooks/
│   ├── data_cleaning_and_exploration.ipynb
│   ├── merging_and_enrichment.ipynb
│   ├── modeling.ipynb
│   └── fairness_and_explainability.ipynb
│
├── outputs/
│   ├── figures/                # All visualizations from all notebooks
│   │   ├── health_unit_status.png
│   │   ├── county_health_unit_distribution.png
│   │   ├── beds_per_county.png
│   │   ├── pharmacy_distribution.png
│   │   ├── ownership_types.png
│   │   ├── infrastructure_score_by_county.png
│   │   ├── beds_vs_functionality_scatter.png
│   │   ├── correlation_heatmap.png
│   │   ├── model_comparison.png
│   │   ├── feature_importance.png
│   │   ├── residuals_by_county.png
│   │   ├── actual_vs_predicted.png
│   │   ├── fairness_by_group.png
│   │   ├── mean_residual_by_region.png
│   │   ├── shap_summary.png
│   │   └── shap_bar.png
│   │
│   └── reports/                # Summary reports
│       ├── fairness_by_region.csv
│       └── fairness_by_tier.csv
│
├── README.md                   # This file
└── requirements.txt            # Python dependencies
```

## Data Sources

All data was acquired from **[open.africa](https://open.africa/dataset/kenya-master-health-facility-list-2020)** , a pan-African open data platform that hosts government and institutional datasets.

| Dataset | Description | Rows | Key Columns |
|---------|-------------|------|-------------|
| `health_units.csv` | Master list of health facilities with operational status | 5,558 | Status, Facility, Location hierarchy, Code, Date_established |
| `hospital_beds.csv` | County-level hospital bed counts | 48 | Geography, Number of hospital beds |
| `hospitals_and_beds.csv` | Facility-level bed and operational data | 12,394 | Code, Name, Beds, Owner type, Keph level, Operational status |
| `pharmacies.csv` | Pharmacy registry with regulatory information | 5,219 | Code, Name, Registration_number, Regulatory body, Owner |




## Phase 1: Data Cleaning & Exploration (`data_cleaning_and_exploration.ipynb`)

### Overview

This notebook establishes the data foundation by cleaning individual datasets, assessing data quality, and performing initial exploratory analysis.

### Data Cleaning Summary

#### 1. Health Units

| Issue | Solution |
|-------|----------|
| Status typo: 'non-functinoal' | Corrected to 'non-functional' |
| Date_established as Excel serial numbers | Converted to datetime; values ≤3000 treated as 4-digit years |
| Mixed case in location columns | Standardized to lowercase |
| Duplicate facilities | Removed based on unique Code |



#### 2. Pharmacies

| Issue | Solution |
|-------|----------|
| 97% missing Registration_number | Added `Is_registered` flag instead of dropping |
| Service_names completely empty | Dropped column |
| Yes/No columns as strings | Converted to boolean (True/False) |
| Mixed case county names | Standardized to lowercase |



#### 3. Hospitals and Beds

| Issue | Solution |
|-------|----------|
| Beds column as string | Converted to numeric |
| 68.8% facilities with 0 beds | Analyzed by facility type |
| Inpatient facilities with 0 beds | Flagged as `Beds_data_suspect` |
| Service_names empty | Dropped column |
| Yes/No columns as strings | Converted to boolean |



#### 4. County Beds

| Issue | Solution |
|-------|----------|
| 'Kenya' summary row included | Removed for county analysis (total preserved) |
| Mixed case county names | Standardized to lowercase |


## Phase 2: Merging & Enrichment (`merging_and_enrichment.ipynb`)

### Overview

This notebook aggregates facility-level data to county level, integrates population data, and engineers features for infrastructure scoring and modeling.

### County Name Standardization

Three specific inconsistencies were corrected:

| Original | Standardized |
|----------|--------------|
| elgeyo marakwet | elgeyo-marakwet |
| muranga | murang'a |
| tharaka nithi | tharaka-nithi |

### Aggregated Metrics (per county)

| Metric | Source | Description |
|--------|--------|-------------|
| Total_beds | county_beds | Official county-level bed count |
| Total_facilities | hospitals_beds | All registered facilities |
| Status counts | health_units | Facilities by operational status |
| Total_pharmacies | pharmacies | All pharmacy records |
| Registered_pharmacies | pharmacies | Pharmacies with registration number |
| Public_facility_ratio | hospitals_beds | Proportion of public facilities |
| Suspect_zero_bed_count | hospitals_beds | Inpatient facilities with 0 beds |

### Enrichment (per-capita metrics)

| Metric | Formula | Purpose |
|--------|---------|---------|
| Beds_per_10k | (Total_beds / Population) × 10,000 | Compare to WHO standard (10 beds/10k) |
| Facilities_per_10k | (Total_facilities / Population) × 10,000 | Facility density |
| Pharmacies_per_10k | (Total_pharmacies / Population) × 10,000 | Pharmacy access |

### Infrastructure Score Formula

Composite score combining three normalized components (equal weight):

```
Infrastructure_score = (Score_beds + Score_functional + Score_public) / 3
```

Where:
- `Score_beds` = normalized Beds_per_10k (0-1 scale)
- `Score_functional` = normalized % fully functional (0-1 scale)
- `Score_public` = Public_facility_ratio (already 0-1)

## Phase 3: Predictive Modeling (`modeling.ipynb`)

### Overview

This notebook develops and evaluates machine learning models to predict county-level health infrastructure scores using seven engineered features.

### Features Used

| Feature | Description |
|---------|-------------|
| Beds_per_10k | Hospital bed availability (per 10k population) |
| Facilities_per_10k | Health facility density (per 10k population) |
| Pharmacies_per_10k | Pharmacy access (per 10k population) |
| Pct_fully_functional | Percentage of facilities operating at full capacity |
| Pct_non_functional | Percentage of facilities not operating |
| Pct_pharmacies_registered | Pharmacy regulatory compliance rate |
| Suspect_zero_bed_count | Count of facilities with suspicious zero-bed reporting |

### Models Evaluated

| Model | Type | Strengths | Weaknesses |
|-------|------|-----------|------------|
| Linear Regression | Baseline | Simple, interpretable | Prone to overfitting with small datasets |
| Ridge Regression | Linear with L2 regularization | Resists overfitting, handles multicollinearity | Requires alpha tuning |
| Random Forest | Ensemble of decision trees | Captures non-linear relationships | Less interpretable, underperformed here |

### Cross-Validation Method

**Leave-One-Out Cross-Validation (LOOCV):**
- Appropriate for small datasets (n=47)
- Each iteration trains on 46 counties, tests on 1
- Provides unbiased performance estimate
- Creates 47 unique train-test splits

### Performance Results (LOOCV)

| Model | MAE | RMSE | R² |
|-------|-----|------|-----|
| Linear Regression | 0.0003 | 0.0003 | 1.0000 |
| **Ridge Regression** | **0.0028** | **0.0037** | **0.9991** |
| Random Forest | 0.0388 | 0.0510 | 0.8180 |

**Selected Model:** Ridge Regression (alpha = 1.0)

### Feature Importance (Permutation Method)

| Feature | Importance (ΔR²) |
|---------|------------------|
| Beds_per_10k | 0.755 |
| Pct_fully_functional | 0.632 |
| Pct_non_functional | 0.015 |
| Pharmacies_per_10k | 0.006 |
| Facilities_per_10k | 0.003 |
| Suspect_zero_bed_count | 0.002 |
| Pct_pharmacies_registered | 0.001 |

Bed availability and facility functionality dominate predictions, accounting for over 95% of explained variance.

### Residual Analysis Highlights

**Most Overestimated Counties (Model predicted better than reality):**

| County | Actual | Predicted | Residual |
|--------|--------|-----------|----------|
| Machakos | 0.196 | 0.203 | -0.007 |
| Meru | 0.249 | 0.255 | -0.006 |
| Bomet | 0.173 | 0.179 | -0.006 |

**Most Underestimated Counties (Reality better than predicted):**

| County | Actual | Predicted | Residual |
|--------|--------|-----------|----------|
| Kisii | 0.537 | 0.528 | +0.009 |
| Tharaka-Nithi | 0.603 | 0.594 | +0.009 |
| Nairobi | 0.490 | 0.483 | +0.007 |


### Model Card

This model predicts a county's health infrastructure score (0–1) from seven features derived from the Kenya Master Health Facility List (KMHFL).

The intended use is exploratory analysis and research. To identify which counties appear underserved relative to their feature profile, and 
to surface counties where the model's predictions are unreliable.

This is not to be used for direct resource allocation decisions without human review. Any use that treats the score as a ground truth measure of health quality.

The Kenya KMHFL data  can be downloaded from [open.africa.](https://open.africa/dataset/kenya-master-health-facility-list-2020) The data reflect facility registration records,  not service quality audits.

**PERFORMANCE (Leave-One-Out Cross-Validation)**
| Model | MAE | RMSE | R² |
|-------|-----|------|-----|
| Ridge Regression | 0.0028 | 0.0037 | 0.9991 |
| Random Forest | 0.0388 | 0.0510 | 0.8180 |

**KNOWN LIMITATIONS**
1. Only 47 data points. Any model based on 47 rows should be treated as exploratory rather than conclusive.
2. The infrastructure score target variable was built by us using equal weights for beds, functionality, and public access. These weights were not validated with health policy experts or affected communities.
3. Public facility ownership data could not be verified and was excluded. This means the model does not account for how accessible facilities actually are to low-income populations.
4. The KMHFL reflects registered facilities. Unregistered or informal health infrastructure, which is more common in underserved counties, is entirely invisible to this model.
5. Counties where the model overestimates the infrastructure score are at risk of receiving fewer resources if predictions are used naively.


## Phase 4: Fairness and Explainability (`fairness_and_explainability.ipynb`)

### Overview

This notebook audits the Ridge Regression model for fairness across geographic regions and infrastructure tiers, and explains predictions using SHAP (SHapley Additive exPlanations).

### Fairness Groups

| Group Type | Categories | Purpose |
|------------|------------|---------|
| **Geographic Region** | Nairobi, Coast, Nyanza, Western, Rift Valley, Central, Eastern, Arid North | Detect regional bias |
| **Score Tier** | Bottom third, Middle third, Top third | Assess performance equity across infrastructure levels |

### Fairness Metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **MAE** | Mean \|Actual - Predicted\| | Higher = less accurate for group |
| **Mean Residual** | Mean (Actual - Predicted) | Negative = overestimation; Positive = underestimation |
| **Bias Direction** | Sign of mean residual | Identifies systematic bias |

### Region Mapping (47 Counties to 8 Regions)

```python
REGION_MAPPING = {
    # Nairobi (1 county)
    'nairobi': 'Nairobi',
    
    # Coast (6 counties)
    'mombasa': 'Coast', 'kilifi': 'Coast', 'kwale': 'Coast',
    'lamu': 'Coast', 'taita taveta': 'Coast', 'tana river': 'Coast',
    
    # Nyanza (6 counties)
    'kisumu': 'Nyanza', 'homa bay': 'Nyanza', 'migori': 'Nyanza',
    'kisii': 'Nyanza', 'nyamira': 'Nyanza', 'siaya': 'Nyanza',
    
    # Western (4 counties)
    'kakamega': 'Western', 'bungoma': 'Western', 'busia': 'Western', 'vihiga': 'Western',
    
    # Rift Valley (13 counties)
    'uasin gishu': 'Rift Valley', 'nakuru': 'Rift Valley', 'kericho': 'Rift Valley',
    'bomet': 'Rift Valley', 'nandi': 'Rift Valley', 'baringo': 'Rift Valley',
    'laikipia': 'Rift Valley', 'narok': 'Rift Valley', 'kajiado': 'Rift Valley',
    'trans nzoia': 'Rift Valley', 'west pokot': 'Rift Valley', 'samburu': 'Rift Valley',
    'elgeyo-marakwet': 'Rift Valley',
    
    # Arid North (6 counties)
    'turkana': 'Arid North', 'marsabit': 'Arid North', 'isiolo': 'Arid North',
    'mandera': 'Arid North', 'wajir': 'Arid North', 'garissa': 'Arid North',
    
    # Central (5 counties)
    'kiambu': 'Central', "murang'a": 'Central', 'kirinyaga': 'Central',
    'nyeri': 'Central', 'nyandarua': 'Central',
    
    # Eastern (6 counties)
    'meru': 'Eastern', 'tharaka-nithi': 'Eastern', 'embu': 'Eastern',
    'kitui': 'Eastern', 'machakos': 'Eastern', 'makueni': 'Eastern'
}
```

### Fairness Results by Region

| Region | Count | MAE | Mean Residual | Bias |
|--------|-------|-----|---------------|------|
| Nairobi | 1 | 0.00693 | +0.00693 | Underestimated |
| Eastern | 6 | 0.00638 | -0.00149 | Overestimated |
| Nyanza | 6 | 0.00431 | +0.00431 | Underestimated |
| Rift Valley | 13 | 0.00201 | -0.00044 | Overestimated |
| Central | 5 | 0.00179 | -0.00157 | Overestimated |
| Coast | 6 | 0.00162 | -0.00134 | Overestimated |
| Western | 4 | 0.00157 | +0.00019 | Underestimated |
| Arid North | 6 | 0.00148 | +0.00018 | Underestimated |

Systematic bias varies by region. Nairobi and Nyanza are underestimated; Eastern, Central, Coast, and Rift Valley are overestimated.

### Fairness Results by Score Tier

| Tier | Count | MAE | Mean Residual | Bias |
|------|-------|-----|---------------|------|
| Top third | 16 | 0.00378 | +0.00366 | Underestimated |
| Bottom third | 16 | 0.00291 | -0.00282 | Overestimated |
| Middle third | 15 | 0.00160 | -0.00062 | Overestimated |

The model is least reliable in the most underserved counties (the bottom third), which are systematically overestimated.

### SHAP Feature Impact

| Feature | Mean Absolute SHAP |
|---------|-------------------|
| Beds_per_10k | 0.755 |
| Pct_fully_functional | 0.632 |
| Pct_non_functional | 0.016 |
| Pharmacies_per_10k | 0.006 |
| Facilities_per_10k | 0.003 |
| Suspect_zero_bed_count | 0.002 |
| Pct_pharmacies_registered | 0.001 |

The model's decisions are driven almost entirely by bed availability and facility functionality.

### Model Blind Spots (Identified via SHAP)

1. **No visibility into informal facilities:** Unregistered health infrastructure is invisible
2. **No quality metrics:** Only counts and functional status, not service quality
3. **Missing ownership data:** Public vs. private accessibility unknown
4. **Registered facilities only:** Undercounts infrastructure in underserved areas

### Fairness Audit Summary

**OVERALL FINDING**

The model produces very small residuals across all 47 counties. At first glance this looks like strong performance. But this partly reflects the fact that the target variable (Infrastructure Score) was built from the same data the model trains on. Small errors are partly good modelling and partly circular reasoning. Any deployment must acknowledge this.

**FAIRNESS BY REGION**

The model errors are not evenly distributed. Some regions show consistent overestimation where the model makes them look slightly better than they are. For regions that are already underserved, even small overestimation matters because it reduces the apparent urgency of intervention.

**FAIRNESS BY SCORE TIER**

Counties in the bottom third of the infrastructure score experience different model behaviour from counties in the top third. This is the most important finding for HCAI purposes: the model is least reliable for the counties that need the most attention.

**RESOURCE ALLOCATION RISK**

When we simulated using model predictions to find the 10 most underserved counties, some counties that appear in the actual bottom 10 were missed. A policymaker using predictions instead of actual data would under-allocate resources to those specific counties. That is the most concrete harm pathway from this model.

**WHAT RESPONSIBLE USE LOOKS LIKE**

Use this model to generate questions, not answers. A county flagged as underserved should trigger a conversation with county health officers and not an automatic resource allocation decision.

## Key Findings

### Phase 1-2: Data Quality & Distribution

| Finding | Implication |
|---------|-------------|
| Health units have no missing data | Reliable for status and location analysis |
| Pharmacies have 97% missing registration numbers | Regulatory record-keeping needs improvement |
| 606 inpatient facilities report zero beds | Bed count data requires cautious interpretation |
| All 47 counties matched across datasets | Reliable for county-level merging |

**Geographic Distribution:**

| Metric | Highest | Lowest | Ratio |
|--------|---------|--------|-------|
| Health units | Nairobi (1,005) | Isiolo (20) | 50:1 |
| Hospital beds | Nairobi (9,504) | Lamu (233) | 41:1 |
| Beds per 10k | Tharaka-Nithi (18.4) | Kwale (6.0) | 3:1 |

**10 counties fall below WHO minimum of 10 beds per 10,000 people.**

**Infrastructure Score (Top 5):** Tharaka-Nithi (0.603), Embu (0.568), Migori (0.555), Nyeri (0.543), Kisii (0.537)

**Infrastructure Score (Bottom 5):** West Pokot (0.147), Bomet (0.173), Machakos (0.196), Trans Nzoia (0.197), Narok (0.203)

### Phase 3: Modeling

| Finding | Implication |
|---------|-------------|
| Ridge Regression achieves R² = 0.9991 | Excellent fit, but small sample size (n=47) limits generalizability |
| Beds and functionality dominate predictions | Model relies on traditional infrastructure metrics |
| Linear Regression shows perfect fit | Potential overfitting despite LOOCV |
| Random Forest underperforms | Relationship appears largely linear |

### Phase 4: Fairness

| Finding | Implication |
|---------|-------------|
| Systematic bias by region | Some regions consistently overestimated, others underestimated |
| Bottom tier counties overestimated | Model is least reliable for those who need it most |
| MAE varies by group (0.0015 to 0.0069) | Fairness disparities exist despite small absolute errors |
| SHAP confirms feature dominance | Model's blind spots (informal facilities) affect underserved areas |

### Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Overestimation of underserved counties | High | Manual review of all bottom-tier predictions |
| Regional bias | Medium | Region-specific calibration if deployed |
| Missing informal infrastructure | High | Supplement with community health worker data |
| Small sample size | High | Treat as exploratory only |

## Quick Reference

### Infrastructure Score Formula

```
Infrastructure_score = (Beds_per_10k_norm + Facilities_per_10k_norm + Pharmacies_per_10k_norm) / 3
```

Where each component is normalized to 0-1 using min-max scaling.

### WHO Benchmark

| Metric | Benchmark |
|--------|-----------|
| Beds per 10,000 population | 10 (minimum) |

**Counties below WHO benchmark:** 10 counties

### Model Performance Summary

| Metric | Ridge Regression |
|--------|------------------|
| MAE | 0.0028 |
| RMSE | 0.0037 |
| R² | 0.9991 |

### Fairness Summary

| Finding | Severity |
|---------|----------|
| Regional bias exists | Medium |
| Bottom tier overestimated | High |
| Model blind spots (informal facilities) | High |

### Key Risk

**Overestimation of underserved counties** Potential under-allocation of resources to counties that need them most.

**Mitigation:** Manual review of all bottom-tier predictions before any resource allocation decision.

## Output Files Summary

### Data Files (`data/processed/`)

| File | Source | Description | Key Columns |
|------|--------|-------------|-------------|
| `county_master.csv` | Phases 1-2 | Final integrated dataset | 23 columns including all features and target |
| `model_results.csv` | Phase 3 | Predictions and residuals | County, Population, Infrastructure_score, Predicted, Residual, Direction |
| `shap_values.csv` | Phase 4 | SHAP values per county/feature | County, [7 feature columns] |
| `fairness_audit.csv` | Phase 4 | County-level fairness data | County, Region, Score_tier, Population, Infrastructure_score, Predicted, Residual, Direction |

### Report Files (`outputs/reports/`)

| File | Source | Description |
|------|--------|-------------|
| `fairness_by_region.csv` | Phase 4 | Regional fairness summary (Group, Count, MAE, Mean_residual, Bias_direction) |
| `fairness_by_tier.csv` | Phase 4 | Tier-based fairness summary |

### Figure Files (`outputs/figures/`)

| File | Source | Description |
|------|--------|-------------|
| `health_unit_status.png` | Phase 1 | Facility operational status bar chart |
| `county_health_unit_distribution.png` | Phase 1 | Top/bottom 10 counties by facilities |
| `beds_per_county.png` | Phase 1 | Hospital beds by county (horizontal bar) |
| `pharmacy_distribution.png` | Phase 1 | Top/bottom 10 counties by pharmacies |
| `ownership_types.png` | Phase 1 | Facility ownership distribution pie chart |
| `infrastructure_score_by_county.png` | Phase 2 | Composite scores by county (horizontal bar) |
| `beds_vs_functionality_scatter.png` | Phase 2 | Beds per 10k vs % fully functional |
| `correlation_heatmap.png` | Phase 2 | Correlation matrix of all numeric features |
| `model_comparison.png` | Phase 3 | Bar chart comparing MAE, RMSE, R² across models |
| `feature_importance.png` | Phase 3 | Permutation importance horizontal bar chart |
| `residuals_by_county.png` | Phase 3 | Residuals by county (color-coded by direction) |
| `actual_vs_predicted.png` | Phase 3 | Scatter plot with perfect prediction line |
| `fairness_by_group.png` | Phase 4 | Side-by-side fairness bar charts |
| `mean_residual_by_region.png` | Phase 4 | Mean residual by region (color-coded) |
| `shap_summary.png` | Phase 4 | SHAP summary dot plot |
| `shap_bar.png` | Phase 4 | Mean absolute SHAP bar chart |

## Data Dictionary (Final `county_master.csv`)

| Column | Type | Description |
|--------|------|-------------|
| County | string | County name (standardized, lowercase) |
| Total_beds | integer | Official county-level bed count |
| Total_facilities | integer | All registered facilities in county |
| closed | integer | Count of closed facilities |
| fully-functional | integer | Count of fully functional facilities |
| non-functional | integer | Count of non-functional facilities |
| semi-functional | integer | Count of semi-functional facilities |
| Pct_fully_functional | float | Percentage of facilities fully functional |
| Pct_non_functional | float | Percentage of facilities non-functional |
| Total_pharmacies | integer | All pharmacy records |
| Registered_pharmacies | integer | Pharmacies with registration number |
| Pct_pharmacies_registered | float | Percentage of pharmacies registered |
| Public_facilities | integer | Count of public facilities |
| Public_facility_ratio | float | Proportion of public facilities (0-1) |
| Suspect_zero_bed_count | integer | Inpatient facilities reporting 0 beds |
| Population | integer | 2019 census population |
| Beds_per_10k | float | Beds per 10,000 population |
| Facilities_per_10k | float | Facilities per 10,000 population |
| Pharmacies_per_10k | float | Pharmacies per 10,000 population |
| Infrastructure_score | float | Composite score (0-1) |

## Recommendations for Responsible Use

### Appropriate Uses

- Exploratory analysis and hypothesis generation
- Identifying counties that warrant deeper investigation
- Research on relationships between facility metrics and infrastructure
- Educational use (understanding ML fairness concepts)

### Inappropriate Uses

- Direct resource allocation without human review
- Comparing counties across regions without accounting for bias
- Any use that treats predictions as ground truth
- Policy decisions without community consultation

### Required Safeguards

1. **Human-in-the-loop** County health officers must review all flagged counties
2. **Community validation** Residents of high-residual counties should verify findings
3. **Transparency** Decision-makers must understand model limitations
4. **Regular auditing** Monitor fairness metrics over time
5. **Clear communication** Present predictions with uncertainty bounds

## Notebook Cell Summary

| Notebook | Total Cells | Code Cells | Markdown Cells |
|----------|-------------|------------|----------------|
| 01_data_cleaning_and_exploration | ~45 | ~25 | ~20 |
| 02_merging_and_enrichment | ~50 | ~30 | ~20 |
| 03_modeling | ~25 | ~18 | ~7 |
| 04_fairness_and_explainability | ~18 | ~14 | ~4 |

*Note: Cell counts are approximate; actual numbers may vary slightly based on version.*

## Running the Notebooks

**Run in this exact order** each notebook depends on outputs from the previous one.

```bash
# Complete workflow

# Phase 1: Clean raw data and explore patterns
jupyter notebook notebooks/ata_cleaning_and_exploration.ipynb

# Phase 2: Aggregate to county level and engineer features
jupyter notebook notebooks/merging_and_enrichment.ipynb

# Phase 3: Train and evaluate predictive models
jupyter notebook notebooks/modeling.ipynb

# Phase 4: Audit fairness and explain model decisions
jupyter notebook notebooks/fairness_and_explainability.ipynb
```

## Dependencies

```bash
# Core dependencies for all notebooks
pip install pandas numpy matplotlib seaborn scikit-learn

# Additional for fairness and explainability
pip install shap scipy
```

### requirements.txt

```txt
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
seaborn>=0.12.0
scikit-learn>=1.2.0
shap>=0.40.0
scipy>=1.9.0
```
## Version Information

| Package | Version |
|---------|---------|
| Python | 3.12.7 |
| pandas | 2.2.3 |
| numpy | 1.26.4 |
| scikit-learn | 1.5.1 |
| matplotlib | 3.9.2 |
| seaborn | 0.13.2 |
| shap | 0.46.0 |
| scipy | 1.14.1 |

## Troubleshooting

### SHAP Import Error

If you see NumPy compatibility errors:

```bash
# Downgrade NumPy to compatible version
pip install "numpy<2"
```

### SHAP Memory Issues

If SHAP calculation is slow or crashes:

```python
# In the fairness notebook, reduce the background dataset size
background = shap.sample(X_scaled, 100)  # Use 100 samples instead of all 47
explainer = shap.TreeExplainer(rf_model, background)
```

### Missing Directories

```python
# Create output directories if they don't exist
import os
os.makedirs('../outputs/figures', exist_ok=True)
os.makedirs('../outputs/reports', exist_ok=True)
os.makedirs('../data/processed', exist_ok=True)
```

### Model Performance Concerns

If models are not performing as expected:

1. **Check data completeness**: Verify all 47 counties have complete feature data
2. **Verify scaling**: Ensure StandardScaler was applied before model training
3. **Check LOOCV implementation**: Confirm train/test splits are correct
4. **Review feature engineering**: Ensure per-capita metrics were calculated correctly

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-04-20 | Initial release: Data cleaning, merging, modeling, fairness audit |

---

## Acknowledgments

- Data sourced from [open.africa](https://open.africa/dataset/kenya-master-health-facility-list-2020), a platform making African government data accessible
- Population data from Kenya National Bureau of Statistics (2019 census)
- This research was conducted as part of an HCAI (Human-Centered AI) research project examining health infrastructure equity in Kenya

## References

- Kenya Master Health Facility List (KMHFL): [open.africa](https://open.africa/dataset/kenya-master-health-facility-list-2020)
- Kenya National Bureau of Statistics: 2019 Population Census
- SHAP Documentation: [shap.readthedocs.io](https://shap.readthedocs.io/)
- scikit-learn Documentation: [scikit-learn.org](https://scikit-learn.org/)


## License

**Data License:** The KMHFL data is provided by open.africa under their terms of use. Please refer to open.africa for data licensing information.

