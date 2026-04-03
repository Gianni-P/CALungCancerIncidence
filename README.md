# Machine Learning Identifies Community-Level Factors Associated with Lung Cancer Incidence


An area-level ecological analysis linking environmental pollution, socioeconomic, and behavioral indicators to age-adjusted lung cancer incidence rates (AAIR) across California's 542 Medical Service Study Areas (MSSAs).

## Overview

This repository provides a fully reproducible analytical pipeline that investigates the association between environmental exposures, socioeconomic factors, and lung cancer incidence at the sub-county level in California. The analysis merges three publicly available datasets at the MSSA level, applies a multi-method feature-selection strategy anchored by an autoencoder, and evaluates eight classifiers to discriminate high- from low-incidence areas. All results use associative language; no causal claims are made from ecological data.

## Data Sources

| Dataset | Source | Time Period | Geographic Level |
|---|---|---|---|
| Lung cancer AAIR (per 100 000 person-years, 2000 US Std Pop) | [California Health Maps](https://californiahealthmaps.com) / California Cancer Registry | 2015–2019 | MSSA |
| Pollution & socioeconomic indicators | [CalEnviroScreen 4.0](https://oehha.ca.gov/calenviroscreen) (OEHHA) | ~2017–2019 (pollution); 2015–2019 ACS (socioeconomic) | Census tract → MSSA (population-weighted) |
| Adult smoking prevalence | CDC PLACES Project via [UCSF Health Atlas](https://healthatlas.ucsf.edu) | 2020 | Census tract → MSSA (population-weighted) |

The unit of analysis is the **Medical Service Study Area (MSSA)** (*N* = 542). Census-tract-level indicators are aggregated to MSSAs using population-weighted means (continuous) or population-weighted sums (count-based).

## Repository Structure

```
├── Data_Preparation_Final.ipynb            # Data assembly pipeline (merge, clean, aggregate)
├── AAIR_Publication_Final.ipynb            # Full analytical pipeline (EDA → classification → fairness)
├── healthmaps_with_ces4_pollution_plus_area.csv  # Merged analysis-ready dataset (output of data prep)
├── californiahealthmaps_mssa_all.csv       # Raw California Health Maps cancer data
├── calenviro2025.csv                       # Raw CalEnviroScreen 4.0 indicators
├── health-atlas-2026-03-24-49fe0d25.csv    # Raw UCSF Health Atlas smoking prevalence
├── Medical_Service_Study_Areas.csv         # MSSA geographic boundaries and metadata
├── geography-crosswalk.csv                 # Census tract ↔ MSSA crosswalk table
└── README.md
```

## Analytical Pipeline

The analysis in `AAIR_Publication_Final.ipynb` proceeds through five phases:

1. **Data Assembly & Quality Assessment** — Loading, cleaning, and documenting missingness in the merged dataset. MSSAs with `Sex = 'Both'` are retained (*N* = 542); 21 MSSAs with missing overall AAIR are excluded from outcome-dependent analyses.

2. **Multicollinearity Diagnostics & Feature Selection** — Variance inflation factors, Spearman correlation analysis, and a five-method feature-selection comparison (L1 regularization, mutual information, tree-based importance, correlation pruning, and a permutation-importance autoencoder with bootstrap rank stability).

3. **Binary Classification** — MSSAs are labeled High (AAIR ≥ 60th percentile) or Low (AAIR ≤ 40th percentile). Eight classifiers are evaluated via stratified 5-fold cross-validation: Logistic Regression, Random Forest, Gradient Boosting, Extra Trees, KNN, SVM (RBF), MLP, and XGBoost. The best model is assessed with bootstrap confidence intervals on a held-out test set.

4. **Equity & Fairness Auditing** — Race/ethnicity-stratified sub-cohort evaluation and algorithmic fairness metrics across majority-race MSSA subgroups.

5. **Regression, Sensitivity & Domain Analyses** — Multivariable OLS regression, percentile-cutoff sweep sensitivity analysis, health-outcome predictor exclusion sensitivity, group comparisons with Benjamini–Hochberg FDR correction, and variable-domain comparisons (pollution vs. environmental vs. behavioral).

## Methodological Safeguards

- **No data leakage** — All scaling and feature selection are performed within training folds via `sklearn.pipeline.Pipeline`.
- **Small-area rate instability** — Population-weighted sensitivity analysis; MSSAs with population < 5 000 are flagged.
- **Autoencoder reproducibility** — Full architecture, optimizer, learning rate, regularization, and random seeds are specified.
- **Multiple testing** — Benjamini–Hochberg FDR correction applied to group-comparison p-values.
- **Visualization accessibility** — Okabe–Ito colorblind-safe palette, ≥ 300 DPI, Arial ≥ 8 pt.

## Requirements

The pipeline runs in Python 3 (tested with `ipykernel`). Key dependencies:

- `pandas`, `numpy`
- `scikit-learn`
- `xgboost`
- `torch` (PyTorch — used for the autoencoder feature-selection step)
- `shap`
- `statsmodels`
- `matplotlib`, `seaborn`

The first cell of `AAIR_Publication_Final.ipynb` will `pip install` any missing packages automatically. The output of cell two gives the following package versions:

| Package | Version |
|---|---|
| Python | 3.12.11 |
| NumPy | 2.2.6 |
| pandas | 2.3.0 |
| scikit-learn | 1.7.0 |
| matplotlib | 3.10.3 |
| PyTorch | 2.11.0+cu130 |
| XGBoost | 3.2.0 |
| SHAP | 0.51.0 |
| statsmodels | 0.14.4 |

Random seed: **42**

## Quickstart

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/CALungCancerIncidence.git
   cd CALungCancerIncidence
   ```

2. **Reproduce the merged dataset** (optional — the output CSV is already included):
   ```
   Open and run Data_Preparation_Final.ipynb
   ```

3. **Run the full analysis:**
   ```
   Open and run AAIR_Publication_Final.ipynb
   ```
   All figures and tables are saved to a `publication_outputs/` directory and bundled into a ZIP archive at the end of the notebook.

## Ethical Statement

All data are publicly available, de-identified, and aggregated at the MSSA level. No IRB review was required per 45 CFR 46.104(d)(4) (publicly available data).

## License

Please add a license file to clarify reuse terms.
