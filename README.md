Kenya Health Infrastructure Analysis - Part 1: Data Cleaning & Integration

## Project Overview

This is **Part 1** of a multi-part HCAI (Human-Centered AI) research project investigating whether geographic accessibility to health facilities in Kenya reflects **population needs** or **historical infrastructure bias**.

**Research Questions (Part 1 focuses on data foundation):**
1. What is the current state of Kenya's health facility data quality?
2. How can four disparate datasets be integrated into a unified county-level view?
3. What patterns emerge in the geographic distribution of health infrastructure?

**Later parts will address:**
- Predictive modeling of infrastructure gaps
- Bias detection and equity analysis
- Accessibility modeling and policy recommendations

  ## Repository Structure (Part 1)

```
kenya-health-infrastructure-part1/
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
│       └── county_master.csv   # Final integrated dataset
│
├── notebooks/
│   ├── 01_data_cleaning_and_exploration.ipynb
│   └── 02_merging_and_enrichment.ipynb
│
├── outputs/
│   └── figures/                # Generated visualizations
│       ├── health_unit_status.png
│       ├── county_health_unit_distribution.png
│       ├── beds_per_county.png
│       ├── pharmacy_distribution.png
│       ├── ownership_types.png
│       ├── infrastructure_score_by_county.png
│       ├── beds_vs_functionality_scatter.png
│       └── correlation_heatmap.png
│
├── README.md                   # This file
└── requirements.txt            # Python dependencies
```

## Data Sources

All data was acquired from **[open.africa](https://open.africa)** , a pan-African open data platform that hosts government and institutional datasets.

| Dataset | Description | Rows | Key Columns |
|---------|-------------|------|-------------|
| `health_units.csv` | Master list of health facilities with operational status | 5,558 | Status, Facility, Location hierarchy, Code, Date_established |
| `hospital_beds.csv` | County-level hospital bed counts | 48 | Geography, Number of hospital beds |
| `hospitals_and_beds.csv` | Facility-level bed and operational data | 12,394 | Code, Name, Beds, Owner type, Keph level, Operational status |
| `pharmacies.csv` | Pharmacy registry with regulatory information | 5,219 | Code, Name, Registration_number, Regulatory body, Owner |

**Note:** Raw data files are not included in this repository due to size. Download from open.africa or contact the author.

## Data Cleaning Summary

### 1. Health Units (`01_data_cleaning_and_exploration.ipynb`)

| Issue | Solution |
|-------|----------|
| Status typo: 'non-functinoal' | Corrected to 'non-functional' |
| Date_established as Excel serial numbers | Converted to datetime; values ≤3000 treated as 4-digit years |
| Mixed case in location columns | Standardized to lowercase |
| Duplicate facilities | Removed based on unique Code |

**Key finding:** No missing values in any column - excellent completeness.

### 2. Pharmacies

| Issue | Solution |
|-------|----------|
| 97% missing Registration_number | Added `Is_registered` flag instead of dropping |
| Service_names completely empty | Dropped column |
| Yes/No columns as strings | Converted to boolean (True/False) |
| Mixed case county names | Standardized to lowercase |

**Key finding:** Registration data quality is poor (only 3% registered) - indicates record-keeping shortcomings.

### 3. Hospitals and Beds

| Issue | Solution |
|-------|----------|
| Beds column as string | Converted to numeric |
| 68.8% facilities with 0 beds | Analyzed by facility type |
| Inpatient facilities with 0 beds | Flagged as `Beds_data_suspect` |
| Service_names empty | Dropped column |
| Yes/No columns as strings | Converted to boolean |

**Key finding:** 606 inpatient facilities report zero beds - likely data entry errors, not true zeros.

### 4. County Beds

| Issue | Solution |
|-------|----------|
| 'Kenya' summary row included | Removed for county analysis (total preserved) |
| Mixed case county names | Standardized to lowercase |

**Validation:** County sums matched national total (74,283 beds).

## Merging & Enrichment Summary

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

### Infrastructure Score

Composite score combining three normalized components (equal weight):

```
Infrastructure_score = (Score_beds + Score_functional + Score_public) / 3
```

Where:
- `Score_beds` = normalized Beds_per_10k (0-1 scale)
- `Score_functional` = normalized % fully functional (0-1 scale)
- `Score_public` = Public_facility_ratio (already 0-1)


