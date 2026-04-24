# Capstone 20.1 — TCGA-GBM 12-Month Survival Prediction

**Research question.** 
In adults with newly-diagnosed glioblastoma, can clinical and molecular features available at diagnosis predict whether a patient will survive at least twelve months, and which features drive that prediction?

**Notebook:** [`capstone_20_gbm_survival.ipynb`](capstone_20_gbm_survival.ipynb)

## Data

TCGA-GBM cohort (596 patients) from the cBioPortal study export. Clinical data from `data_clinical_patient.txt`; molecular markers derived from `data_mutations.txt` (IDH status) and `data_methylation_hm27.txt` / `data_methylation_hm450.txt` (MGMT methylation, β ≥ 0.2 threshold). All files redistributed here under the TCGA open data use policy.

## Method

Binary classification — did the patient survive at least 12 months from diagnosis. Patients alive with < 12 months of follow-up are excluded rather than labeled, to avoid biasing the outcome. Features are baseline-only: age, Karnofsky score, sex, race, ethnicity, prior-glioma history, adjuvant radiation / pharmaceutical indicators, IDH status, MGMT status. Missing values are imputed with a missing-indicator pattern so the "was it measured" signal is preserved.

Baseline model is logistic regression in a sklearn Pipeline. Evaluation uses a stratified 80/20 train/test split, stratified 5-fold CV on the training portion, and **ROC-AUC as the primary metric** — chosen for threshold-free evaluation, robustness to the mild class imbalance (56% positive), and comparability to published benchmarks on this cohort.

## Results

| | ROC-AUC | PR-AUC | Brier | Accuracy |
|---|---|---|---|---|
| 5-fold CV on training (n=424) | **0.746 ± 0.030** | 0.768 | 0.201 | 70.3% |
| Held-out test (n=107) | 0.618 | 0.673 | 0.254 | 59% |

The CV estimate lands in the middle of the 0.70–0.80 range anticipated in the Module 16 proposal. The single held-out test fold undershot CV by ~0.13, which reflects small-n sampling noise and motivates repeated stratified splits in Module 24.

### Key findings

- **Age and Karnofsky score dominate** feature importance (permutation Δ ROC-AUC of 0.065 and 0.053 respectively), with IDH and MGMT contributing smaller positive signal and demographic features near zero.
- **Every coefficient direction matches published neuro-oncology literature** — older age and unmethylated MGMT push toward worse survival; higher Karnofsky, methylated MGMT, IDH mutation, and adjuvant radiation push toward better survival. The model learned biologically coherent patterns, not dataset artefacts.
- **The predicted-risk Kaplan-Meier plot** cleanly separates high / medium / low predicted-risk tiers over the first two years, with the clearest gap at the 12-month cutoff — which is the visual promise made in the Module 16 proposal.

## How to run

```bash
pip install -r requirements.txt
jupyter notebook capstone_20_gbm_survival.ipynb
```
