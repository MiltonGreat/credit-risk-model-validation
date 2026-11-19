# Lending Club Credit Risk Model Validation

An independent validation that caught a 27% performance inflation and a severe fairness defect — before deployment.

**Python | OSFI E-23 | SR 11-7 | Verdict Status: REJECT**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![OSFI E-23](https://img.shields.io/badge/OSFI%20E--23-Aligned-lightgrey)
![SR 11-7](https://img.shields.io/badge/SR%2011--7-Partially%20Aligned-yellow)
![Verdict](https://img.shields.io/badge/Verdict-REJECT-red)
![Status](https://img.shields.io/badge/Status-Portfolio%20Artifact-lightgrey)

---

## The business problem

A first-line model development team built a consumer credit risk model for early-warning collections prioritization (16+ days past due). They reported AUC 0.865 and requested approval for deployment.

Before a model can be deployed in a regulated lending environment, an independent validator must answer four questions:
1. Does the model perform as claimed?
2. Is it fair across protected groups?
3. Does it meet the regulatory requirements it will be held to?
4. Can it be deployed?

This project answers all four. The short answers are: not as claimed; no; partially; and not without remediation.

---

## The two findings that mattered most

### 1. The reported performance was inflated by 27%
Nine features in the training data — `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, and `issue_month` — contain post-origination information. When a bank decides whether to approve a loan, it does not know whether the borrower will pay late next month or charge off in two years. The nine features are exactly that future information.

Including them in training teaches the model to "predict" the past. When they were removed, the model's held-out AUC dropped from 0.865 to 0.631 — a 27.1% inflation of measured performance (23.4 AUC percentage points).

* **Why this matters in business terms:** A model deployed against the inflated metric would have looked like it performed well in validation and then silently underperformed in production. Collections teams would have prioritized the wrong accounts. The bank would have spent months trying to understand why the model's real-world results didn't match its validation.
* **What the drop does not mean:** It does not mean the developer lied or made an error. The 0.865 is the leakage-included AUC — what the model would achieve if the leakage features were available at decision time. The 0.631 is the leakage-free AUC — what the model achieves when the leakage features are removed. Both numbers are correct; they measure different models.
* **Status:** **CRITICAL, resolved** (leakage features removed in the pipeline; independently recomputed in NB03).

### 2. The model approved applicants at materially different rates across U.S. regions
The model approves applicants at substantially different rates across U.S. census-style regions. Region is the geographic proxy for protected-class composition in this dataset — the data has no direct race or ethnicity field.

* **The measurement:** The Disparate Impact ratio (DI) compares the lowest-approval group's approval rate to the highest. The 4/5ths rule (the industry standard) requires DI $\ge 0.80$ for a group not to be flagged. After aggregating the 50 U.S. states into four regions (South, West, Northeast, Midwest), DI was **0.5414** — below 0.80 and inside the "SEVERE" band ($\text{DI} < 0.60$).
* **Why per-state DI would be invalid:** The 4/5ths rule is designed for binary or low-cardinality protected attributes. Applied to 50 one-hot state columns, it produces false positives (sampling noise at cardinality 50) and false negatives (a genuinely disadvantaged state masked by the maximum-approval state). The FD-05 cardinality check ($\le 50$ groups) is the gate; the region mapping is the mechanism.
* **Why this matters in business terms:** A bank deploying this model without mitigation would face fair-lending exposure. The finding blocks deployment until the disparity is remediated.
* **Status:** **CRITICAL, open.** This is the primary driver of the REJECT verdict.

---

## The verdict

**6/10 → HIGH → REJECT for deployment in current form.**

| Component | Score | Basis |
| :--- | :--- | :--- |
| **Performance** | 1 / 3 | AUC 0.631, MODERATE band |
| **Stability** | 0 / 2 | PSI not assessed (random split) |
| **Calibration** | 2 / 2 | XGBoost poor |
| **Fairness** | 3 / 3 | DI 0.5414, SEVERE |
| **Leakage** | 0 / 3 | Resolved |

The rejection is fairness-driven, not performance-driven. AUC 0.631 is a MODERATE finding and contributes only 1 of the 6 points. The score reaches HIGH because fairness (3 points) and calibration (2 points) dominate. 

* **What the model may be used for:** Human-led early-warning collections prioritization, subject to quarterly fairness monitoring and monthly performance monitoring.
* **What it may not be used for:** Fairness-sensitive decisions without further mitigation, IFRS 9 provisioning, Canadian portfolios, or automated decisions without human oversight.

### Full approval requires:
1. Fairness remediation to $\text{DI} \ge 0.80$ on a low-cardinality attribute (FD-05 compliant).
2. Validation on Canadian data.
3. Calibration finding resolved (HL implementation verified; poor models recalibrated or replaced).
4. Walk-forward split implemented, or a documented DCR amendment.
5. Investigation of four counterintuitive SHAP features (F-03.07).

> *Read with:* `governance/remediation-plan.md` — each condition mapped to a finding ID, owner, deadline, and success criterion.

---

## How to read this project

### The chain of enforcement
The verdict lives in Notebook 03 (independent validation). Notebook 04 (governance) inherits that verdict — it does not re-derive it. The chain of enforcement is explicit at three checkpoints:
* **NB02 writes:** `does_not_contain: ["risk_score", "risk_rating", "recommendation"]` (NB02's self-check rejects unapproved writes).
* **NB03 reads:** Verifies schema contracts, raising a `ValueError` if violated.
* **NB04 reads:** Asserts `authored_by == "NB03"` and verifies risk score totals before generating regulatory reports.

### The analyst's role
As a GRC Risk Analyst, the work here is validation, not development: independently assessing model claims, quantifying material issues (data leakage, fairness, calibration), determining regulatory compliance status, and producing a formal findings register with severity, ownership, and remediation paths.

---

## Key findings register

Findings summary (NB04 confirmed output): **11 total** — 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 resolved, 10 open.

| ID | Finding | Severity | What it means in plain terms | Status |
| :--- | :--- | :--- | :--- | :--- |
| **F-03.01** | Data leakage — 9 features | CRITICAL | Model used post-origination information not available at decision time. | **RESOLVED** |
| **F-03.03** | Severe disparate impact | CRITICAL | Approval rates differed materially across U.S. regions ($\text{DI} = 0.5414$). Deployment blocked. | **OPEN** |
| **F-03.02** | Walk-forward split not implemented | HIGH | Trained on random sample rather than temporal splits. | **OPEN** |
| **F-03.10** | IFRS 9 misalignment | HIGH | Model defines default as 16+ days past due (Stage 3 requires 30+). | **OPEN** |
| **F-03.09** | Foreign data applicability | HIGH | Trained on U.S. Lending Club data; not validated on Canadian portfolios. | **OPEN** |
| **F-03.04** | Poor calibration (XGBoost) | HIGH | Predicted probabilities are unreliable. | **OPEN** |
| **F-03.08** | No macroeconomic factors | HIGH | Model does not account for macroeconomic cycles. | **OPEN** |
| **F-03.06** | Weak discrimination (KS = 0.267) | HIGH | Weak separation of defaulters in top tail. | **OPEN** |
| **F-03.07** | Counterintuitive SHAP relationships | HIGH | Top-10 drivers behave opposite to credit-risk theory. | **OPEN** |
| **F-03.11** | Low precision (0.024–0.085) | MEDIUM | High false-positive rate when flagging defaults. | **OPEN** |
| **F-03.05** | Suspicious HL p-values | MEDIUM | Hosmer-Lemeshow test degeneracy (p=0 or p=1). | **OPEN** |
| **F-03.12** | Protected attribute is a proxy | MEDIUM | Region used as proxy for protected class. | **OPEN** |

> *Read with:* `outputs/reports/validation_findings_register.csv` and `outputs/reports/validation_report_v2.txt`.

---

## Model performance (Lending Club)

Best model per held-out AUC, after removal of the nine leakage features:

| Model | AUC-ROC | Precision | Recall | Brier | HL p-value | Calibration |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Random Forest** | **0.631** | 0.085 | 0.333 | 0.127 | 1.000 (suspicious) | GOOD* |
| **LightGBM** | **0.621** | 0.029 | 0.694 | 0.084 | 0.729 | GOOD |
| **XGBoost** | **0.604** | 0.029 | 0.444 | 0.070 | 0.000 (POOR) | POOR |
| **Logistic Regression** | **0.536** | 0.039 | 0.333 | 0.220 | 1.000 (suspicious) | GOOD* |

*\* HL p-value of exactly 1.000 is a known degeneracy pattern (Finding F-03.05).*

---

## Regulatory compliance status

* **OSFI E-23:** 5 of 7 fully compliant (71%) — *Gap: Foreign data not validated for Canada, monthly monitoring framework missing.*
* **SR 11-7:** 4 of 6 fully compliant (67%) — *Gap: Ongoing monitoring not established, severe disparate impact.*
* **ECOA / CFPB:** 0 of 2 fully compliant — *Gap: Disparate impact testing ($\text{DI} < 0.80$), region proxy.*
* **IFRS 9:** 0 of 2 fully compliant — *Gap: Stage 3 classification threshold (16 vs 30 days), scope restriction (collections vs. provisioning).*

---

## Project structure

| Notebook | Purpose | Key Output |
| :--- | :--- | :--- |
| **`01_exploratory_data_analysis`** | Data quality, leakage detection, feature inventory | Data quality summary, temporal audit, feature inventory |
| **`02_model_development`** | Model training, leakage quantification, fairness diagnostic | Trained models, metrics, `performance_summary_v2.json` |
| **`03_model_validation`** | Independent validation — the authoritative verdict | Validation report, risk rating (6/10 REJECT), findings register |
| **`04_governance_compliance_report`** | Regulatory mapping and compliance scorecard | Compliance report, gap analysis, scorecard JSON |
| **`05_german_credit_cross_validation`** | Cross-population test & leakage-free predictive validation | Leakage-free AUC (~0.76–0.78), `SMALL_DIRECTIONAL` fairness finding |

---

## German Credit cross-validation (Notebook 05)

Notebook 05 provides both **leakage-free predictive cross-validation** (confirming models achieve robust out-of-sample AUCs between `0.76` and `0.78` under Run ID `20261118_170144`) and a **cross-population fairness test** of proxy-feature mitigation:

| Dataset | Baseline DI | Status | Mitigation | Post-mitigation DI | Status | $\Delta$ DI |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lending Club** | 0.5414 | SEVERE | Not applied in NB03 | `not_assessed` | — | — |
| **German Credit** | 0.9375 | OK | Removed Housing | 0.9740 | OK | +0.0365 |

* **Classification:** `SMALL_DIRECTIONAL`. Proxy removal improves Disparate Impact in the same direction across datasets, but German Credit's baseline was already acceptable ($\text{DI} = 0.9375$), whereas Lending Club's was severe ($\text{DI} = 0.5414$). The cross-validation establishes directional validity, not magnitude equivalence.

---

## Technical stack

* **Language:** Python 3.10+
* **Core Libraries:** `pandas`, `numpy`, `scipy`, `scikit-learn`
* **Models:** XGBoost, LightGBM, Random Forest, Logistic Regression
* **Explainability & Fairness:** SHAP, `fairlearn`, hand-computed metrics
* **Governance Outputs:** Immutable JSON and CSV artifacts carrying unique `run_id` and version tags.

---

## Key documents & governance folders

* **`governance/`** — 12 project-governance artifacts (Problem Definition, Analytical Specification, Data Spec, Feature Log, Methodology, Risk Plan, Decision Criteria, Consolidated Findings, Remediation Plan, Approval Record, Analysis Code, Validation Opinion).
* **`reports/`** — `validation-opinion.md` (signed independent validation verdict).
* **`outputs/reports/`** — Machine-readable validation reports, fairness validation CSVs, performance summaries, and German Credit cross-validation JSONs.
