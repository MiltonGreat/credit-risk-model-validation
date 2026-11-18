---
Artifact ID: LC-A05-MD
Version: 1.2
Run ID: 20261119_140600
Document Title: Methodology Document
Project: Lending Club Consumer Credit Risk Model Validation
---

# Methodology Document (A05)

This document describes how the analysis is implemented and how it is validated. It explicitly answers six core questions:

---

## 1. What candidate methodologies were considered?

Four candidate model classes were evaluated under identical hyperparameter constraints and data inputs[cite: 1]:

| Candidate | Description | AUC (out-of-time test) | Interpretability | Reproducible | Selected? |
|---|---|---|---|---|---|
| A — Logistic regression | Application-time features only[cite: 1] | 0.63[cite: 1] | High[cite: 1] | Yes[cite: 1] | No — below tolerance[cite: 1] |
| B — Gradient boosted trees | Application-time features only[cite: 1] | 0.68[cite: 1] | Medium[cite: 1] | Yes[cite: 1] | No — below tolerance[cite: 1] |
| C — Ensemble (XGBoost + Random Forest) | Application-time features + feature engineering[cite: 1] | 0.72 (with grade/sub_grade); 0.66 (without)[cite: 1] | Medium[cite: 1] | Yes, given a seed[cite: 1] | **Yes — best balance**[cite: 1] |
| D — Neural network | Feature engineering[cite: 1] | 0.71[cite: 1] | Low[cite: 1] | Difficult[cite: 1] | No — less interpretable, harder to reproduce[cite: 1] |

---

## 2. Why was one selected?

"The selected method is the simplest method that satisfies all critical criteria. Candidate C is selected because it is the simplest method that meets the AUC, recall, and precision criteria while remaining reproducible and interpretable enough to support the validation. Candidate D is rejected because it is less interpretable and harder to reproduce, not because it performs worse. Candidate A is rejected because it does not meet the tolerance."[cite: 1]

---

## 3. What baselines are used for comparison?

To evaluate the predictive lift of advanced tree-based ensembles, the framework establishes baseline comparisons:
* **Linear Baseline (Candidate A - Logistic Regression):** Serves as the benchmark linear classifier using application-time features only, testing whether non-linear models yield statistically significant performance gains over standard risk scoring.
* **Unadjusted vs. Leakage-Free Baselines:** Compares developer-claimed performance (incorporating post-origination leakage features) against the rigorous leakage-free feature set, isolating the exact performance delta driven by data contamination.

---

## 4. What performance measures apply?

Held-out evaluation reports a comprehensive suite of metrics to address severe class imbalance (1.78% default rate):
* **Primary Discrimination Metric:** Held-out AUC-ROC (DCR §1 threshold: $\ge 0.65$)[cite: 1].
* **Ranking Sharpness:** Kolmogorov-Smirnov (KS) statistic computed from the ROC curve (thresholds: 0.30 for moderate, 0.40 for strong).
* **Secondary & Operating Point Metrics:** PR-AUC, Brier score, Log Loss, Accuracy, Precision, Recall, and F1 score evaluated at both default 0.5 and optimal Youden thresholds[cite: 1].

---

## 5. How is the methodology validated?

The validation methodology enforces strict independence and technical rigor:
* **Independent Recomputation:** The validator (NB03) independently recomputes all metrics directly from model predictions rather than relying on developer-reported figures, enforcing a reconciliation tolerance within $1e-6$[cite: 1].
* **Integrity Checks:** Implements Hosmer-Lemeshow (HL) calibration tests with explicit degeneracy checks (flagging p-values of exactly 0.0 or 1.0 as numerical artifacts rather than valid calibration results)[cite: 1].
* **Governance Chain:** Enforces a strict verdict-ownership chain where NB02 trains without authoring a verdict, NB03 independently authors the verdict and risk score, and downstream notebooks inherit via strict cryptographic/schema assertions[cite: 1].

---

## 6. How is the methodology explained?

Model transparency and economic logic are verified via explainability tools:
* **Feature Importance & Directionality:** TreeExplainer (SHAP) is applied to a 100-row sample to extract top-10 feature contributions[cite: 1].
* **Monotonicity Checks:** Verified against established credit-risk theory to ensure feature relationships (e.g., higher income or lower debt-to-income reducing default probability) align logically with financial expectations, flagging counterintuitive directions for remediation[cite: 1].

---

## 7. Parameter specification (frozen values)

Hyperparameters are frozen prior to execution to preserve validation independence[cite: 1]:

| Parameter | LR | XGBoost | Random Forest | LightGBM |
|---|---|---|---|---|
| `n_estimators` | — | 100[cite: 1] | 100[cite: 1] | 100[cite: 1] |
| `max_depth` | — | 4[cite: 1] | 6[cite: 1] | 4[cite: 1] |
| `learning_rate` | — | 0.1[cite: 1] | — | default[cite: 1] |
| `subsample` | — | 0.8[cite: 1] | — | default[cite: 1] |
| `colsample_bytree` | — | 0.8[cite: 1] | — | default[cite: 1] |
| `class_weight` / `scale_pos_weight` | balanced dict[cite: 1] | 55.34 (ratio)[cite: 1] | balanced dict[cite: 1] | balanced dict[cite: 1] |
| `C` (regularization) | 1.0[cite: 1] | — | — | — |
| `solver` | lbfgs[cite: 1] | — | — | — |
| `random_state` | 42[cite: 1] | 42[cite: 1] | 42[cite: 1] | 42[cite: 1] |
| `n_jobs` | default[cite: 1] | default[cite: 1] | −1[cite: 1] | 1[cite: 1] |

* **Seed requirement:** `RANDOM_STATE = 42` (AS-001 §8). Changing the seed invalidates all reproducibility claims[cite: 1].

---

## 8. Training and evaluation methodology

* **Split Exception:** While walk-forward splitting is prescribed, a random stratified 80/20 split was utilized due to the removal of the `issue_month` date column during leakage elimination. This exception is recorded as `F-03.02`[cite: 1].
* **Class Imbalance Handling:** Utilizes `class_weight="balanced"` and `scale_pos_weight` without synthetic oversampling (SMOTE) to preserve natural data distributions[cite: 1].

---

## 9. Fairness and calibration methodology

* **Fairness Proxy:** Evaluated using the 4/5ths rule on Census regions (lowest-cardinality geographic proxy) with group size sufficiency threshold $N \ge 30$ (`FD-03`)[cite: 1].
* **Calibration Testing:** Hosmer-Lemeshow test (10 percentile bins) paired with logistic regression slope/intercept analysis of $y$ on $y_{proba}$[cite: 1].

---

## 10. Known methodological limitations and open items

| ID | Limitation / Open Item | Impact | Tracking |
|---|---|---|---|
| **M1** | Random split instead of walk-forward | Not time-validated; PSI unassessable | `LC-R-02` in MAP-RP-001[cite: 1] |
| **M2** | Limited four-model comparison scope | Comparison is informative, not exhaustive | Documented[cite: 1] |
| **M3** | SHAP computed on 100-row sample | Feature importance is indicative, not exhaustive | Documented[cite: 1] |
| **M4** | HL degeneracy for 3 of 4 models | Calibration uncertain for non-LightGBM models | `LC-R-03` in MAP-RP-001[cite: 1] |
| **M5** | Region-as-proxy fairness | Fairness findings are indicative, not definitive | Documented[cite: 1] |
| **M6** | Single-threshold operating point | Operating-point sensitivity of DI is untested | Documented[cite: 1] |
