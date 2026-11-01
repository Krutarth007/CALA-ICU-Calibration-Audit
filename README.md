# CALA-ICU-Calibration-Audit

**Special Topic AI in CDS: A Multi-Database Calibration and Recalibration Audit for Safe Deployment of Imported ICU Mortality Models (CALA)**

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3115/)
[![TRIPOD+AI](https://img.shields.io/badge/reporting-TRIPOD%2BAI-green.svg)](https://www.bmj.com/content/385/bmj-2023-078378)
[![PhysioNet](https://img.shields.io/badge/data-PhysioNet%20credentialed-orange.svg)](https://physionet.org)
[![License: MIT](https://img.shields.io/badge/license-MIT-lightgrey.svg)](LICENSE)
[![SHA-256 Provenance](https://img.shields.io/badge/provenance-SHA--256%20hash%20chain-purple.svg)](#hash-chain-verification)
[![Revision](https://img.shields.io/badge/version-R1%20(major%20revision)-red.svg)](#what-changed-in-revision-r1)

> **Manuscript:** *Special Topic AI in CDS: A Multi-Database Calibration and Recalibration Audit for Safe Deployment of Imported ICU Mortality Models (CALA)*
> XXX · XXX · XXX · XXX
>
> **This is the code and results for revision R1.** All numbers below match the revised manuscript. The findings of the original submission were affected by data-handling errors that are corrected here (see [What changed in revision R1](#what-changed-in-revision-r1)).

---

## Overview

CALA (Clinical AI Lifecycle Audit) is a reproducible audit of what happens to an imported ICU mortality model when a health system deploys it unchanged as a clinical decision support (CDS) input. A gradient-boosted model predicting in-hospital mortality 24 hours after ICU admission is trained once on MIMIC-IV, frozen, and evaluated at four deployment stages:

| Stage | Question a governance committee asks | Cohort | n |
|---|---|---|---|
| 1. Internal | Does it work where it was built? | MIMIC-IV 2008-2013 test set | 3,288 |
| 2. Temporal | Does it still work there years later? | MIMIC-IV 2014-2019 | 18,247 |
| 3. US multi-site | Does it work at other US hospitals? | eICU-CRD, 208 hospitals | 93,187 |
| 4. External transport | Does it work at a single center in another country? | SICdb, Salzburg, Austria | 12,302 |

The pipeline:

1. Harmonizes three structurally different ICU databases into one 19-variable Common Data Model (CDM), with harmonized outcome definitions, data sources, units, sampling frequency and time anchors.
2. Measures discrimination and calibration at each stage with conventional measures (calibration intercept and slope, ECE with its value expected under perfect calibration, ICI) and four descriptive summary measures (CDH-50, CDC, SDII, RHR), each with uncertainty.
3. Tests six post-hoc recalibration methods over 200 repeated splits, with patient-level bootstrap intervals and small local fitting samples (100 to 4,000 stays).
4. Simulates how many local stays a receiving site needs to detect a calibration shift, for the observed shift and for known shifts of 0.1 to 0.5.
5. Links data, model and results through a SHA-256 hash chain, so any change to an upstream artifact is detectable.

The model is a **24-hour landmark model** for the 24-hour reassessment (senior review, escalation planning, goals-of-care conversations), not an admission-time triage tool.

### Key findings (revision R1)

| Finding | Result |
|---|---|
| Discrimination, Stages 1 to 4 | AUROC 0.844, 0.862, 0.802, 0.796 |
| Calibration over time (development site) | No deterioration: ECE 0.028 and 0.027; β₁ = -0.034 ln-ECE per year (95% CI -0.078 to -0.003), so CDH-50 is not estimable |
| Pooled across 208 US hospitals | Similar to internal: ECE 0.027 (95% CI 0.026 to 0.029), intercept 0.104 |
| Single European center (SICdb) | Under-prediction: ECE 0.043 (0.040 to 0.048), intercept 0.769 (0.675 to 0.845); mean predicted 5.9% vs observed 9.5% |
| Where the European error sits | Mainly high-level ICUs (CWIN 14.5% observed vs 8.8% predicted; INBD 16.0% vs 8.7%); lower-level units close to calibrated (3.5% vs 3.3%) |
| Calibration slope | Below 1 at every stage (0.602 to 0.699): predictions too extreme, even internally |
| Recalibration, Stage 4 | Logistic recalibration ECE 0.044 → 0.010 (RHR 0.71, patient-level bootstrap 95% CI 0.56 to 0.82); intercept-only updating achieved only 17% of the reduction |
| Small local samples | Logistic recalibration fitted on 250 stays gave median Stage 4 ECE 0.019 |
| Local audit size | 250 / 500 stays detected the Stage 4 shift in 79% / 95% of simulated audits, but a known shift of 0.2 in only 14% / 22% |
| Data handling | Corrected outcome definitions and extraction reduced the apparent Stage 4 ECE from 0.117 (original submission) to 0.043 |

**Interpretation.** Calibration changed with transport, not with time. Because institution, case mix, period, documentation, database structure and country all differ between the databases, and Stage 4 is a single center, these results describe transport to the evaluated cohorts. They do **not** show that geography or jurisdiction dominates time.

---

## What changed in revision R1

| Area | Original submission | Revision R1 |
|---|---|---|
| SICdb outcome | Any recorded death (`OffsetOfDeath`), which includes registry deaths up to one year after admission: 18.5% "in-hospital" mortality | Death on or before the hospital discharge day; reproduces the published full-database in-hospital rate (7.9%); cohort in-hospital mortality 9.5% |
| SICdb time anchor | Feature window and 24-hour stay rule counted from admission to the documentation system | Counted from ICU admission (`ICUOffset`) |
| SICdb first stay | Chosen by `CaseID`, which is randomly assigned | Earliest admission (`OffsetAfterFirstAdmission`) |
| SICdb laboratory codes | Keyword matching (included urine values, hemoglobin fractions, creatinine clearance and others) | Explicit list of equivalent blood measurements; urea converted to BUN |
| SICdb ICU type | `cases.HospitalUnit` (last unit of the case) | Unit in `unitlog` at 24 hours after ICU admission |
| eICU-CRD vital signs | Invasive pressure and `vitalPeriodic` temperature only; 5-minute counts | Adds `vitalAperiodic` (non-invasive pressure) and `nurseCharting` (temperature); values averaged hourly |
| SICdb cohort | 14,688 stays (total 148,042) | 12,302 stays (total 145,656) |
| Calibration measures | ECE (15 bins) | Adds intercept, slope, ICI, alternative binnings, ECE expected under perfect calibration |
| Recalibration | 4 methods, single 50/50 split | 6 methods (adds intercept-only and logistic), 200 repeated splits, patient-level bootstrap, fit sizes 100 to 4,000 |
| CDC denominator | Largest single calendar period (a small-sample fluctuation) | Stage 2 change, floored; the absolute difference ΔECE₄ − ΔECE₂ is the primary contrast |
| Governance | ECE > 0.05 and SDII > 2 triggers; quarterly/monthly monitoring | Candidate intercept and slope checks, framed as hypotheses for prospective evaluation; no monitoring schedule |
| Code/manuscript | 5-fold CV stated; class weighting not stated | 3-fold CV with early stopping; class weighting (`scale_pos_weight` 9.24) disclosed; refit on training + validation |

The frozen model itself did not change: its SHA-256 is identical to the original submission (`885cb2f8…`). Only the data fed to it, the outcome definitions and the analyses changed.

---

## Databases

This pipeline requires credentialed access to three PhysioNet databases. **Raw data cannot be redistributed.** You must apply for access individually:

| Database | Version | N (stays) | In-hospital mortality | Access |
|---|---|---|---|---|
| [MIMIC-IV](https://physionet.org/content/mimiciv/2.2/) | v2.2 | 40,167 | 10.0% | PhysioNet credentialed |
| [eICU-CRD](https://physionet.org/content/eicu-crd/2.0/) | v2.0 | 93,187 | 9.4% | PhysioNet credentialed |
| [SICdb](https://physionet.org/content/sicdb/1.0.8/) | v1.0.8 | 12,302 | 9.5% | PhysioNet credentialed |

Access requires completion of the [CITI Data or Specimens Only Research](https://about.citiprogram.org/) training and signature of the PhysioNet credentialed health data use agreement.

---

## Repository Structure

```
CALA-ICU-Calibration-Audit/
│
├── CALA_Combined_R1.ipynb                 # Parts 1-3 + Section 3.16 (revision analyses), with outputs
│
├── CALA_Output/
│   ├── provenance.json                     # Top-level provenance ledger (CDM hash, counts, versions)
│   │
│   ├── tables/                             # Part 1 tables
│   │   ├── table1_cohort_characteristics.csv / .md
│   │   ├── cross_db_missingness.csv
│   │   ├── R1_sicdb_outcome_rules.csv       # candidate SICdb in-hospital death rules
│   │   ├── R1_sicdb_lab_codes_units.csv     # SICdb laboratory codes used, units, conversions
│   │   ├── R1_eicu_vital_source_tables.csv  # eICU-CRD vital-sign sources
│   │   ├── R1_eicu_raw_counts_before_hourly.csv
│   │   └── R1_unit_qc_by_database.csv       # cross-database unit check
│   │
│   ├── figures/                            # Part 1 diagnostic figures
│   │
│   ├── cache/                              # NOT shared (patient-level, PhysioNet DUA)
│   │
│   ├── part1_artifacts/
│   │   └── manifest.json                   # CDM SHA-256: 9f4e3628...
│   │
│   ├── part2_artifacts/
│   │   ├── manifest_part2.json             # GBM SHA-256: 885cb2f8...
│   │   ├── models/                         # frozen LightGBM, logistic regression, imputer, scaler, column list
│   │   ├── figures/
│   │   └── tables/                         # internal metrics, MDL-AUROC, SHAP, CV log, imputation check
│   │
│   ├── part3_artifacts/                    # original Part 3 code on the corrected data (kept for continuity)
│   │   ├── manifest_part3.json             # closes the original hash chain
│   │   ├── figures/
│   │   └── tables/
│   │
│   └── revision_r1/                        # ALL RESULTS REPORTED IN THE REVISED PAPER
│       ├── manifest_r1.json                # CDM + GBM hashes, settings, versions, SHA-256 of every table
│       ├── revision_numbers.json           # every number quoted in the manuscript and response letter
│       ├── Supplementary_Material_R1.docx  # supplement built from the tables below
│       ├── figures/                        # Figs. 1 to 6 and Supplementary Figs. S1 to S3 (PNG + PDF, 300 dpi)
│       └── tables/                         # one CSV per main or supplementary table (see Outputs Reference)
│
├── LICENSE
└── README.md
```

---

## Notebook Pipeline

The analysis is a single notebook, `CALA_Combined_R1b.ipynb`, in three parts plus the revision section. Each part reads the frozen outputs of the previous one and writes a manifest of SHA-256 hashes, forming a closed hash chain.

### Part 1: Data Foundations (Sections 1.1 to 1.13)

- Loads MIMIC-IV, eICU-CRD and SICdb from local PhysioNet installations with a chunked, format-aware reader (`smart_read`).
- Applies identical inclusion criteria: age 18 or older, **first ICU stay per patient**, ICU stay of at least 24 hours, non-missing in-hospital outcome.
- SICdb: ICU admission from `ICUOffset`; all vital-sign and laboratory offsets re-expressed relative to it; first admission by `OffsetAfterFirstAdmission`; explicit laboratory whitelist (`SICDB_LAB_WHITELIST`); urea converted to BUN; ICU type from `unitlog` at 24 hours.
- eICU-CRD: vital signs from `vitalPeriodic`, `vitalAperiodic` and `nurseCharting`, averaged within each hour.
- Outcome: `hospital_expire_flag` (MIMIC-IV), discharge status "Expired" (eICU-CRD), death on or before the hospital discharge day (SICdb); the SICdb rule is selected by evaluating candidate rules on the full database.
- Six first-24-hour summaries per variable (min, max, mean, SD, last, count): 114 features + 114 missingness indicators.

**CDM SHA-256:** `9f4e362835235be3e9ae728e7f72d94f1023fed017119dadebf17e5f7c888fa4` (145,656 stays × 129 columns)

### Part 2: Model Development (Sections 2.1 to 2.12)

- Temporal partitioning: MIMIC-IV 2008-2013 → training (n = 15,344), internal validation (3,288), test (3,288); MIMIC-IV 2014-2019 → temporal hold-out; eICU-CRD and SICdb withheld entirely.
- Training-set median imputation, frozen for all cohorts; missingness indicators kept as inputs.
- Missingness demographic leakage screen: logistic regression on the 114 indicators alone predicts sex, age < 65 and age ≥ 80 (5-fold CV); MDL-AUROC > 0.55 is a sensitive screen, compared with 20 permutations of each attribute.
- LightGBM with positive-class weighting (`scale_pos_weight` = 9.24), selected by 3-fold stratified CV with early stopping over four configurations; selected `num_leaves=63, learning_rate=0.05, n_estimators=400`; refit on training + validation (n = 18,632) and frozen. Seed 42.
- Comparators: class-weighted (original) and unweighted logistic regression.
- Section 2.8b (revision): sensitivity models without count features, without counts and indicators, unweighted, with age and sex, and a regularized unweighted reference GBM.
- SHAP TreeExplainer on the internal test set.

**Internal performance (Stage 1):** AUROC 0.844 (95% CI 0.823 to 0.866), ECE 0.028 (0.012 expected under perfect calibration), intercept -0.113, slope 0.675, Brier 0.073.

**GBM SHA-256:** `885cb2f8fb6fcf37a3be7ce83ddd3d518ac414fbb0a5a1ebff1fb93b3b002a74` (unchanged since the original submission)

### Part 3: Lifecycle Audit (Sections 3.1 to 3.15)

The original Part 3 code, run on the corrected data. It is kept so the hash chain and the original specification remain inspectable. Note that its outputs use the **original** specification (four recalibration methods, single split, original CDC denominator, and a "stable (>100 years)" label for CDH-50); the revised paper does not use these labels and reports Section 3.16 instead.

### Section 3.16: Revision R1 analyses (reported in the paper)

| Cell group | What it computes |
|---|---|
| Stage frames | Predictions of the frozen GBM and comparators at all four stages |
| Stage-level metrics | AUROC, AUPRC, Brier, ECE (10/15/20 equal-width, 15 equal-mass), ECE expected under perfect calibration (500 simulations), ICI, E50, E90, intercept, slope, MCE; bootstrap 95% CIs (500 replicates) |
| Summary measures | Temporal slope β₁ and CDH-50; CDC with three floors and three denominators; absolute ΔECE; SDII (raw, null-corrected, ICI); joint bootstrap resampling every component |
| Subgroups | Sex × age strata: n, deaths, observed vs predicted, ECE with CI and null, intercept |
| Recalibration | Six methods × Stages 2 to 4 × 200 repeated stratified 50/50 splits; fit-size curve (100 to 4,000 stays, 200 draws each); subgroup metrics after recalibration |
| Cell D | Patient-level bootstrap of RHR (200 replicates, recalibrators refitted) |
| Decision analysis | Net benefit (2% to 50%); flagged share, sensitivity and missed deaths at 10/20/30% |
| Hospitals | Per-hospital ECE and intercept for 114 eICU-CRD hospitals; SICdb benchmarked at the median hospital size |
| ICU type, case mix | Calibration by unit type; domain-classifier reweighting to the development case mix |
| Local audits | Simulated audits at 100 to 4,000 stays (300 per size); Cell E: known intercept shifts of 0.1 to 0.5 imposed on Stage 2 |
| Outputs | Tables, figures, `revision_numbers.json`, `manifest_r1.json`, `Supplementary_Material_R1.docx` |

---

## Hash Chain Verification

```
Part 1 → CDM parquet
  SHA-256: 9f4e362835235be3e9ae728e7f72d94f1023fed017119dadebf17e5f7c888fa4
       ↓
Part 2 → frozen GBM
  SHA-256: 885cb2f8fb6fcf37a3be7ce83ddd3d518ac414fbb0a5a1ebff1fb93b3b002a74
       ↓
Part 3 → original-specification tables
  table2 SHA-256: a7836aab715891b9aa7884dba572ad9fc303fde823c69d18c6201d9a6d2deead
  table3 SHA-256: 596197b3c944c0845a2b22b48c2db103eb0e8eeb700582e777705f65f83fe9a5
       ↓
Section 3.16 → revision tables
  manifest_r1.json: CDM and GBM hashes + SHA-256 of all 34 revision tables
```

Anyone can check that the published revision tables are the ones the notebook wrote, without data access:

```python
import hashlib, json
from pathlib import Path

def sha256_file(path):
    h = hashlib.sha256()
    with open(path, 'rb') as f:
        for chunk in iter(lambda: f.read(65536), b''):
            h.update(chunk)
    return h.hexdigest()

root = Path('CALA_Output/revision_r1')
manifest = json.loads((root / 'manifest_r1.json').read_text())
bad = [name for name, h in manifest['tables_sha256'].items()
       if sha256_file(root / 'tables' / name) != h]
print('All revision tables match the manifest.' if not bad else f'Mismatch: {bad}')
```

After rerunning the pipeline with your own PhysioNet access, the CDM and model hashes can be checked against `part3_artifacts/manifest_part3.json` (`chain.part1_cdm_sha256`, `chain.part2_gbm_sha256`).

---

## Installation

### Requirements

- Python 3.11 (tested on 3.11.5)
- Jupyter Notebook or JupyterLab
- 16 GB RAM recommended (eICU-CRD and SICdb are streamed in chunks)
- About 5 GB free disk space for cache and outputs

### Setup

```bash
git clone <this repository URL>
cd CALA-ICU-Calibration-Audit

pip install numpy==1.24.4 pandas==2.3.3 scikit-learn==1.3.0 lightgbm==4.6.0 shap==0.42.1 \
            scipy matplotlib seaborn tqdm joblib pyarrow tabulate python-docx
```

Versions are those recorded in `manifest_r1.json`. `tabulate` is needed for the Markdown version of Table 1, and `python-docx` for the supplement. No PyHealth, MIMIC-Extract or database-specific framework is required.

---

## Reproducing the Results

### Step 1: Configure paths

In Section 1.3, set the PhysioNet locations in `CALAConfig`, and set the same `OUTPUT_ROOT` in `CALA2Config` (Section 2.1) and `CALA3Config` (Section 3.1):

```python
@dataclass
class CALAConfig:
    MIMIC_ROOT:  Path = Path('/path/to/mimic-iv-2.2')
    EICU_ROOT:   Path = Path('/path/to/eicu-collaborative-research-database-2.0')
    SICDB_ROOT:  Path = Path('/path/to/salzburg-intensive-care-database-1.0.8')
    OUTPUT_ROOT: Path = Path('./CALA_Output')
    FORCE_RECOMPUTE: bool = False
```

`CALA3Config.ORIGINAL_OUTPUT_ROOT` is optional. It points to a copy of the original-submission outputs and is used only for the "original versus corrected" comparison (Supplementary Table S4); if absent, that table uses the values reported in the original manuscript.

### Step 2: Run all cells

Restart the kernel and run all cells in order. Cached Parquet files are reused on later runs; set `FORCE_RECOMPUTE = True` to rebuild them.

Approximate time on a 16 GB laptop: 30 to 60 minutes for first-time extraction (eICU-CRD and SICdb I/O dominate), then about 3 hours for Parts 2 and 3 and Section 3.16 (the 200 repeated splits, bootstraps and audit simulations dominate).

### Step 3: Check against the paper

Your run should reproduce, for example: cohort sizes 40,167 / 93,187 / 12,302; Stage 4 AUROC 0.796, ECE 0.043, intercept 0.769; Stage 1 AUROC 0.844. The headline numbers are in `revision_r1/revision_numbers.json`, and every number in the manuscript can be traced to a table in `revision_r1/tables/`.

---

## Outputs Reference

### Main figures (`CALA_Output/revision_r1/figures/`)

| Manuscript | File |
|---|---|
| Fig. 1, cohort flow | `Fig1_cohort_flow.png` |
| Fig. 2, SHAP feature importance | `Fig2A_shap_top20.png` |
| Fig. 3A, missingness by database | `Fig3A_missingness.png` |
| Fig. 3B, missingness demographic leakage | `Fig3B_mdl_auroc.png` |
| Fig. 4, reliability by stage | `Fig4_reliability_by_stage.png` |
| Fig. 5, calibration by stage, over time, by subgroup, after recalibration | `Fig5_summary.png` |
| Fig. 6, per-hospital calibration within eICU-CRD | `Fig2B_per_hospital_ece.png` |
| Supplementary Fig. S1, recalibration by local fitting size | `FigS2_recalibration_fit_size.png` |
| Supplementary Fig. S2, decision curves | `FigS1_decision_curves.png` |
| Supplementary Fig. S3, simulated local audits | `FigS3_local_audit_sample_size.png` |

File names follow the notebook; the manuscript numbers figures in order of first citation, hence the differences above.

### Tables (`CALA_Output/revision_r1/tables/` unless stated)

| Manuscript | File(s) |
|---|---|
| Table 1 | `CALA_Output/tables/table1_cohort_characteristics.csv` |
| Table 2 | `T2_stage_metrics_GBM.csv`, `T2_stage_metrics_LR_unweighted.csv` |
| Table 3 | `T3_mdl_auroc_permutation.csv` |
| Table 4 | `T4_delta_vs_stage1.csv`, `T4_cdc_sensitivity.csv`, `T4_sdii_bootstrap.csv` |
| Table 5 | `T5_subgroup_metrics.csv` |
| Table 6 | `T6_recalibration_repeated_splits.csv` |
| S1 | TRIPOD+AI checklist (in the supplement) |
| S2a, S2b | `S_sicdb_outcome_rules.csv`, `T1_outcomes_by_database.csv` |
| S3a to S3d | `S_eicu_vital_source_tables.csv`, `S_eicu_raw_counts_before_hourly.csv`, `S_sicdb_lab_codes_units.csv`, `S_unit_qc_by_database.csv` |
| S4 | `S_original_vs_corrected.csv` |
| S5 | `T2_stage_metrics_GBM.csv`, `T2_stage_metrics_LR_unweighted.csv`, `T2_stage_metrics_LR_weighted.csv` (discrimination diagnostic) |
| S6 | `T3_mdl_auroc_permutation.csv` |
| S7 | `T2_stage_metrics_GBM.csv` (alternative binnings, ICI, E50, E90) |
| S8 | `T4_cdc_sensitivity.csv` |
| S9a, S9b | `T4_sdii_bootstrap.csv`, `T5_subgroup_delta_bootstrap.csv` |
| S10 | `S_calibration_by_icu_type.csv` |
| S11 | `S_casemix_reweighting.csv` |
| S12 | `T6_recalibration_repeated_splits.csv`, `S_recalibration_LR_unweighted.csv`, `S_recalibration_LR_weighted.csv` |
| S13 | `S_rhr_patient_bootstrap.csv` |
| S14 | `S_recalibration_fit_size.csv` |
| S15 | `S_recalibration_by_subgroup.csv` |
| S16 | `S_threshold_analysis.csv` (decision curves: `S_decision_curve.csv`) |
| S17 | `S_per_hospital_summary.csv`, `S_per_hospital_ece.csv` |
| S18 | `CALA_Output/part2_artifacts/tables/imputation_sensitivity.csv` |
| S19 | `CALA_Output/part3_artifacts/tables/hl_nsensitivity.csv` |
| S20 | `S_sensitivity_models.csv` |
| S21 | `S_reference_model_regularised.csv` |
| S22 | `S_local_audit_sample_size.csv` |
| S23 | `S_local_audit_controlled_shifts.csv` |

---

## Calibration Measures: Quick Reference

Conventional measures come first in the paper; the four CALA summary measures are descriptive and always reported with the absolute differences they summarize.

- **Calibration intercept** (slope fixed at 1): 0 is ideal; > 0 means under-prediction.
- **Calibration slope:** 1 is ideal; < 1 means predictions are too extreme.
- **ECE** (15 equal-width bins) is reported with its **value expected under perfect calibration** (mean ECE when outcomes are simulated from the model's own predictions), because binned ECE is biased upward in small samples. **ICI** is a binning-free check.
- **CDH-50** (Calibration Decay Half-Life) = ln 2 / β₁, where β₁ is the change in natural-log ECE per calendar year over four MIMIC-IV periods. Defined only when β₁ > 0; here β₁ < 0, so it is not estimable.
- **CDC** (Continental Drift Coefficient) = ΔECE₄ / max(ΔECE₂, 0.005), with ΔECEₛ = ECEₛ − ECE₁. A descriptive ratio whose name implies no continental cause. Because ΔECE₂ ≈ 0, CDC depends on its floor (1.55 to 6.20), so the absolute difference ΔECE₄ − ΔECE₂ = 0.016 (95% CI 0.011 to 0.023) is the primary contrast.
- **SDII** (Subgroup Decay Inequity Index) = range of ECE across four sex × age strata at a stage ÷ the range at Stage 1. Values: 0.60, 0.83 and 1.35 at Stages 2 to 4; every 95% CI includes 1, because the Stage 1 range is close to sampling noise.
- **RHR** (Recalibration Half-Life Recovery) = 1 − ECE_post / ECE_pre: a proportional reduction relative to the uncalibrated model, not recovery to the development baseline (reported separately). Read it alongside AUROC: a model without discrimination reaches near-zero ECE by predicting the base rate.

---

## Candidate Governance Checks (hypotheses for prospective evaluation)

These are starting points, not validated rules. Neither they nor any monitoring interval has been shown to improve decisions or outcomes.

1. **Assess locally before use.** On the receiving site's own recent data: calibration intercept and slope with CIs, a reliability curve, and ECE or ICI judged against its value expected under perfect calibration. Recalibrate when indicated, not automatically because a boundary has been crossed.
2. **Candidate trigger for review:** |intercept| ≥ 0.2 (about 20% in the odds of death) with a 95% CI excluding 0, or a slope CI excluding 1.
3. **Know what a local audit can detect.** In our simulations, 250 to 500 stays detected a large shift (the Stage 4 intercept of 0.77) in 79% to 95% of audits, with 5% to 11% false alarms, but a shift of 0.2 in only 14% to 22% (87% with 4,000 stays), and a shift of 0.1 in at most 34%.
4. **Prefer logistic recalibration with small samples.** With 250 local stays it was the most stable method; intercept-only updating is not enough when predictions are too extreme.
5. **Check subgroups and units locally.** Error concentrated in particular ICUs as much as in demographic groups.
6. **Harmonize first.** Outcome definitions, data sources, units, sampling frequency and time anchors must match before miscalibration is attributed to the model. In this study they caused most of the originally reported drift.

The ECE > 0.05 and SDII > 2 triggers and the monitoring schedule proposed in the original submission are withdrawn: the triggers would not have flagged the European center, where recalibration reduced ECE by 78%.

---

## Limitations

Stage 4 is one European center, so country, health-system and center effects cannot be separated. Race and ethnicity are unavailable in SICdb. SICdb units do not record the level of care each patient received, and one unit code (6% of stays) is not decoded in the SICdb reference table. The 24-hour landmark design excludes patients who died or left the ICU within 24 hours. Count features and missingness indicators partly encode documentation practice, and external calibration was sensitive to them. Recalibration used contemporaneous labels. The primary model's low internal ECE partly reflects its class weighting. Two MIMIC-IV item mappings fixed at development (a urine pH item, and one item listed under both lactate and bicarbonate) were left unchanged to preserve the frozen model.

---

## Data Restrictions

In accordance with PhysioNet data use agreements:
- Raw patient-level data **cannot** be redistributed in any form, and the `cache/` folder (patient-level Parquet files) is not shared.
- The shared outputs are aggregate tables, figures, manifests and model artifacts. Their redistribution is governed by the terms of each database's data use agreement.
- Access to the underlying databases must be obtained individually through [physionet.org](https://physionet.org).

---

## Reporting Standards

The study is reported according to **TRIPOD+AI** (Collins et al., BMJ 2024); the completed checklist is Supplementary Table S1. PROBAST-AI domains guided design choices, but no formal risk-of-bias rating was made.

---

## Citation

If you use this framework, code or the CALA measures, please cite:

```bibtex
@article{cala2026,
  title   = {Special Topic AI in CDS: A Multi-Database Calibration and Recalibration Audit for Safe Deployment of Imported ICU Mortality Models (CALA)},
  author  = {XXX and XXX and XXX and XXX},
  journal = {Applied Clinical Informatics},
  year    = {2026},
  note    = {Under review}
}
```

## Authors

**XXX**

**XXX**

**XXX**

**XXX**

## License

The code in this repository is released under the [MIT License](LICENSE). Data artifacts derived from PhysioNet databases are subject to the terms of the respective PhysioNet data use agreements and are not covered by this license.

---

## Acknowledgements

The authors acknowledge the MIT Laboratory for Computational Physiology for MIMIC-IV; Philips Healthcare and the MIT Critical Data group for eICU-CRD; and Dr. Niklas Rodemund and colleagues at Salzburg University Hospital for SICdb. We thank PhysioNet for credentialed access infrastructure and the maintainers of scikit-learn, LightGBM, SHAP, pandas, NumPy and matplotlib.
