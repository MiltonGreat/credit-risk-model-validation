# Lending Club Credit Risk Model Validation

## Project Overview

**Role:** GRC Risk Analyst
**Focus:** Independent model validation and regulatory compliance
**Regulatory Frameworks:** OSFI E-23, SR 11-7, ECOA/CFPB, IFRS 9, OSFI B-13

This project is an independent validation of a credit risk model developed on
Lending Club data. The validation follows OSFI E-23 and SR 11-7 principles,
assessing model performance, data quality, fairness, and regulatory compliance.
It is a portfolio artifact — the methodology and governance structure are the
point, not the specific findings on Lending Club data.

### Business problem

A first-line model development team built a credit risk model intended to
support early-warning collections prioritization (16+ days past due). This
validation addresses four questions:

1. Does the model perform as claimed?
2. Is it fair across protected groups?
3. Does it meet the regulatory requirements it will be held to?
4. Can it be deployed?

The short answers: not as claimed; no; partially; and not without remediation.
The long answers are in the notebooks.

### The analyst's role

As a GRC Risk Analyst, the work here is validation, not development:

- Independently assess the model built by the development team
- Identify and quantify material issues (data leakage, fairness, calibration)
- Determine the model's regulatory compliance status
- Produce a findings register with severity, owner, remediation, and status
- Report a risk rating and deployment recommendation

### What makes this a validation project, not a modeling project

The model exists. The question is whether it is fit for purpose. Everything in
Notebooks 03, 04, and 05 is written from the perspective of someone checking a
claim made by someone else — not someone building a model and reporting its
performance.

---

## Key findings

| Finding | Severity | Evidence | Status |
|---|---|---|---|
| **Data leakage — 9 features** | CRITICAL | AUC inflated from 0.621 to 0.865 (28.2%) | RESOLVED |
| **Severe disparate impact** | CRITICAL | DI 0.244 before mitigation; 0.798 after — still below 0.80 threshold | OPEN |
| **Random train/test split** | CRITICAL | No time-based split; look-ahead bias risk | OPEN |
| **IFRS 9 misalignment** | HIGH | 16+ day threshold vs. IFRS 9's 30+ day Stage 3 | OPEN |
| **Foreign data applicability** | HIGH | U.S. data only; not validated for Canadian portfolios | OPEN |
| **Poor calibration (XGBoost)** | HIGH | HL p-value = 0.0000 | OPEN |
| **No macroeconomic factors** | HIGH | Cannot capture economic cycle risk | OPEN |
| **Low precision** | MEDIUM | Precision 0.02–0.04 (high false positive rate) | OPEN |
| **Protected attribute proxy** | MEDIUM | State used as geographic proxy for race/ethnicity | OPEN |

**Findings summary:** 12 total — 4 CRITICAL, 4 HIGH, 4 MEDIUM. 2 resolved, 10 open.

### The two findings that matter most

**Data leakage (resolved).** Nine features — including balance, payment amounts,
and months-since-delinquency — contained post-origination information that would
not exist at the moment of a real credit decision. Removing them dropped the
model's AUC from an inflated 0.865 to a realistic 0.621. The original performance
claim was invalid.

**Severe disparate impact (open).** The model approved applicants at substantially
different rates across the states tested, with a minimum Disparate Impact ratio
of 0.244 — well below the 0.80 threshold. The mitigation (removing state features)
improved DI to 0.798. That is an improvement of +0.554 DI points, but it does not
cross the threshold: 0.798 is still below 0.80. Deployment is blocked until the
finding is closed.

### Fairness improvement

| Metric | Before mitigation | After mitigation | Threshold |
|---|---|---|---|
| Disparate Impact Ratio | 0.244 (SEVERE) | 0.798 (MONITOR) | ≥ 0.80 |
| Features removed | — | 49 state features | — |
| Status | Deployment blocked | Deployment still blocked | — |

### Final recommendation

> **HIGH risk rating. REJECT for deployment in current form.**
>
> The model may be used for human-led early-warning collections prioritization,
> subject to quarterly fairness monitoring and monthly performance monitoring.
> It may **not** be deployed for:
>
> - Fairness-sensitive decisions without further mitigation
> - IFRS 9 provisioning
> - Canadian portfolios
> - Automated decisions without human oversight
>
> Full approval requires: fairness remediation to DI ≥ 0.80, validation on
> Canadian data, and resolution of the calibration finding.

---

## Regulatory compliance status

The compliance assessment reflects the corrected counting logic in Notebook 04:
only requirements with status exactly "COMPLIANT" are counted as compliant.
Partially compliant entries are treated as not-yet-compliant — the conservative
reading appropriate for a governance report.

### OSFI E-23 compliance

| Section | Status | Gap |
|---|---|---|
| 3.1 — Model Validation | COMPLIANT | Foreign data not validated for Canada |
| 3.2 — Performance Monitoring | PARTIALLY COMPLIANT | Monthly monitoring framework not yet established |
| 3.3 — Data Quality | COMPLIANT | Macroeconomic factors missing |
| 4.1 — Model Documentation | COMPLIANT | — |
| 4.2 — Model Governance | COMPLIANT | — |
| 5.1 — Model Risk | COMPLIANT | — |
| 5.2 — Fairness | **NON-COMPLIANT** | Severe disparate impact (DI 0.798 < 0.80) |

**Rate: 4 of 7 fully compliant (57%).**

### SR 11-7 compliance

| Principle | Status | Gap |
|---|---|---|
| 1 — Validation | COMPLIANT | — |
| 2 — Ongoing Monitoring | PARTIALLY COMPLIANT | Monthly tracking not established |
| 3 — Model Documentation | COMPLIANT | — |
| 4 — Model Risk | COMPLIANT | — |
| 5 — Fairness | **NON-COMPLIANT** | Severe disparate impact |
| 6 — Data Quality | COMPLIANT | Macroeconomic factors missing |

**Rate: 4 of 6 fully compliant (67%).**

### ECOA/CFPB fair lending

| Requirement | Status | Gap |
|---|---|---|
| Disparate impact testing | **NON-COMPLIANT** | DI 0.798 < 0.80 |
| Protected attribute analysis | PARTIALLY COMPLIANT | State proxy only; no explicit race/gender data |

**Rate: 0 of 2 fully compliant (0%).**

### IFRS 9

| Requirement | Status | Gap |
|---|---|---|
| Stage 3 classification (30+ days) | **NON-COMPLIANT** | Model uses 16+ day threshold |
| Provisioning (ECL calculation) | **NON-COMPLIANT** | Model not designed for provisioning |

**Rate: 0 of 2 fully compliant (0%).**

---

## Project structure

| Notebook | Purpose | Key output |
|---|---|---|
| `01_exploratory_data_analysis` | Data quality, leakage detection, feature inventory | Data quality report, findings register, temporal audit |
| `02_model_development` | Model training, leakage quantification, fairness testing | Trained models, performance metrics, leakage impact analysis |
| `03_model_validation` | Independent validation — the authoritative verdict | Validation report, risk rating, 12-item findings register |
| `04_governance_compliance_report` | Regulatory mapping and compliance scorecard | Compliance report, gap analysis, decision log |
| `05_german_credit_cross_validation` | Cross-population test of the fairness mitigation | Cross-validation comparison, generalization finding |

### How the notebooks fit together

Notebooks 01 through 03 are the technical work: prepare the data, build the
models, validate them. Notebook 04 is the governance work: map the findings to
regulatory frameworks. Notebook 05 asks whether the fairness mitigation
transfers to a structurally different dataset.

The **verdict flows in one direction.** Notebook 03 (independent validation)
owns the risk rating and recommendation. Notebook 04 inherits them — it does
not re-derive them, because two notebooks reporting different verdicts on the
same model would be a governance failure, not a feature.

---

## Technical stack

- **Python 3.10+** with pandas, numpy, scikit-learn, scipy
- **Machine learning:** XGBoost, LightGBM, Random Forest, Logistic Regression
- **Explainability:** SHAP
- **Fairness:** fairlearn (cross-validation notebook); hand-computed DI/DPD/EOD
- **Visualization:** matplotlib, seaborn
- **Governance:** JSON and CSV artifacts for downstream consumption

---

## Model performance

Realistic performance after removal of the nine leakage features:

| Model | AUC-ROC | Precision | Recall | Calibration (HL p-value) |
|---|---|---|---|---|
| XGBoost | 0.621 | 0.032 | 0.500 | 0.0000 (POOR) |
| LightGBM | 0.613 | 0.024 | 0.778 | 0.2972 (OK) |
| Random Forest | 0.611 | 0.042 | 0.361 | 1.0000 (GOOD) |
| Logistic Regression | 0.589 | 0.035 | 0.389 | 1.0000 (GOOD) |

**Best model by held-out AUC:** XGBoost (0.621) — also the worst-calibrated.
This tension is documented as Finding 7 in the validation register.

**KS statistic (XGBoost):** 0.2195 — weak discrimination by conventional thresholds.

---

## Key documents

| Document | Description |
|---|---|
| [`docs/intended_use_statement.md`](./docs/intended_use_statement.md) | Model purpose, intended use, and documented limitations |
| [`docs/data_dictionary.md`](./docs/data_dictionary.md) | Feature definitions, sources, and exclusion rationale |
| [`docs/regulatory_mapping.md`](./docs/regulatory_mapping.md) | Full OSFI E-23 / SR 11-7 / ECOA / IFRS 9 mapping |
| [`docs/remediation_plan.md`](./docs/remediation_plan.md) | Sequenced remediation actions with owners and timelines |
| [`outputs/reports/validation_report_v2.txt`](./outputs/reports/validation_report_v2.txt) | Full independent validation report |
| [`outputs/reports/compliance_report_v2.txt`](./outputs/reports/compliance_report_v2.txt) | Governance and compliance report |
| [`outputs/reports/validation_findings_register.csv`](./outputs/reports/validation_findings_register.csv) | 12-item findings register (machine-readable) |
| [`outputs/reports/performance_summary_v2.json`](./outputs/reports/performance_summary_v2.json) | Authoritative validation verdict |

---

## Getting started

### Installation

```bash
git clone https://github.com/yourusername/lending-club-model-validation.git
cd lending-club-model-validation
pip install -r requirements.txt
