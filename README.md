# Lending Club Credit Risk Model Validation

**An independent validation that caught a 27% performance inflation and a severe fairness defect — before deployment.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![OSFI E-23](https://img.shields.io/badge/OSFI%20E--23-Aligned-lightgrey)
![SR 11-7](https://img.shields.io/badge/SR%2011--7-Partially%20Aligned-yellow)
![Verdict](https://img.shields.io/badge/Verdict-REJECT-red)
![Status](https://img.shields.io/badge/Status-Portfolio%20Artifact-lightgrey)

---

## The business problem

A first-line model development team built a consumer credit risk model for early-warning collections prioritization (16+ days past due). They reported **AUC 0.865** and requested approval for deployment.

Before a model can be deployed in a regulated lending environment, an independent validator must answer four questions:

1. **Does the model perform as claimed?**
2. **Is it fair across protected groups?**
3. **Does it meet the regulatory requirements it will be held to?**
4. **Can it be deployed?**

This project answers all four. The short answers are **not as claimed; no; partially; and not without remediation.**

---

## The two findings that mattered most

### 1. The reported performance was inflated by 27%

Nine features in the training data — including `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, and `issue_month` — contain **post-origination information**. When a bank decides whether to approve a loan, it does not know whether the borrower will pay late next month or charge off in two years. The nine features are exactly that future information.

Including them in training teaches the model to "predict" the past. When they were removed, the model's reported AUC dropped from **0.865 to 0.631** — a **27.1% inflation** of measured performance.

**Why this matters in business terms.** A model deployed against the inflated metric would have looked like it performed well in validation and then silently underperformed in production. Collections teams would have prioritized the wrong accounts. The bank would have spent months trying to understand why the model's real-world results didn't match its validation.

**Status:** CRITICAL, **resolved** (leakage features removed in the pipeline).

### 2. The model approved applicants at materially different rates across U.S. regions

The model approves applicants at substantially different rates across U.S. census-style regions. Region is the geographic proxy for protected-class composition in this dataset — the data has no direct race or ethnicity field.

**The measurement.** The Disparate Impact ratio (DI) compares the lowest-approval group's approval rate to the highest. The 4/5ths rule (the industry standard) requires DI ≥ 0.80 for a group not to be flagged. After aggregating the 50 U.S. states into four regions (South, West, Northeast, Midwest), **DI was 0.5414** — below 0.80 and inside the "SEVERE" band (DI < 0.60).

**Why this matters in business terms.** A bank deploying this model without mitigation would face fair-lending exposure. The finding blocks deployment until the disparity is remediated — and remediation is not trivial: the technique tested (proxy-feature removal) improves DI directionally but the effect is small (Δ DI +0.036 in the German Credit cross-validation).

**Status:** CRITICAL, **open**. This is the primary driver of the REJECT verdict.

---

## The verdict

**6/10 → HIGH → REJECT for deployment in current form.**

| Component | Score | Basis |
|---|---|---|
| Performance | 1 / 3 | AUC 0.631, MODERATE band |
| Stability | 0 / 2 | PSI not assessed (random split) |
| Calibration | 2 / 2 | XGBoost poor |
| **Fairness** | **3 / 3** | **DI 0.5414, SEVERE** |
| Leakage | 0 / 3 | Resolved |

**The rejection is fairness-driven, not performance-driven.** AUC 0.631 is a MODERATE finding and contributes only 1 of the 6 points. The score reaches HIGH because fairness (3 points) and calibration (2 points) dominate. A reader who sees "AUC 0.631, REJECT" and concludes performance was the driver has misread the verdict — the README says this explicitly because the misreading is likely.

**What the model may be used for:** human-led early-warning collections prioritization, subject to quarterly fairness monitoring and monthly performance monitoring.

**What it may not be used for:** fairness-sensitive decisions without further mitigation, IFRS 9 provisioning, Canadian portfolios, or automated decisions without human oversight.

**Full approval requires:**
1. Fairness remediation to DI ≥ 0.80 on a low-cardinality attribute (FD-05 compliant).
2. Validation on Canadian data.
3. Calibration finding resolved (HL implementation verified; poor models recalibrated or replaced).
4. Walk-forward split implemented, or a documented DCR amendment.
5. Investigation of four counterintuitive SHAP features (F-03.07).

> **Read with:** [`docs/remediation_plan.md`](docs/remediation_plan.md) — each condition mapped to a finding ID, owner, deadline, and success criterion.

---

## How to read this project

If you're new to model validation, this section explains what the project is doing.

**The setup.** A model development team built a credit risk model to support early-warning collections prioritization — identifying which accounts are likely to default in time to intervene. They reported strong performance and requested approval for deployment.

**The validation.** My job is not to build a model. My job is to answer four questions independently: Does the model perform as claimed? Is it fair? Does it meet regulatory requirements? Can it be deployed?

**The governance discipline.** The verdict lives in **Notebook 03 (independent validation)**. Notebook 04 (governance) inherits that verdict — it does not re-derive it. Two notebooks reporting different verdicts on the same model would be a governance failure, not a feature. The `performance_summary_v2.json` artifact records which notebook authored the verdict (`authored_by: "NB03"`) so the chain is auditable.

---

## The analyst's role

As a GRC Risk Analyst, the work here is validation, not development:

- **Independently assess** the model built by the development team.
- **Identify and quantify** material issues (data leakage, fairness, calibration).
- **Determine** the model's regulatory compliance status.
- **Produce** a findings register with severity, owner, remediation, and status.
- **Report** a risk rating and deployment recommendation.

**What makes this a validation project, not a modeling project.** The model exists. The question is whether it is fit for purpose. Everything in Notebooks 03, 04, and 05 is written from the perspective of someone checking a claim made by someone else — not someone building a model and reporting its performance. The notebooks do not produce a narrative about the developer. They compute the numbers and export them. The verdict is a coded risk rating with a coded recommendation. The prose interpretation — "the developer's claim was overstated" — is authored by me, in the Validation Opinion, not by the notebooks.

---

## Key findings

Each finding below traces to a `Finding ID` in the machine-readable register (`outputs/reports/validation_findings_register.csv`) and to a specific notebook phase.

| ID | Finding | Severity | What it means in plain terms | Status |
|---|---|---|---|---|
| F-03.01 | **Data leakage — 9 features** | CRITICAL | The model was using information that does not exist at the moment a credit decision has to be made. The originally-reported performance was measuring the wrong thing. | RESOLVED |
| F-03.03 | **Severe disparate impact** | CRITICAL | Approval rates differed materially across U.S. regions. A group in one region was approved at 54% of the rate of the highest-approval region — below the 80% threshold. Deployment is blocked until remediated. | OPEN |
| F-03.02 | **Walk-forward split not implemented** | HIGH | The model was trained on a random sample rather than "past loans predict future loans." This can inflate measured performance and masks behavior on newer borrowers. | OPEN |
| F-03.10 | **IFRS 9 misalignment** | HIGH | Model defines "default" as 16+ days past due. IFRS 9 requires 30+ days for Stage 3. Cannot be used for IFRS 9 provisioning without recalibration. | OPEN |
| F-03.09 | **Foreign data applicability** | HIGH | Trained on U.S. Lending Club data. Not validated on Canadian portfolios. Cannot be deployed in Canada without revalidation. | OPEN |
| F-03.04 | **Poor calibration (XGBoost)** | HIGH | Predicted probabilities are not reliable. A model that says "30% chance of default" does not mean 30% of such borrowers default. | OPEN |
| F-03.08 | **No macroeconomic factors** | HIGH | The model does not account for the state of the economy. Cannot anticipate how a recession changes default behavior. | OPEN |
| F-03.06 | **Weak discrimination (KS = 0.267)** | HIGH | The model is weak at separating likely defaulters from likely non-defaulters in the top tail. | OPEN |
| F-03.07 | **Counterintuitive SHAP relationships** | HIGH | Four of the model's top-10 drivers behave in the opposite direction from credit-risk theory. Requires investigation before drivers can be trusted. | OPEN |
| F-03.11 | **Low precision (0.024–0.085)** | MEDIUM | When the model flags an account as "will default," it is wrong most of the time. May be acceptable for prioritization if recall is high. | OPEN |
| F-03.05 | **Suspicious HL p-values (3 models)** | MEDIUM | HL calibration test returned p-values of exactly 0 or 1 for three of four models. Known degeneracy pattern. Needs HL implementation verified. | OPEN |
| F-03.12 | **Protected attribute is a proxy** | MEDIUM | Dataset has no direct race/ethnicity field. Region used as a proxy. Proxy-based fairness findings are indicative, not definitive. | OPEN |

**Findings summary:** 11 total — 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 resolved, 10 open.

> **Read with:** `outputs/reports/validation_findings_register.csv` (machine-readable) and `outputs/reports/validation_report_v2.txt` (narrative).

---

## Model performance

Best model per held-out AUC, after removal of the nine leakage features.

| Model | AUC-ROC | Precision | Recall | Brier | HL p-value | Calibration |
|---|---|---|---|---|---|---|
| **Random Forest** | **0.631** | 0.085 | 0.333 | 0.127 | 1.000 (suspicious) | GOOD* |
| XGBoost | 0.604 | 0.029 | 0.444 | 0.070 | 0.000 (POOR) | POOR |
| LightGBM | 0.621 | 0.029 | 0.694 | 0.084 | 0.729 | GOOD |
| Logistic Regression | 0.536 | 0.039 | 0.333 | 0.220 | 1.000 (suspicious) | GOOD* |

\* HL p-value of exactly 1.000 is a known degeneracy pattern, flagged as Finding F-03.05. The calibration for those models cannot be trusted until the HL implementation is verified.

**Best by held-out AUC:** Random Forest (0.6307). **Developer's original claim:** AUC 0.865 (with leakage). **Realistic performance:** AUC 0.631. **Drop:** 0.234 AUC points (27.1%). **KS:** 0.2671 — weak discrimination by conventional thresholds.

---

## Regulatory compliance status

The compliance assessment reflects Notebook 04's exact-match counting: only requirements with status exactly `COMPLIANT` are counted. Partially compliant entries are treated as not-yet-compliant — the conservative reading appropriate for a governance report.

### OSFI E-23 (5 of 7 fully compliant, 71%)

| Section | Status | Gap |
|---|---|---|
| 3.1 — Model Validation | COMPLIANT | Foreign data not validated for Canada |
| 3.2 — Performance Monitoring | PARTIALLY COMPLIANT | Monthly monitoring framework not established |
| 3.3 — Data Quality | COMPLIANT | Macroeconomic factors missing |
| 4.1 — Model Documentation | COMPLIANT | — |
| 4.2 — Model Governance | COMPLIANT | — |
| 5.1 — Model Risk | COMPLIANT | — |
| 5.2 — Fairness | **NON-COMPLIANT** | Severe disparate impact (DI 0.5414 < 0.60) |

### SR 11-7 (4 of 6 fully compliant, 67%)

| Principle | Status | Gap |
|---|---|---|
| 1 — Validation | COMPLIANT | — |
| 2 — Ongoing Monitoring | PARTIALLY COMPLIANT | Monthly tracking not established |
| 3 — Model Documentation | COMPLIANT | — |
| 4 — Model Risk | COMPLIANT | — |
| 5 — Fairness | **NON-COMPLIANT** | Severe disparate impact |
| 6 — Data Quality | COMPLIANT | Macroeconomic factors missing |

### ECOA/CFPB (0 of 2 fully compliant)

| Requirement | Status | Gap |
|---|---|---|
| Disparate impact testing | **NON-COMPLIANT** | DI 0.5414 < 0.80 |
| Protected attribute analysis | PARTIALLY COMPLIANT | Region proxy only |

### IFRS 9 (0 of 2 fully compliant)

| Requirement | Status | Gap |
|---|---|---|
| Stage 3 classification (30+ days) | **NON-COMPLIANT** | Model uses 16+ day threshold |
| Provisioning (ECL calculation) | **NON-COMPLIANT** | Model not designed for provisioning |

> **Read with:** `outputs/reports/regulatory_mapping_v2.csv` and `outputs/reports/compliance_report_v2.txt`.

---

## Project structure

| Notebook | Purpose | Key output |
|---|---|---|
| 01_exploratory_data_analysis | Data quality, leakage detection, feature inventory | Data quality summary, NB01 findings register, temporal audit, feature inventory |
| 02_model_development | Model training, leakage quantification, fairness diagnostic | Trained models, performance metrics, leakage impact, `performance_summary_v2.json` (no verdict — NB02 must not author one) |
| 03_model_validation | Independent validation — **the authoritative verdict** | Validation report, risk rating (**6/10 → HIGH → REJECT**), 11-item findings register, region-level fairness validation, `performance_summary_v2.json` (with verdict, `authored_by: "NB03"`) |
| 04_governance_compliance_report | Regulatory mapping and compliance scorecard | Compliance report, gap analysis, `compliance_scorecard_v2.json` |
| 05_german_credit_cross_validation | Cross-population test of the fairness mitigation | Cross-validation comparison, `SMALL_DIRECTIONAL` generalization finding |

### How the notebooks fit together

**Notebooks 01 through 03 are the technical work.** **Notebook 04 is the governance work.** **Notebook 05 asks whether the fairness mitigation transfers to a structurally different dataset.**

**The verdict flows in one direction.** Notebook 03 (independent validation) owns the risk rating and recommendation. Notebook 04 inherits them — it does not re-derive them, because two notebooks reporting different verdicts on the same model would be a governance failure, not a feature. The `performance_summary_v2.json` artifact is the contract between them: NB02 writes it with computed metrics but **no verdict**; NB03 asserts the file has no verdict, then **writes the verdict** with `authored_by: "NB03"`; NB04 asserts `authored_by == "NB03"` before reading.

---

## German Credit cross-validation (Notebook 05)

**Why a second dataset?** The Lending Club finding (region proxy for protected class → severe disparate impact) is dataset-specific. A validator should ask: does the mitigation technique generalize? German Credit is a structurally different dataset (1,000 records, 1970s–80s, direct `Sex` attribute — no geography). If removing proxy-like features improves fairness there too, the technique has some cross-population validity.

**What was tested.** Three candidate proxies: `Housing`, `Job`, `Purpose`. All three were tested (no silent coverage gaps).

| Dataset | Baseline DI | Status | Mitigation | Post-mitigation DI | Status | Δ DI |
|---|---|---|---|---|---|---|
| Lending Club | 0.5414 | SEVERE | Not applied in NB03 | not_assessed | — | — |
| German Credit | 0.9375 | OK | Removed Housing | 0.9740 | OK | +0.0365 |

**Classification:** `SMALL_DIRECTIONAL`. The technique improves DI in the *same direction* on both datasets, but the effect on German Credit is small relative to Lending Club. The study claims *direction*, not *magnitude*. Note also that the Lending Club mitigation was **not** applied in this run — the comparison tests the mitigation *technique* against the Lending Club *finding*, not a mitigation that was already applied.

> **Read with:** `outputs/reports/german_credit_cross_validation.json`.

---

## Technical stack

- **Language:** Python 3.10+
- **Core:** pandas, numpy, scipy, scikit-learn
- **Models:** XGBoost, LightGBM, Random Forest, Logistic Regression
- **Explainability:** SHAP
- **Fairness:** fairlearn (German Credit); hand-computed DI/DPD/EOD (Lending Club)
- **Visualization:** matplotlib, seaborn
- **Governance artifacts:** JSON and CSV, each carrying `run_id` and `notebook_version`

### Reproducibility

Every notebook run produces an audit log, a `run_id` on every artifact, a SHA-256-16 hash on every saved file, and a `performance_summary_v2.json` with `authored_by` identifying which notebook last wrote the verdict. The same random seed (42) is used throughout. Same data + same seed → same artifacts.

---

## Key documents

| Document | Description |
|---|---|
| `docs/intended_use_statement.md` | Model purpose, intended use, documented limitations |
| `docs/data_dictionary.md` | Feature definitions, sources, exclusion rationale |
| `docs/regulatory_mapping.md` | Full OSFI E-23 / SR 11-7 / ECOA / IFRS 9 mapping |
| `docs/remediation_plan.md` | Sequenced remediation actions with owners and timelines |
| `outputs/reports/validation_report_v2.txt` | Full independent validation report |
| `outputs/reports/validation_findings_register.csv` | 11-item findings register (machine-readable) |
| `outputs/reports/fairness_validation.csv` | Region-level fairness metrics |
| `outputs/reports/performance_summary_v2.json` | Authoritative validation verdict (`authored_by: "NB03"`) |

---

## Related repositories

- [model-analysis-package](https://github.com/MiltonGreat/model-analysis-package) — the MAP anchor: five governance deliverables, including this project's Validation Report
- [fannie-mae-ifrs9-framework](https://github.com/MiltonGreat/fannie-mae-ifrs9-portfolio/tree/main) — the companion IFRS 9 stress-testing framework

## License

MIT
