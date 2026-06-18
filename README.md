# CALA-ICU-Calibration-Audit

**Calibration drift in ICU mortality prediction across temporal, multi-site, and cross-continental settings: a reproducible external validation and recalibration audit (CALA)**

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/)
[![TRIPOD+AI](https://img.shields.io/badge/reporting-TRIPOD%2BAI-green.svg)](https://www.bmj.com/content/385/bmj-2023-078378)
[![PhysioNet](https://img.shields.io/badge/data-PhysioNet%20credentialed-orange.svg)](https://physionet.org)
[![License: MIT](https://img.shields.io/badge/license-MIT-lightgrey.svg)](LICENSE)
[![SHA-256 Provenance](https://img.shields.io/badge/provenance-SHA--256%20hash%20chain-purple.svg)](#hash-chain-verification)

> **Manuscript:** *Calibration drift in ICU mortality prediction across temporal, multi-site, and cross-continental settings: a reproducible external validation and recalibration audit (CALA)*
> Krutarth Patel, MS · Phanindra Beedala, BSc · Rohan Desai, MS · Harshil Mehta, MBBS

---

## Overview

Clinical AI models trained in one institution or country routinely fail silently when deployed elsewhere — not because they stop ranking patients correctly, but because their **probability estimates become wrong**. This repository contains the complete analytical pipeline for CALA, a unified framework that:

1. Decomposes deployment-time calibration shift into four orthogonal lifecycle stages (internal → temporal → multi-site → cross-continental)
2. Quantifies drift at each stage using four novel metrics: **CDH-50, CDC, SDII, and RHR**
3. Evaluates four post-hoc recalibration strategies across 148,042 ICU stays spanning two continents
4. Anchors end-to-end reproducibility through a closed cryptographic SHA-256 hash chain

### Key Findings

| Metric | Value | Interpretation |
|---|---|---|
| Calibration Decay Half-Life (CDH-50) | Stable (> 100 years) | No detectable temporal calibration decay across MIMIC-IV's 11-year window |
| Continental Drift Coefficient (CDC) | **11.07** (95% CI 7.58–18.03) | Cross-continental shift exceeds within-continent temporal shift by an order of magnitude |
| Subgroup Decay Inequity Index (SDII) — Stage 4 | **4.24** | Subgroup calibration inequity widens 4.2-fold at cross-continental deployment |
| Best Recalibration Half-Life Recovery (RHR) | **0.90** (isotonic, Stage 4) | 90% of cross-continental calibration loss is recoverable without model retraining |

Elderly female patients showed the steepest calibration deterioration at cross-continental deployment (Stage 4 ECE = 0.157, a 3.0-fold increase from the Stage 1 baseline of 0.052).

---

## Databases

This pipeline requires credentialed access to three PhysioNet databases. **Raw data cannot be redistributed.** You must apply for access individually:

| Database | Version | N (stays) | In-hospital mortality | Access |
|---|---|---|---|---|
| [MIMIC-IV](https://physionet.org/content/mimiciv/2.2/) | v2.2 | 40,167 | 10.0% | PhysioNet credentialed |
| [eICU-CRD](https://physionet.org/content/eicu-crd/2.0/) | v2.0 | 93,187 | 9.4% | PhysioNet credentialed |
| [SICdb](https://physionet.org/content/sicdb/1.0.8/) | v1.0.8 | 14,688 | 18.5% | PhysioNet credentialed |

Access requires completion of the [CITI Data or Specimens Only Research](https://about.citiprogram.org/) training and signature of the PhysioNet credentialed health data use agreement.

---

## Repository Structure

```
CALA-ICU-Calibration-Audit/
│
├── CALA_Part1_DataFoundations.ipynb        # Cohort harmonisation, CDM construction
├── CALA_Part2_ModelDevelopment.ipynb       # Imputation, missingness audit, model training, SHAP
├── CALA_Part3_LifecycleAudit.ipynb         # CALA metrics, recalibration, publication figures
│
├── CALA_Output/
│   ├── provenance.json                     # Top-level provenance ledger (Part 1 CDM hash)
│   │
│   ├── tables/
│   │   ├── table1_cohort_characteristics.csv
│   │   ├── table1_cohort_characteristics.md
│   │   └── cross_db_missingness.csv
│   │
│   ├── figures/
│   │   ├── figS1_cdm_variable_availability.png
│   │   └── figS2_cohort_distributions.png
│   │
│   ├── cache/ (Can not share it here as per the Physionet data sharing agreement)
│   │   ├── cdm_unified.parquet 
│   │   ├── eicu_cohort.parquet 
│   │   ├── eicu_features.parquet
│   │   ├── eicu_labs_long.parquet 
│   │   ├── eicu_vitals_long.parquet 
│   │   ├── mimic_cohort.parquet
│   │   ├── mimic_features.parquet 
│   │   ├── mimic_labs_long.parquet 
│   │   ├── mimic_vitals_long.parquet 
│   │   ├── sicdb_cohort.parquet 
│   │   ├── sicdb_events_long.parquet
│   │   ├── sicdb_features.parquet 
│   │
│   ├── part1_artifacts/
│   │   └── manifest.json                  # CDM SHA-256: 2b3eb15e...
│   │
│   ├── part2_artifacts/
│   │   ├── manifest_part2.json            # GBM model SHA-256: 885cb2f8...
│   │   ├── models/
│   │   │   ├── gbm.pkl                    # Frozen LightGBM model
│   │   │   ├── logreg.pkl
│   │   │   ├── median_imputer.pkl         # Training-set-frozen imputer
│   │   │   └── scaler.pkl
│   │   ├── figures/
│   │   │   ├── fig1_consort_flow.png
│   │   │   ├── fig2a_missingness_heatmap.png
│   │   │   ├── fig2b_mdl_auroc.png
│   │   │   ├── fig3_internal_reliability.png
│   │   │   ├── fig4_subgroup_brier_decomposition.png
│   │   │   ├── fig5a_shap_top20.png
│   │   │   ├── fig5b_shap_beeswarm.png
│   │   │   └── fig5c_missingness_reliance.png
│   │   └── tables/
│   │       ├── internal_eval_metrics.csv
│   │       ├── mdl_auroc.csv
│   │       ├── missingness_by_db.csv
│   │       ├── little_mcar_by_db.csv
│   │       ├── shap_importance.csv
│   │       ├── subgroup_brier_decomposition.csv
│   │       ├── split_summary.csv
│   │       ├── gbm_cv_log.csv
│   │       └── imputation_sensitivity.csv
│   │
│   └── part3_artifacts/
│       ├── manifest_part3.json            # Final chain hash: links Part1 CDM + Part2 GBM + Part3 tables
│       ├── figures/
│       │   ├── fig6_reliability_all_stages.png
│       │   ├── fig7_cdh50.png
│       │   ├── fig8_sdii_heatmap.png
│       │   ├── fig9_rhr_bars.png
│       │   ├── fig10_per_hospital_ece.png
│       │   └── fig11_cala_summary.png
│       └── tables/
│           ├── table2_multi_stage_metrics.csv
│           ├── table2_publication.csv
│           ├── table3_cala_metrics.csv
│           ├── table4_recalibration_outcomes.csv
│           ├── recalibration_outcomes.csv
│           ├── rhr_per_method_stage.csv
│           ├── per_hospital_ece.csv
│           ├── discrimination_diagnostic.csv
│           ├── recal_fraction_sensitivity.csv
│           └── hl_nsensitivity.csv
│
└── README.md
```

---

## Notebook Pipeline (Parts 1–3)

The analytical pipeline is split into three sequential Jupyter notebooks. Each notebook reads the frozen outputs of the previous one and writes a manifest recording SHA-256 hashes of its principal artifacts, forming a closed hash chain.

### Part 1 — Data Foundations (`CALA_Part1_DataFoundations.ipynb`)

**What it does:**
- Loads MIMIC-IV, eICU-CRD, and SICdb from local PhysioNet installations using a format-aware, chunked I/O layer (`smart_read`) that handles both folder-nested (MIMIC-IV, eICU) and flat-file (SICdb) layouts
- Applies identical inclusion criteria across all three databases: age ≥ 18, first ICU stay per hospitalisation, ICU LOS ≥ 24 hours, non-missing in-hospital mortality
- Harmonises 19 CDM variables (7 vitals, 12 labs) using integer item-ID lookup (MIMIC-IV), string pattern matching (eICU-CRD), and a LOINC-coded multilingual dictionary (SICdb)
- Computes six aggregations (min, max, mean, last, std, count) per variable over the first 24-hour ICU window, yielding 114 numerical features + 114 binary missingness indicators + demographics = **128 columns**
- Produces supplementary figures: CDM variable availability heatmap (figS1) and cohort distribution plots (figS2)
- Writes `provenance.json` and `manifest.json` containing the CDM SHA-256 hash

**Principal output:** `CALA_Output/part1_artifacts/` including the harmonised CDM parquet and manifest

**CDM SHA-256:** `2b3eb15e1669c5508e9eeaff4bb18d95134e6392a47e56285d85979423d3770d`

---

### Part 2 — Model Development (`CALA_Part2_ModelDevelopment.ipynb`)

**What it does:**
- Performs temporally stratified data partitioning: MIMIC-IV 2008–2013 → development cohort (train 70% / val 15% / test 15%); MIMIC-IV 2014–2019 → temporal hold-out; eICU-CRD and SICdb → withheld entirely
- Fits training-set median imputation and freezes it for all downstream cohorts (zero data leakage)
- Runs the missingness demographic leakage screen: 5-fold stratified logistic regression on 114 missingness indicators alone to predict sex, age < 65, age ≥ 80, with MDL-AUROC > 0.55 flagging significant demographic encoding
- Runs Little's MCAR test across all three databases
- Trains a class-weighted L2 logistic regression baseline and a LightGBM gradient-boosted model with 5-fold CV hyperparameter search (num_leaves ∈ {31, 63}, learning_rate ∈ {0.05, 0.10}, n_estimators ∈ {200, 400})
- Selected GBM configuration: `num_leaves=63, learning_rate=0.05, n_estimators=400`
- Computes SHAP values via TreeExplainer and generates top-20 bar chart, beeswarm plot, and missingness-reliance figure
- Freezes and serialises the GBM model, logistic regression, median imputer, and scaler

**Principal outputs:**
- Frozen GBM: `part2_artifacts/models/gbm.pkl`
- Internal evaluation: AUROC 0.844 (95% CI 0.823–0.866), ECE 0.028, Brier 0.073
- `manifest_part2.json` containing GBM SHA-256

**GBM SHA-256:** `885cb2f8fb6fcf37a3be7ce83ddd3d518ac414fbb0a5a1ebff1fb93b3b002a74`

---

### Part 3 — Lifecycle Audit & CALA Metrics (`CALA_Part3_LifecycleAudit.ipynb`)

**What it does:**
- Evaluates the frozen GBM and logistic regression across all four lifecycle stages with stratified bootstrap 95% CIs (500 iterations): discrimination (AUROC, AUPRC), probabilistic accuracy (Brier), and calibration (ECE at 15 uniform bins, slope, intercept, MCE)
- Flags degenerate models (AUROC < 0.60) via the discrimination diagnostic — logistic regression collapses on eICU-CRD (AUROC 0.554); its apparent post-recalibration ECE is reported as base-rate prediction, not recalibration success
- Computes all four CALA metrics:
  - **CDH-50**: log-linear regression of ECE on time across four MIMIC temporal bins
  - **CDC**: cross-continental ΔECE ÷ max within-continent ΔECE (denominator floored at ε = 0.005)
  - **SDII**: subgroup ECE range ratio across four sex × age strata
  - **RHR**: fractional calibration recovery per recalibration method per stage
- Applies four post-hoc recalibration methods (Platt, isotonic, temperature, beta) at Stages 2–4 using 50/50 stratified splits
- Computes within-eICU per-hospital ECE heterogeneity across 114 eligible hospitals (≥ 200 stays)
- Runs all three pre-specified sensitivity analyses: imputation strategy (MICE vs median), recalibration fit-fraction (30/50/70%), Hosmer–Lemeshow sample-size dependence
- Generates all publication figures (fig6–fig11) and writes `manifest_part3.json` closing the hash chain

**Publication table outputs (exact values):**

*Multi-stage performance (GBM):*

| Stage | n | AUROC | ECE |
|---|---|---|---|
| Stage 1: Internal | 3,288 | 0.844 | 0.028 |
| Stage 2: Temporal | 18,247 | 0.862 | 0.027 |
| Stage 3: Multi-site | 93,187 | 0.784 | 0.037 |
| Stage 4: Cross-continental | 14,688 | 0.720 | 0.117 |

*CALA metrics:*

| Metric | Value | 95% CI |
|---|---|---|
| CDH-50 | Stable (> 100 years) | — |
| CDC | 11.07 | [7.58, 18.03] |
| SDII Stage 2 | 0.60 | — |
| SDII Stage 3 | 0.67 | — |
| SDII Stage 4 | 4.24 | — |
| Best RHR Stage 4 (isotonic) | 0.90 | — |

---

## Hash Chain Verification

The pipeline implements a closed SHA-256 hash chain across all three notebooks. Any post-hoc modification to upstream artifacts is detectable by recomputing hashes.

```
Part 1 → CDM parquet
  SHA-256: 2b3eb15e1669c5508e9eeaff4bb18d95134e6392a47e56285d85979423d3770d
       ↓
Part 2 → GBM model
  SHA-256: 885cb2f8fb6fcf37a3be7ce83ddd3d518ac414fbb0a5a1ebff1fb93b3b002a74
       ↓
Part 3 → Publication tables
  table2 SHA-256: 339aa976c0eae582e8d68b3e0ab4be589847a1e43df86501d4a6e7ddf1a14e65
  table3 SHA-256: 773d06fa6cb7f536e192e4636e54af9d941a5ea3e3d3f25f5602d51be97ef537
```

To verify on your machine after rerunning the pipeline:

```python
import hashlib, json
from pathlib import Path

def sha256_file(path):
    h = hashlib.sha256()
    with open(path, 'rb') as f:
        for chunk in iter(lambda: f.read(65536), b''):
            h.update(chunk)
    return h.hexdigest()

# Load the final manifest and verify the chain
manifest = json.loads(Path('CALA_Output/part3_artifacts/manifest_part3.json').read_text())
chain = manifest['chain']

cdm_hash   = sha256_file('CALA_Output/part1_artifacts/cdm.parquet')  # or your cached parquet path
model_hash = sha256_file('CALA_Output/part2_artifacts/models/gbm.pkl')

assert cdm_hash   == chain['part1_cdm_sha256'], 'CDM hash mismatch — upstream artifact modified'
assert model_hash == chain['part2_gbm_sha256'], 'Model hash mismatch — upstream artifact modified'
print('Hash chain verified.')
```

---

## Installation

### Requirements

- Python 3.10 or 3.11 (tested on 3.11.5)
- Jupyter Notebook or JupyterLab
- ~8 GB RAM minimum; 16 GB recommended for eICU-CRD (93,187 rows with chunked I/O)
- ~5 GB free disk space for cache and outputs

### Setup

```bash
git clone https://github.com/Krutarth007/CALA-ICU-Calibration-Audit.git
cd CALA-ICU-Calibration-Audit

pip install numpy==1.24.4 pandas==2.0.3 scikit-learn==1.4.0 lightgbm==4.1.0 \
            shap==0.44.0 matplotlib seaborn scipy tqdm joblib
```

No PyHealth, MIMIC-Extract, or database-specific frameworks are required. The pipeline uses only the standard scientific Python stack so the code is fully transparent to reviewers.

---

## Reproducing the Results

### Step 1 — Configure data paths

Open `CALA_Part1_DataFoundations.ipynb` and update the `CALAConfig` dataclass to point to your local PhysioNet installations:

```python
@dataclass
class CALAConfig:
    MIMIC_ROOT: Path = Path('/path/to/mimic-iv-2.2')
    EICU_ROOT:  Path = Path('/path/to/eicu-collaborative-research-database-2.0')
    SICDB_ROOT: Path = Path('/path/to/salzburg-intensive-care-database-1.0.8')
    OUTPUT_ROOT: Path = Path('./CALA_Output')
```

The `smart_read` helper automatically handles both the folder-nested layout of MIMIC-IV and eICU (where `admissions.csv` is a folder containing `admissions.csv` or `admissions.csv.gz`) and the flat layout of SICdb.

### Step 2 — Run the notebooks in order

```
Part 1 → Part 2 → Part 3
```

Each notebook checks for cached Parquet artifacts and skips recomputation on subsequent runs. Set `FORCE_RECOMPUTE = True` in `CALAConfig` to force a full rerun.

### Step 3 — Verify the hash chain

Run the hash verification snippet above to confirm your outputs match the published artifacts exactly.

### Expected runtime (first run, no cache)

| Notebook | Approximate time |
|---|---|
| Part 1 (cohort construction) | 20–40 min (eICU-CRD I/O dominates) |
| Part 2 (model development) | 15–30 min (LightGBM CV + SHAP) |
| Part 3 (lifecycle audit) | 10–20 min (bootstrap CIs, recalibration) |

Subsequent runs with cache: < 5 min per notebook.

---

## Outputs Reference

### Figures

| File | Description |
|---|---|
| `figS1_cdm_variable_availability.png` | Cross-database CDM variable availability heatmap (Supplementary) |
| `figS2_cohort_distributions.png` | Age, sex, and mortality distributions by database (Supplementary) |
| `fig1_consort_flow.png` | CONSORT-style cohort flow diagram |
| `fig2a_missingness_heatmap.png` | Per-database missingness rate heatmap |
| `fig2b_mdl_auroc.png` | Missingness demographic leakage AUROC bar chart |
| `fig3_internal_reliability.png` | Reliability diagram — Stage 1 internal |
| `fig4_subgroup_brier_decomposition.png` | Subgroup Brier score decomposition |
| `fig5a_shap_top20.png` | SHAP top-20 feature importance bar chart |
| `fig5b_shap_beeswarm.png` | SHAP beeswarm plot |
| `fig5c_missingness_reliance.png` | Missingness indicator SHAP contribution |
| `fig6_reliability_all_stages.png` | Reliability diagrams across all four lifecycle stages |
| `fig7_cdh50.png` | CDH-50 log-ECE regression plot |
| `fig8_sdii_heatmap.png` | SDII subgroup ECE heatmap across stages |
| `fig9_rhr_bars.png` | RHR bar chart by recalibration method and stage |
| `fig10_per_hospital_ece.png` | Per-hospital ECE distribution within eICU-CRD |
| `fig11_cala_summary.png` | CALA four-metric summary panel |

### Key Tables

| File | Description |
|---|---|
| `table1_cohort_characteristics.csv` | Baseline cohort characteristics across three databases |
| `table2_publication.csv` | AUROC, Brier, ECE across four lifecycle stages (both models) |
| `table3_cala_metrics.csv` | CDH-50, CDC, SDII, RHR headline results |
| `table4_recalibration_outcomes.csv` | ECE pre/post recalibration by method and stage |
| `per_hospital_ece.csv` | Per-hospital ECE for 114 eligible eICU hospitals |
| `discrimination_diagnostic.csv` | Degenerate-model flag table |
| `recal_fraction_sensitivity.csv` | RHR sensitivity to recalibration fit-fraction (30/50/70%) |
| `hl_nsensitivity.csv` | Hosmer–Lemeshow test p-value across nested sample sizes |

---

## CALA Metrics — Quick Reference

### CDH-50 (Calibration Decay Half-Life)
The temporal horizon (in years) at which a model's ECE would double relative to its development baseline, estimated by fitting `log ECE(t) = β₀ + β₁(t − t₀)`. When β₁ ≤ 0 (no detectable decay), CDH-50 is reported as stable (> 100 years).

### CDC (Continental Drift Coefficient)
`CDC = (ECE_Stage4 − ECE_Stage1) / max(ECE_temporal_max − ECE_Stage1, ε)` where ε = 0.005 floors the denominator when within-continent shift is negligible. CDC > 1 indicates cross-continental shift exceeds temporal shift; CDC > 10 indicates an order-of-magnitude difference.

### SDII (Subgroup Decay Inequity Index)
For four demographic strata (Female < 65, Female ≥ 65, Male < 65, Male ≥ 65): `SDII_s = (max_g ECE_{g,s} − min_g ECE_{g,s}) / (max_g ECE_{g,1} − min_g ECE_{g,1})`. SDII > 1 = widening inequity; SDII < 1 = narrowing inequity relative to internal baseline.

### RHR (Recalibration Half-Life Recovery)
`RHR_{m,s} = 1 − ECE_post / ECE_pre` where m is the recalibration method and s is the lifecycle stage. RHR = 1 denotes complete recovery; RHR = 0 denotes no improvement. Note: RHR must be interpreted alongside AUROC — high RHR on a degenerate model (AUROC < 0.60) reflects base-rate prediction, not rescue.

---

## Governance Thresholds (Proposed Operational Heuristics)

These thresholds are proposed heuristics derived from the CALA analysis and should be treated as starting points for prospective validation, not prescriptive clinical standards:

| Trigger | Threshold | Action |
|---|---|---|
| Subgroup calibration error | ECE > 0.05 in any demographic subgroup | Mandatory recalibration before continued clinical use |
| Subgroup inequity widening | SDII > 2.0 | Subgroup-specific recalibration |
| Cross-jurisdictional deployment | Anticipated CDC > 3 | Isotonic or beta recalibration on a target-site validation cohort prior to deployment |

Within-continent monitoring cadence: quarterly. Cross-continental deployments: monthly until a local baseline is established.

---

## Data Restrictions

In accordance with PhysioNet data use agreements:
- Raw patient-level data **cannot** be redistributed in any form
- The harmonised CDM parquet and trained model artifacts in `CALA_Output/` are derived outputs, not raw data; their redistribution is governed by the terms of each database's data use agreement
- Access to the underlying databases must be obtained individually through [physionet.org](https://physionet.org)

---

## Reporting Standards

This study was conducted in accordance with the **TRIPOD+AI** statement (Collins et al., BMJ 2024), with risk-of-bias considerations informed by **PROBAST-AI** guidance. The IJMEDI machine learning checklist is provided as Supplementary Material S1 in the manuscript.

---

## Citation

If you use this framework, code, or the CALA metrics in your work, please cite:

```bibtex
@article{patel2026cala,
  title   = {The Clinical {AI} Lifecycle Audit ({CALA}): A Framework for Quantifying
             Calibration Drift in {ICU} Mortality Prediction Across Temporal,
             Multi-Site, and Cross-Continental Settings},
  author  = {Krutarth Patel MS, Phanindra Beedala BSc, Rohan Desai MS, Harshil Mehta MBBS},
  journal = {},
  year    = {2026},
  note    = {}
}
```

---

## Related Preprints

- Patel K, Beedala P. *Calibration drift under cross-institutional deployment: an external validation framework for ICU mortality prediction across MIMIC-IV and eICU.* medRxiv. 2026. https://doi.org/10.64898/2026.05.03.26352335v1

- Patel K, Beedala P. *Unmeasured but not unbiased: the Missingness Demographic Leakage Audit (MDLA) for calibration-aware fairness evaluation in critical care mortality prediction.* medRxiv. 2026. https://doi.org/10.64898/2026.05.01.26352193v1

---

## Authors

**Krutarth Patel, MS**
Senior Cloud Solutions Engineer, Department of Interoperability, Humana Inc., Louisville, KY, USA
ORCID: [0009-0002-8748-8098](https://orcid.org/0009-0002-8748-8098)
Email: krutarthpatel1997@gmail.com

**Phanindra Beedala, BSc**
Independent Researcher, Louisville, KY, USA

**Rohan Desai, MS** 
R1RCM, Dallas, TX, USA, ORCID: [0009-0002-6119-9482]

**Harshil Mehta, MBBS**
Health and Wellness Center, Waghwala, India

## License

The code in this repository is released under the [MIT License](LICENSE). Data artifacts derived from PhysioNet databases are subject to the terms of the respective PhysioNet data use agreements and are not covered by this license.

---

## Acknowledgements

The authors acknowledge the MIT Laboratory for Computational Physiology for MIMIC-IV; Philips Healthcare and the MIT Critical Data group for eICU-CRD; and Drs. Niklas Rodemund and colleagues at Salzburg University Hospital for SICdb. We thank PhysioNet for credentialed access infrastructure and the maintainers of scikit-learn, LightGBM, SHAP, pandas, NumPy, and matplotlib.

---
