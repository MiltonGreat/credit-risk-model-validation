---
Artifact ID: LC-A05-MD
Version: 1.2
Run ID: 20261119_141500
Document Title: Methodology Document
Project: Lending Club Consumer Credit Risk Model Validation
---

# Methodology Document (A05)

This document describes how the analysis is implemented and how it is validated. It explicitly answers six core questions:

---

## 1. What candidate methodologies were considered?

Four candidate model classes were evaluated under identical hyperparameter constraints and data inputs:

| Candidate | Description | Intended Role / Target Metric | Interpretability | Reproducible | Selected? |
|---|---|---|---|---|---|
| A — Logistic Regression | Linear benchmark using application-time features | Linear baseline for discrimination | High | Yes | No — below performance threshold |
| B — XGBoost | Gradient boosted decision trees | Non-linear feature interaction capture | Medium | Yes, given seed | No — outperformed by ensemble/bagging on this snapshot |
| C — Random Forest | Bagged decision tree ensemble | Robust non-linear ranking and stability | Medium | Yes, given seed | **Yes — selected model** |
| D — LightGBM | Fast gradient boosting framework | Alternative gradient boosting architecture | Medium | Yes, given seed | No — kept for calibration/diagnostic comparison |

---

## 2. Why was one selected?

The selected method is the simplest method that satisfies all pre-registered criteria. **Random Forest (Candidate C)** is selected because it provides the best balance of non-linear discrimination, robustness against overfitting on tabular credit data, and sufficient interpretability via tree-based feature attributions (SHAP) to support independent validation. Logistic Regression serves as the linear baseline, while XGBoost and LightGBM are retained in the pipeline for cross-model comparison and calibration diagnostics.

---

## 3. What baselines are used for comparison?

To evaluate the predictive lift and structural validity of the selected model, the framework establishes benchmark comparisons:
* **Linear Baseline (Logistic Regression):** Establishes the linear performance floor using application-time features only, determining whether complex non-linear models yield statistically meaningful gains over traditional scoring.
* **Leakage-Free vs. Unadjusted Baseline:** Enforces a structural comparison where developer-claimed performance metrics (which incorporate post-origination outcome features) are contrasted against the leakage-free feature space to isolate the performance impact of data contamination.

---

## 4. What performance measures apply?

To address severe class imbalance (approx. 1.78% default rate), held-out evaluation mandates a specialized suite of metrics:
* **Primary Discrimination Metric:** Held-out AUC-ROC (pre-registered threshold: $\ge 0.65$).
* **Ranking Sharpness:** Kolmogorov-Smirnov (KS) statistic computed from the ROC curve (pre-registered floors: 0.30 for moderate, 0.40 for strong separation).
* **Secondary & Operating Point Metrics:** PR-AUC, Brier score, Log Loss, Accuracy, Precision, Recall, and F1 score evaluated at both default (0.5) and optimal Youden thresholds.

---

## 5. How is the methodology validated?

The validation methodology enforces strict independence and operational rigor:
* **Independent Recomputation:** The validator (NB03) independently recomputes all metrics directly from model predictions rather than relying on developer-reported CSVs, enforcing a reconciliation tolerance within $1e-6$.
* **Calibration & Integrity Checks:** Implements Hosmer-Lemeshow (HL) calibration tests paired with an explicit degeneracy check (flagging HL p-values of exactly 0.0 or 1.0 as numerical artifacts rather than valid calibration outcomes).
* **Governance Chain:** Enforces a strict verdict-ownership chain where NB02 trains without authoring a verdict, NB03 independently authors the validation verdict and risk score, and downstream notebooks inherit via strict cryptographic/schema assertions.

---

## 6. How is the methodology explained?

Model transparency and alignment with economic logic are verified via structured explainability tools:
* **Feature Importance & Directionality:** TreeExplainer (SHAP) is applied to extract top feature contributions.
* **Monotonicity Checks:** Verified against established credit-risk theory to ensure feature relationships (e.g., higher income or lower debt-to-income reducing default risk) align logically with financial principles, flagging counterintuitive directions for formal remediation.

---

## 7. Parameter specification (frozen values)

Hyperparameters are frozen prior to execution to preserve validation independence:

| Parameter | LR | XGBoost | Random Forest | LightGBM |
|---|---|---|---|---|
| `n_estimators` | — | 100 | 100 | 100 |
| `max_depth` | — | 4 | 6 | 4 |
| `learning_rate` | — | 0.1 | — | default |
| `subsample` | — | 0.8 | — | default |
| `colsample_bytree` | — | 0.8 | — | default |
| `class_weight` / `scale_pos_weight` | balanced dict | 55.34 (ratio) | balanced dict | balanced dict |
| `C` (regularization) | 1.0 | — | — | — |
| `solver` | lbfgs | — | — | — |
| `random_state` | 42 | 42 | 42 | 42 |
| `n_jobs` | default | default | −1 | 1 |

* **Seed requirement:** `RANDOM_STATE = 42` (AS-001 §8). Changing the seed invalidates all reproducibility claims.

---

## 8. Training and evaluation methodology

* **Split Exception:** While walk-forward splitting is prescribed, a random stratified 80/20 split is utilized due to the removal of the `issue_month` date column during leakage elimination. This exception is formally recorded and tracked.
* **Class Imbalance Handling:** Utilizes `class_weight="balanced"` and `scale_pos_weight` without synthetic oversampling (SMOTE) to preserve natural data distributions.

---

## 9. Fairness and calibration methodology

* **Fairness Proxy:** Evaluated using the 4/5ths rule on Census regions (lowest-cardinality geographic proxy) with group size sufficiency threshold $N \ge 30$ (`FD-03`).
* **Calibration Testing:** Hosmer-Lemeshow test (10 percentile bins) paired with logistic regression slope/intercept analysis of $y$ on $y_{proba}$.

---

## 10. Known methodological limitations and open items

| ID | Limitation / Open Item | Impact | Tracking |
|---|---|---|---|
| **M1** | Random split instead of walk-forward | Not time-validated; PSI unassessable | `LC-R-02` in MAP-RP-001 |
| **M2** | Limited four-model comparison scope | Comparison is informative, not exhaustive | Documented |
| **M3** | SHAP computed on sub-sample | Feature importance is indicative, not exhaustive | Documented |
| **M4** | HL degeneracy for subset of models | Calibration uncertain for non-validated models | `LC-R-03` in MAP-RP-001 |
| **M5** | Region-as-proxy fairness | Fairness findings are indicative, not definitive | Documented |
| **M6** | Single-threshold operating point | Operating-point sensitivity of DI is untested | Documented |
