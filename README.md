# Lending Club Credit Risk Model Validation

**A GRC analyst's independent validation of a consumer credit risk model.**

## Project Overview

**Role:** GRC Risk Analyst
**Focus:** Independent model validation and regulatory compliance
**Regulatory frameworks:** OSFI E-23, SR 11-7, ECOA/CFPB, IFRS 9, OSFI B-13

This project is an independent validation of a credit risk model developed on Lending Club data. The validation follows OSFI E-23 and SR 11-7 principles, assessing model performance, data quality, fairness, and regulatory compliance.

It is a portfolio artifact: the methodology and governance structure are the point, not the specific findings on Lending Club data.

---

## How to read this project

If you're new to model validation, this section explains what the project is doing.

**The setup.** A model development team built a credit risk model to support early-warning collections prioritization — identifying which accounts are likely to default in time to intervene. They reported strong performance and requested approval for deployment.

**The validation.** My job is not to build a model. My job is to answer four questions independently:

1. Does the model perform as claimed?
2. Is it fair across protected groups?
3. Does it meet the regulatory requirements it will be held to?
4. Can it be deployed?

**The answers.** Not as claimed; no; partially; and not without remediation.

**The governance discipline.** The verdict lives in **Notebook 03 (independent validation)**. Notebook 04 (governance) inherits that verdict — it does not re-derive it. Two notebooks reporting different verdicts on the same model would be a governance failure, not a feature. The `performance_summary_v2.json` artifact records which notebook authored the verdict (`authored_by: "NB03"`) so the chain is auditable.

---

## Business problem

A first-line model development team built a credit risk model intended to support early-warning collections prioritization (16+ days past due).

This validation addresses four questions:

1. Does the model perform as claimed?
2. Is it fair across protected groups?
3. Does it meet the regulatory requirements it will be held to?
4. Can it be deployed?

**Short answers:** not as claimed; no; partially; and not without remediation.

**Long answers:** in the notebooks.

---

## The analyst's role

As a GRC Risk Analyst, the work here is validation, not development:

- **Independently assess** the model built by the development team
- **Identify and quantify** material issues (data leakage, fairness, calibration)
- **Determine** the model's regulatory compliance status
- **Produce** a findings register with severity, owner, remediation, and status
- **Report** a risk rating and deployment recommendation

### What makes this a validation project, not a modeling project

The model exists. The question is whether it is fit for purpose. Everything in Notebooks 03, 04, and 05 is written from the perspective of someone checking a claim made by someone else — not someone building a model and reporting its performance.

The notebooks do **not** produce a narrative about the developer. They compute the numbers and export them. The verdict is a coded risk rating (`LOW` / `MODERATE` / `HIGH` / `CRITICAL`) with a coded recommendation (`APPROVE` / `APPROVE WITH CONDITIONS` / `REJECT`). The prose interpretation — "the developer's claim was overstated" — is authored by me, in the Validation Opinion, not by the notebooks.

---

## Key findings

Each finding below traces to a `Finding ID` in the machine-readable register (`outputs/reports/validation_findings_register.csv`) and to a specific notebook phase.

| ID | Finding | Severity | What it means in plain terms | Status |
|---|---|---|---|---|
| F-03.01 | **Data leakage — 9 features** | CRITICAL | The model was using information that does not exist at the moment a credit decision has to be made (e.g., whether the borrower later paid late). The originally-reported performance was measuring the wrong thing. | RESOLVED |
| F-03.03 | **Severe disparate impact (fairness)** | CRITICAL | Approval rates differed materially across U.S. regions (the proxy for protected-class composition). A group in one region was approved at 54% of the rate of the highest-approval region — below the industry-standard 80% threshold. Deployment is blocked until remediated. | OPEN |
| F-03.02 | **Walk-forward split not implemented** | HIGH | The model was trained on a random sample of loans rather than "past loans predict future loans." This can inflate measured performance and masks how the model behaves on newer borrower behavior. | OPEN |
| F-03.10 | **IFRS 9 misalignment** | HIGH | The model defines "default" as 16+ days past due. IFRS 9 requires 30+ days for Stage 3. The model cannot be used for IFRS 9 provisioning without recalibration. | OPEN |
| F-03.09 | **Foreign data applicability** | HIGH | Trained on U.S. Lending Club data. Not validated on Canadian portfolios. Cannot be deployed in Canada without revalidation. | OPEN |
| F-03.04 | **Poor calibration (XGBoost)** | HIGH | Predicted probabilities are not reliable. A model that says "30% chance of default" does not mean 30% of such borrowers default. | OPEN |
| F-03.08 | **No macroeconomic factors** | HIGH | The model does not account for the state of the economy. It cannot anticipate how a recession changes default behavior. | OPEN |
| F-03.06 | **Weak discrimination (KS = 0.267)** | HIGH | The model is weak at separating likely defaulters from likely non-defaulters in the top tail. KS (Kolmogorov-Smirnov) is a standard separation statistic; below 0.30 is conventionally "weak." | OPEN |
| F-03.07 | **Counterintuitive SHAP relationships (4 features)** | HIGH | Four of the model's top-10 drivers behave in the opposite direction from credit-risk theory (e.g., higher loan-to-income correlates with *lower* predicted risk). These require investigation before the model's drivers can be trusted. | OPEN |
| F-03.11 | **Low precision (0.024–0.085)** | MEDIUM | When the model flags an account as "will default," it is wrong most of the time. This may still be acceptable for a prioritization tool if recall is high — see the operating-point discussion in NB03 Phase 5. | OPEN |
| F-03.05 | **Suspicious HL p-values (3 models)** | MEDIUM | The Hosmer-Lemeshow calibration test returned p-values of exactly 0 or 1 for three of four models. This is a known degeneracy pattern and needs the HL implementation verified before the calibration finding is treated as final. | OPEN |
| F-03.12 | **Protected attribute is a proxy** | MEDIUM | The dataset has no direct race/ethnicity field. Region was used as a proxy. Proxy-based fairness findings are indicative, not definitive. | OPEN |

**Findings summary:** 11 total — 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 resolved, 10 open.

> **Read with:** `outputs/reports/validation_findings_register.csv` (machine-readable) and `outputs/reports/validation_report_v2.txt` (narrative).

---

## The two findings that matter most

### Data leakage (resolved)

Nine features — including `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, and `issue_month` — contained post-origination information that would not exist at the moment of a real credit decision.

**What "post-origination" means.** When a bank decides whether to approve a loan, it has the application, the borrower's credit file, and the loan terms. It does not know whether the borrower will pay on time next month, or whether the loan will be charged off in two years. The nine features above are exactly that future information. Including them in training teaches the model to "predict" the past.

**Why it inflates AUC.** A model that has seen the outcome has an easier job. Removing the nine features dropped the model's reported AUC from **0.865 to 0.631** — a **27.1% inflation** of measured performance. The original performance claim was therefore invalid.

**Why this is critical, not high.** Leakage in a deployed model would have silently caused worse real-world decisions than the measured performance suggested. Without remediation, the model would have been approved against a metric that could not be reproduced in production.

### Severe disparate impact (open)

The model approves applicants at substantially different rates across U.S. regions. Region is the geographic proxy for protected-class composition in this dataset — the data has no direct race/ethnicity field.

**The measurement.** The Disparate Impact ratio (DI) compares the lowest-approval group's approval rate to the highest. The 4/5ths rule (the industry standard) requires DI ≥ 0.80 for a group not to be flagged.

**What this run found.** After aggregating the 50 U.S. states into 4 census-style regions (South, West, Northeast, Midwest — plus an "Other" bucket excluded for insufficient sample size), DI was **0.5414** — below the 0.80 threshold and inside the "SEVERE" band (DI < 0.60).

**What "mitigation" means here.** The mitigation tested in this project is proxy-feature removal: drop the geographic feature and retrain. The German Credit cross-validation (Notebook 05) tests whether this technique generalizes to a structurally different dataset. It does — directionally — but the effect is small (delta DI +0.036).

**Status.** Deployment is blocked until the finding is closed. "Closed" means DI ≥ 0.80 on a low-cardinality attribute, measured on a properly held-out population, with the fairness test applied as FD-05 specifies (cardinality ≤ 50, N ≥ 30 per group).

> **Read with:** `outputs/reports/fairness_validation.csv` (per-region metrics) and `docs/remediation_plan.md` §R-02 (the action plan).

> **Note on numbers.** Earlier versions of this README cited a baseline DI of 0.244 and a post-mitigation DI of 0.798. Those came from a pre-correction run that measured fairness at the per-state one-hot level, which is not the 4/5ths test. The corrected pipeline measures at the **region level** — DI 0.5414. The two figures are not comparable; the corrected one is the one to cite.

---

## Model performance

Best model per held-out AUC, after removal of the nine leakage features. Higher AUC is better (0.5 = random, 1.0 = perfect). Precision is the fraction of flagged accounts that actually default. Recall is the fraction of actual defaulters that get flagged.

| Model | AUC-ROC | Precision | Recall | Brier | HL p-value | Calibration |
|---|---|---|---|---|---|---|
| **Random Forest** | **0.631** | 0.085 | 0.333 | 0.127 | 1.000 (suspicious) | GOOD* |
| XGBoost | 0.604 | 0.029 | 0.444 | 0.070 | 0.000 (POOR) | POOR |
| LightGBM | 0.621 | 0.029 | 0.694 | 0.084 | 0.729 | GOOD |
| Logistic Regression | 0.536 | 0.039 | 0.333 | 0.220 | 1.000 (suspicious) | GOOD* |

\* HL p-value of exactly 1.000 is a known degeneracy pattern, flagged as Finding F-03.05. The calibration for those models cannot be trusted until the HL implementation is verified.

**Best by held-out AUC:** Random Forest (0.6307).

**Developer's original claim:** AUC 0.865 (with leakage features).

**Realistic performance:** AUC 0.631.

**Drop:** 0.234 AUC points (27.1%).

**KS statistic (Random Forest):** 0.2671 — weak discrimination by conventional thresholds.

> **Read with:** `models/performance_metrics.csv` (raw metrics) and `outputs/reports/validation_report_v2.txt` §2 (narrative).

---

## Regulatory compliance status

The compliance assessment reflects Notebook 04's exact-match counting: only requirements with status exactly `COMPLIANT` are counted as compliant. Partially compliant entries are treated as not-yet-compliant — the conservative reading appropriate for a governance report.

### OSFI E-23 compliance

| Section | Status | Gap |
|---|---|---|
| 3.1 — Model Validation | COMPLIANT | Foreign data not validated for Canada |
| 3.2 — Performance Monitoring | PARTIALLY COMPLIANT | Monthly monitoring framework not established |
| 3.3 — Data Quality | COMPLIANT | Macroeconomic factors missing |
| 4.1 — Model Documentation | COMPLIANT | — |
| 4.2 — Model Governance | COMPLIANT | — |
| 5.1 — Model Risk | COMPLIANT | — |
| 5.2 — Fairness | NON-COMPLIANT | Severe disparate impact (DI 0.5414 < 0.60, SEVERE band) |

**Rate:** 5 of 7 fully compliant (71%).

### SR 11-7 compliance

| Principle | Status | Gap |
|---|---|---|
| 1 — Validation | COMPLIANT | — |
| 2 — Ongoing Monitoring | PARTIALLY COMPLIANT | Monthly tracking not established |
| 3 — Model Documentation | COMPLIANT | — |
| 4 — Model Risk | COMPLIANT | — |
| 5 — Fairness | NON-COMPLIANT | Severe disparate impact |
| 6 — Data Quality | COMPLIANT | Macroeconomic factors missing |

**Rate:** 4 of 6 fully compliant (67%).

### ECOA/CFPB fair lending

| Requirement | Status | Gap |
|---|---|---|
| Disparate impact testing | NON-COMPLIANT | DI 0.5414 < 0.80 |
| Protected attribute analysis | PARTIALLY COMPLIANT | Region proxy only; no explicit race/gender data |

**Rate:** 0 of 2 fully compliant (0%).

### IFRS 9

| Requirement | Status | Gap |
|---|---|---|
| Stage 3 classification (30+ days) | NON-COMPLIANT | Model uses 16+ day threshold |
| Provisioning (ECL calculation) | NON-COMPLIANT | Model not designed for provisioning |

**Rate:** 0 of 2 fully compliant (0%).

> **Read with:** `outputs/reports/regulatory_mapping_v2.csv` (full mapping) and `outputs/reports/compliance_report_v2.txt` (narrative).

---

## Project structure

| Notebook | Purpose | Key output |
|---|---|---|
| 01_exploratory_data_analysis | Data quality, leakage detection, feature inventory | Data quality summary, findings register (NB01), temporal audit, feature inventory |
| 02_model_development | Model training, leakage quantification, fairness diagnostic | Trained models, performance metrics, leakage impact analysis, `performance_summary_v2.json` (no verdict — NB02 must not author one) |
| 03_model_validation | Independent validation — **the authoritative verdict** | Validation report, risk rating (**6/10 → HIGH → REJECT**), 11-item findings register, region-level fairness validation, `performance_summary_v2.json` (with verdict, `authored_by: "NB03"`) |
| 04_governance_compliance_report | Regulatory mapping and compliance scorecard | Compliance report, gap analysis, `compliance_scorecard_v2.json` |
| 05_german_credit_cross_validation | Cross-population test of the fairness mitigation | Cross-validation comparison, `SMALL_DIRECTIONAL` generalization finding |

### How the notebooks fit together

**Notebooks 01 through 03 are the technical work:** prepare the data, build the models, validate them.

**Notebook 04 is the governance work:** map the findings to regulatory frameworks.

**Notebook 05 asks whether the fairness mitigation transfers** to a structurally different dataset.

**The verdict flows in one direction.** Notebook 03 (independent validation) owns the risk rating and recommendation. Notebook 04 inherits them — it does not re-derive them, because two notebooks reporting different verdicts on the same model would be a governance failure, not a feature.

The `performance_summary_v2.json` artifact is the contract between them:
- NB02 writes it with computed metrics but **no verdict**.
- NB03 asserts the file has no verdict, then **writes the verdict** (`risk_score`, `risk_rating`, `recommendation`) back to the same file, with `authored_by: "NB03"`.
- NB04 asserts `authored_by == "NB03"` before reading the verdict, and refuses to proceed if the assertion fails.

---

## Final recommendation

**HIGH risk rating. REJECT for deployment in current form.**

**Risk score:** 6/10 (components: performance 1, stability 0, calibration 2, fairness 3, leakage 0).

**Primary reason for rejection:** fairness (region-level DI = 0.5414, SEVERE). Not performance. AUC 0.631 is a MODERATE finding and contributes only 1 point; the score reaches HIGH because fairness (3 points) and calibration (2 points) dominate.

The model may be used for human-led early-warning collections prioritization, subject to quarterly fairness monitoring and monthly performance monitoring. It may not be deployed for:

- Fairness-sensitive decisions without further mitigation
- IFRS 9 provisioning
- Canadian portfolios
- Automated decisions without human oversight

**Full approval requires:**

1. Fairness remediation to DI ≥ 0.80 on a low-cardinality attribute (FD-05 compliant).
2. Validation on Canadian data.
3. Resolution of the calibration finding (HL implementation verified; poor models recalibrated or replaced).
4. Walk-forward split implemented, or a documented amendment to MD-001 with disclosure.
5. Investigation of the 4 counterintuitive SHAP features (F-03.07).

> **Read with:** `docs/remediation_plan.md` (each condition mapped to a finding ID, owner, deadline, success criterion).

---

## German Credit cross-validation (Notebook 05)

**Why a second dataset?** The Lending Club finding (region proxy for protected class → severe disparate impact) is dataset-specific. A validator should ask: *does the mitigation technique generalize?* German Credit is a structurally different dataset (1,000 records, 1970s–80s, direct `Sex` attribute — no geography). If removing proxy-like features improves fairness there too, the technique has some cross-population validity.

**What was tested.** Three candidate proxies: `Housing`, `Job`, `Purpose`. All three were tested (no silent coverage gaps).

| Dataset | Baseline DI | Status | Mitigation | Post-mitigation DI | Status | Δ DI |
|---|---|---|---|---|---|---|
| Lending Club | 0.5414 | SEVERE | Not applied in NB03 run | not_assessed | — | — |
| German Credit | 0.9375 | OK | Removed Housing | 0.9740 | OK | +0.0365 |

**Classification:** `SMALL_DIRECTIONAL`.

**What that means.** The technique improves DI in the *same direction* on both datasets, but the effect on German Credit is small relative to Lending Club. The study claims *direction*, not *magnitude*. Note also that the Lending Club mitigation was **not** applied in this run — the comparison tests the mitigation *technique* against the Lending Club *finding*, not a mitigation that was already applied.

> **Read with:** `outputs/reports/german_credit_cross_validation.json` and the German Credit section of `docs/remediation_plan.md`.

---

## Technical stack

- **Language:** Python 3.10+
- **Core:** pandas, numpy, scipy, scikit-learn
- **Models:** XGBoost, LightGBM, Random Forest, Logistic Regression
- **Explainability:** SHAP
- **Fairness:** fairlearn (German Credit); hand-computed DI/DPD/EOD (Lending Club)
- **Visualization:** matplotlib, seaborn
- **Governance artifacts:** JSON and CSV, each carrying `run_id` and `notebook_version` for traceability

### Reproducibility

Every notebook run produces:
- An audit log at `logs/*_audit_trail_<run_id>.log`
- A `run_id` (timestamp-based) on every artifact
- A SHA-256-16 hash on every saved file
- A `performance_summary_v2.json` with `authored_by` identifying which notebook last wrote the verdict

The same random seed (42) is used throughout. Same-data-same-seed → same-artifacts.

---

## Key documents

| Document | Description |
|---|---|
| `docs/intended_use_statement.md` | Model purpose, intended use, and documented limitations |
| `docs/data_dictionary.md` | Feature definitions, sources, and exclusion rationale |
| `docs/regulatory_mapping.md` | Full OSFI E-23 / SR 11-7 / ECOA / IFRS 9 mapping |
| `docs/remediation_plan.md` | Sequenced remediation actions with owners and timelines |
| `outputs/reports/validation_report_v2.txt` | Full independent validation report |
| `outputs/reports/compliance_report_v2.txt` | Governance and compliance report |
| `outputs/reports/validation_findings_register.csv` | 11-item findings register (machine-readable) |
| `outputs/reports/fairness_validation.csv` | Region-level fairness metrics |
| `outputs/reports/german_credit_cross_validation.json` | Cross-population generalization result |
| `outputs/reports/performance_summary_v2.json` | Authoritative validation verdict (`authored_by: "NB03"`) |
| `outputs/reports/compliance_scorecard_v2.json` | Governance scorecard |
| `outputs/reports/regulatory_mapping_v2.csv` | Consolidated compliance mapping |

---

## Getting started

### Installation

```bash
git clone https://github.com/yourusername/lending-club-model-validation.git
cd lending-club-model-validation
pip install -r requirements.txt
