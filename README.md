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



