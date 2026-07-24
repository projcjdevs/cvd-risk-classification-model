# PH-Aligned, Lifestyle-Augmented CVD Risk Model

A supervised learning project exploring whether lifestyle features (physical activity, diet, alcohol use,
self-rated health) add predictive value for cardiovascular disease risk beyond the standard clinical variables
used in equation-based tools like Globorisk and PhilPEN.

This project is a data-driven complement to
[`cvd-risk-scorer`](https://github.com/projcjdevs/cvd-risk-scorer) — an existing Globorisk/PhilPEN-based,
PH-calibrated CVD risk scorer. That tool is equation-based and deterministic (a fixed formula with published
coefficients); this project instead trains a model on real survey data with a broader feature set, and benchmarks
its output against that equation.

**Educational / portfolio project. Not clinically validated. Not for medical use.**

## Motivation

Cardiovascular disease is the leading cause of death in the Philippines, driven largely by modifiable lifestyle
risk factors (smoking, inactivity, poor diet, alcohol use). Clinical risk equations like Globorisk are simple and
validated by design, but structurally limited to a narrow set of inputs (age, sex, BP, smoking, BMI/cholesterol,
diabetes). This project asks: does a supervised model trained on richer lifestyle indicators pick up meaningful
signal that a fixed clinical equation cannot?

## Repository Structure

```
├── notebooks/
│   ├── 00_project_overview.ipynb              # Motivation, literature review, framework
│   ├── 01_data_understanding_and_cleaning.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_model_training.ipynb
│   ├── 04_model_validation.ipynb
│   ├── 05_ph_calibration.ipynb
│   └── 06_globorisk_benchmark_and_results.ipynb
├── data/
│   ├── raw/                                    # Original downloaded CSV (not committed)
│   └── processed/                              # Cleaned / feature-engineered outputs per sprint
└── README.md
```

Each notebook is a separate kernel — there is no shared memory between them. Every notebook re-imports its own
libraries and loads its input from `data/processed/`, which is written by the previous notebook in the sequence.

## Dataset

**CDC BRFSS Heart Disease Health Indicators Dataset** (2015 Behavioral Risk Factor Surveillance System, cleaned)
- Source: https://www.kaggle.com/datasets/alexteboul/heart-disease-health-indicators-dataset
- 253,680 responses, 21 features, binary target `HeartDiseaseorAttack`
- ~9.4% positive class (significant imbalance)
- Download the CSV from Kaggle and place it at `data/raw/heart_disease_health_indicators_BRFSS2015.csv`

**Known limitation:** this is a US population survey. No public, row-level Filipino clinical + lifestyle dataset
currently exists (the National Nutrition Survey is restricted-access; the Philippines has not run a general adult
WHO STEPS survey). This is addressed via feature-level and calibration-level PH alignment — see below — not by
substituting or merging in an unrelated dataset.

## Methodology Summary

1. **Data cleaning** — duplicate/null checks, value-range sanity checks
2. **Feature engineering** — composite lifestyle risk score, comorbidity count, PH-aligned BMI categories
   (using Filipino/Asian-Pacific obesity cutoffs rather than WHO global defaults)
3. **Model training** — Logistic Regression, Random Forest, and LightGBM baselines with class-imbalance handling
4. **Validation** — ROC-AUC, PR-AUC, F1, recall, stratified cross-validation, feature importance
5. **PH calibration** — recalibrating predicted probabilities using published Philippine prevalence statistics
   (hypertension, diabetes, smoking, obesity), mirroring how Globorisk itself is calibrated per-country using
   incidence statistics rather than retrained on local patient microdata
6. **Globorisk benchmark** — running synthetic test patients from the `cvd-risk-scorer` repo through the trained
   model and comparing outputs against the Globorisk-PH tier for the same profiles

## Setup

```bash
pip install pandas numpy scikit-learn lightgbm matplotlib seaborn
```

Run notebooks in numeric order (`00` → `06`); each depends on the processed output of the previous one.

## References

- Globorisk CVD Risk Score — https://www.globorisk.org/
- PhilPEN Protocol, Department of Health — https://doh.gov.ph/sites/default/files/publications/PhilPEN%20Protocol.pdf
- WHO HEARTS Technical Package — https://www.who.int/teams/noncommunicable-diseases/cardiovascular-diseases/management/tools/hearts
- `cvd-risk-scorer` repository — https://github.com/projcjdevs/cvd-risk-scorer
- WHO NCD Microdata Repository (STEPS surveys) — https://extranet.who.int/ncdsmicrodata/index.php/home
- CDC BRFSS Heart Disease Health Indicators Dataset (Kaggle) — https://www.kaggle.com/datasets/alexteboul/heart-disease-health-indicators-dataset
- Cardiometabolic risk profile of young adults with diabetes in the Philippines (8th NNS) — https://pmc.ncbi.nlm.nih.gov/articles/PMC8214347/
- Risk factor contributions to socioeconomic inequality in cardiovascular risk in the Philippines — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10092926/
- Dietary pattern and nutrient intakes in association with NCD risk factors among Filipino adults — https://pmc.ncbi.nlm.nih.gov/articles/PMC7397579/

## License

Educational/portfolio use. Not a clinical or diagnostic tool.
