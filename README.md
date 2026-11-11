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

Nine features in the training data — `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, and `issue_month` — contain **post-origination information**. When a bank decides whether to approve a loan, it does not know whether the borrower will pay late next month or charge off in two years. The nine features are exactly that future information.

Including them in training teaches the model to "predict" the past. When they were removed, the model's held-out AUC dropped from **0.865 to 0.631** — a **27.1% inflation** of measured performance (23.4 AUC percentage points).

**Why this matters in business terms.** A model deployed against the inflated metric would have looked like it performed well in validation and then silently underperformed in production. Collections teams would have prioritized the wrong accounts. The bank would have spent months trying to understand why the model's real-world results didn't match its validation.

**What the drop does not mean.** It does not mean the developer lied or made an error. The 0.865 is the *leakage-included* AUC — what the model would achieve if the leakage features were available at decision time. The 0.631 is the *leakage-free* AUC — what the model achieves when the leakage features are removed. **Both numbers are correct; they measure different models.** The 0.865 is what the model *could* do with future knowledge; the 0.631 is what it *can* do without.

**Status:** CRITICAL, **resolved** (leakage features removed in the pipeline; independently recomputed in NB03).

### 2. The model approved applicants at materially different rates across U.S. regions

The model approves applicants at substantially different rates across U.S. census-style regions. Region is the geographic proxy for protected-class composition in this dataset — the data has no direct race or ethnicity field.

**The measurement.** The Disparate Impact ratio (DI) compares the lowest-approval group's approval rate to the highest. The 4/5ths rule (the industry standard) requires DI ≥ 0.80 for a group not to be flagged. After aggregating the 50 U.S. states into four regions (South, West, Northeast, Midwest), **DI was 0.5414** — below 0.80 and inside the "SEVERE" band (DI < 0.60).

**Why per-state DI would be invalid.** The 4/5ths rule is designed for binary or low-cardinality protected attributes. Applied to 50 one-hot state columns, it produces false positives (sampling noise at cardinality 50) and false negatives (a genuinely disadvantaged state masked by the maximum-approval state). The FD-05 cardinality check (≤ 50 groups) is the gate; the region mapping is the mechanism. **The 0.5414 region-level finding is the correct fairness finding; per-state values would be statistical artifacts of the rule's misapplication.**

**Why this matters in business terms.** A bank deploying this model without mitigation would face fair-lending exposure. The finding blocks deployment until the disparity is remediated.

**What the German Credit cross-validation showed about remediation.** The technique tested (proxy-feature removal) improves DI *in direction* on German Credit (Δ DI +0.0365, SMALL_DIRECTIONAL), but **the effect is small and the comparison is directional only** — see the cross-validation section below. The cross-validation does *not* establish that proxy removal would fix the Lending Club's SEVERE disparity.

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

**The rejection is fairness-driven, not performance-driven.** AUC 0.631 is a MODERATE finding and contributes only 1 of the 6 points. The score reaches HIGH because fairness (3 points) and calibration (2 points) dominate. **A reader who sees "AUC 0.631, REJECT" and concludes performance was the driver has misread the verdict.**

The arithmetic is what makes this clear. If fairness were OK (DI ≥ 0.80), the score would be 3 → MODERATE → APPROVE WITH CONDITIONS. The rejection would not occur. The fairness finding alone is sufficient to reject; the performance finding is not.

**What the model may be used for:** human-led early-warning collections prioritization, subject to quarterly fairness monitoring and monthly performance monitoring.

**What it may not be used for:** fairness-sensitive decisions without further mitigation, IFRS 9 provisioning, Canadian portfolios, or automated decisions without human oversight.

**Full approval requires:**
1. Fairness remediation to DI ≥ 0.80 on a low-cardinality attribute (FD-05 compliant).
2. Validation on Canadian data.
3. Calibration finding resolved (HL implementation verified; poor models recalibrated or replaced).
4. Walk-forward split implemented, or a documented DCR amendment.
5. Investigation of four counterintuitive SHAP features (F-03.07).

> **Read with:** [`governance/remediation-plan.md`](governance/remediation-plan.md) — each condition mapped to a finding ID, owner, deadline, and success criterion.

---

## How to read this project

If you're new to model validation, this section explains what the project is doing.

**The setup.** A model development team built a credit risk model to support early-warning collections prioritization — identifying which accounts are likely to default in time to intervene. They reported strong performance and requested approval for deployment.

**The validation.** My job is not to build a model. My job is to answer four questions independently: Does the model perform as claimed? Is it fair? Does it meet regulatory requirements? Can it be deployed?

**The governance discipline.** The verdict lives in **Notebook 03 (independent validation)**. Notebook 04 (governance) inherits that verdict — it does not re-derive it. Two notebooks reporting different verdicts on the same model would be a governance failure, not a feature.

**The chain of enforcement is explicit at three points:**

| Point | Notebook | What it checks | What happens on failure |
|---|---|---|---|
| NB02 writes | NB02 | `does_not_contain: ["risk_score", "risk_rating", "recommendation"]` | NB02's self-check rejects the write |
| NB03 reads | NB03 | `any(k in nb02_summary for k in (...))` | `ValueError`, NB03 does not proceed |
| NB04 reads | NB04 | `authored_by == "NB03"` and `sum(risk_score_components.values()) == risk_score` | `AssertionError`, NB04 does not proceed |

**Why three checks instead of one.** Each check catches a different failure mode. The NB02 self-check catches the accidental write at source. The NB03 read check catches the case where NB02's self-check was bypassed (a manually edited JSON, or a stale file from an earlier version). The NB04 read check catches the case where NB03's own writing was incomplete or corrupted. **A single check would leave a gap; three checks close it.**

**What strict inheritance means.** NB04 reads NB03's verdict verbatim and translates it into regulatory frameworks. It does not recompute. If NB04 recomputed and got a different number, the pipeline would have two conflicting verdicts. The `authored_by == "NB03"` assertion prevents NB04 from being a second verdict authority. **NB04 is not a second validator — it is a consistency-checked reporter.**

---

## The analyst's role

As a GRC Risk Analyst, the work here is validation, not development:

- **Independently assess** the model built by the development team.
- **Identify and quantify** material issues (data leakage, fairness, calibration).
- **Determine** the model's regulatory compliance status.
- **Produce** a findings register with severity, owner, remediation, and status.
- **Report** a risk rating and deployment recommendation.

**What makes this a validation project, not a modeling project.** The model exists. The question is whether it is fit for purpose. Everything in Notebooks 03, 04, and 05 is written from the perspective of someone checking a claim made by someone else — not someone building a model and reporting its performance. The notebooks do not produce a narrative about the developer. They compute the numbers and export them. The verdict is a coded risk rating with a coded recommendation. The prose interpretation — "the developer's claim was overstated" — is authored by me, in the Validation Opinion, not by the notebooks.

**What independent re-computation looks like in practice.** At NB03 Phase 4, the notebook loads NB02's predictions but **recomputes** the AUC from its own `y_val` and the loaded predictions. It does not copy `nb02_summary["realistic_auc"]`. The recompute is the independent-validation move — the difference between *"NB02's number is 0.631"* (a claim) and *"NB03 computed 0.631 from the same predictions and labels"* (a finding about NB02's implementation). If the two disagree by more than 1e-6, NB03 logs a WARNING and proceeds with the recomputed value.

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

**Findings summary (NB04 confirmed output):** 11 total — 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 resolved, 10 open.

> **Read with:** `outputs/reports/validation_findings_register.csv` (machine-readable) and `outputs/reports/validation_report_v2.txt` (narrative).

### About the HL degeneracy (F-03.05)

The Hosmer–Lemeshow test has a known degeneracy pattern: p-values of exactly 0 or 1 are **numerical artifacts** of the test's chi-square computation when a decile has zero observed events or zero predicted probability. They are not valid test results, and they do not distinguish **two failure modes**: (a) the model is genuinely poorly calibrated, or (b) the test itself is degenerate and the model's calibration cannot be assessed from this test. The notebook's "SUSPICIOUS" label is correct; the right response is to look at Brier and the calibration curve (which do not degenerate), not to conclude the model is badly calibrated.

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

### About the class imbalance

The default rate in this dataset is ~1.78% (NB01 confirmed). A trivial model that predicts "performing" for every loan achieves 98.2% accuracy and zero utility. The `class_weight='balanced'` and `scale_pos_weight` parameters are what prevent the model from collapsing to the majority class. A reader who sees high accuracy in a future run and AUC near 0.50 has seen the model collapse — the class-weight parameters are what make the model useful. **The metrics reported are AUC (threshold-independent), precision, recall, and Brier — none of which can be gamed by the trivial model.**

### About the comparison to the developer's claim

The 27.1% inflation is a **percentage of the claimed value** (0.234 / 0.865). It is not a percentage-point difference — the two are different claims. Both are correct; the percentage-of-claimed form is more dramatic (27.1% vs. 23.4 pp). The comparison is to the developer's claim, not to a benchmark. **The notebook does not compare 0.631 to a reasonable alternative** (a logistic regression baseline, published Lending Club benchmarks, an industry median). That comparison would tell the reader whether 0.631 is *good enough* for the model's intended use, not just whether it is *worse than the inflated claim*. It is a real analytical gap, and it belongs to the Validation Opinion, not to the notebook.

---

## Regulatory compliance status

The compliance assessment reflects Notebook 04's exact-match counting: only requirements with status exactly `COMPLIANT` are counted. Partially compliant entries are treated as not-yet-compliant — the conservative reading appropriate for a governance report.

### Reading "0 of 2 (0%) compliant"

A reader seeing "0% compliant" for ECOA and IFRS 9 might conclude the model is a complete failure on both frameworks. The correct reading is more nuanced.

**ECOA 0/2 decomposes to:**
- *Fair Lending* — NON-COMPLIANT (because fairness is SEVERE). This is the model's failure.
- *Protected Classes* — PARTIALLY COMPLIANT (region proxy, not direct protected attribute). This is the data's limitation.

The two are different kinds of non-compliance, with different remediations: the first needs fairness mitigation; the second needs better data.

**IFRS 9 0/2 decomposes to:**
- *Staging* — NON-COMPLIANT (16+ days vs. 30+ days).
- *Provisioning* — NON-COMPLIANT (model not designed for provisioning).

Both are **scope restrictions, not defects.** The model is a collections prioritization model, not an IFRS 9 provisioning model. **The IFRS 9 non-compliance is a scoping statement, not a failure.** The model correctly does not claim to be an IFRS 9 model.

**The percentage is not the story; the decomposition is.**

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

### About the dynamic regulatory mapping

The fairness statuses in these tables are **computed at runtime** from `fairness_category`. A static mapping would be a claim about a *previous* run; the dynamic mapping is a claim about *this* run. If the fairness finding were later resolved (e.g., by feature engineering in a future version), the status would change to COMPLIANT — because it is computed from the finding, not written in prose.

The same dynamic discipline applies to PSI `not_assessed` → Section 3.2 PARTIALLY COMPLIANT and to the split-method gap. **Every status in the regulatory mapping is derived from an artifact, not from an assertion.**

---

## What the compliance scorecard means (and does not mean)

The compliance scorecard translates the run's metrics into a single weighted number: **74.1/100 → MODERATE**.

| Category | Score | Weight | Derivation |
|---|---|---|---|
| Data Quality | 58.9 | 20% | Direct read of NB01's data quality score |
| Model Performance | 97.0 | 25% | `100 × (AUC / 0.65)`, capped at 100 |
| Documentation | 85.0 | 20% | Analyst judgment |
| Fairness/Bias | 54.1 | 20% | `100 × DI`, capped at 100 |
| Risk Management | 68.0 | 15% | Analyst judgment |

Two of the five scores are formulas; three are analyst judgment. **The weights themselves are analyst judgment.** The composite is a **blended artifact**: part metric, part judgment, weighted by judgment. **The weighted score is not a regulatory rating.** It is a governance summary metric — a single number a model risk committee can track across periods.

**Why the score can be MODERATE while the verdict is REJECT.** The verdict is HIGH → REJECT because of one SEVERE finding (fairness). The compliance score is MODERATE because the **other four categories are relatively strong**. The weighted average pulls toward MODERATE even though the fairness category (54.1) is well below the threshold. **A model can be REJECT for deployment and still score MODERATE on a composite governance metric.** A reader who equates the two is reading the wrong number.

**The composite hides the fairness problem; the individual categories expose it.** That is why the scorecard shows both.

---

## Two ratings, two questions

The pipeline produces **two ratings for the same model**:

| Rating | Value | Question it answers | Who uses it |
|---|---|---|---|
| **Validation verdict** | 6/10 → HIGH → REJECT | *Is this model fit for deployment as-is?* | The deployment decision-maker |
| **Governance-local risk** | 65.5/100 → MODERATE | *How much ongoing governance attention does this model require?* | The model risk committee's tracking function |

**These are different questions.** The verdict is a binary deployability determination. The governance-local risk is a composite of category scores (Data 75, Model 70, Implementation 72, Regulatory 45) that answers a fuzzier question: *what is the overall picture of this model's risk profile, for the purpose of setting monitoring intensity?*

**Why they can differ in direction.** A model with a severe defect in one dimension (fairness) but otherwise fine governance will show HIGH on the deployability verdict (the defect blocks deployment) and MODERATE on the governance-local risk (most of the governance profile is fine). **Both readings are correct.**

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

**The verdict flows in one direction.** NB02 writes `performance_summary_v2.json` with computed metrics but **no verdict**. NB03 asserts the file has no verdict, then **writes the verdict** with `authored_by: "NB03"`. NB04 asserts `authored_by == "NB03"` before reading, then inherits the verdict verbatim and translates it into regulatory frameworks.

**What each notebook teaches:**

- **NB01** — how a central assumptions register works as a single source of truth; how missing indicators are created before imputation; how the no-synthetic-fallback rule prevents a plausible-looking report on fake data; how a temporal audit classifies features by availability at decision time; how the leakage list is **pre-registered** (a decision made before the data was seen), and how the scan is a diagnostic, not a verdict.
- **NB02** — how a first-line development environment is shaped by the governance requirements that will be applied to it. It computes the model's metrics (AUC, precision, recall, Brier, KS, HL) without writing a verdict; it asserts the leakage count and feature set against NB01's contract; it discloses the walk-forward deviation in `split_disclosure` rather than hiding it; it applies class weights to prevent the trivial-majority-class collapse; it flags the HL degeneracy as "SUSPICIOUS" rather than trusting the p-value.
- **NB03** — how independent re-computation works (AUC recomputed from predictions, not copied); how the verdict-ownership chain is enforced (three checks); how scoring works by frozen rules, not by feeling; how the fairness grouping aggregates 50 states to 4 regions to satisfy FD-05; how the rejection reason is attributed to fairness (3 points) not AUC (1 point).
- **NB04** — how a reporting layer translates a numerical verdict into regulatory frameworks; how strict inheritance works (three assertions, no recomputation); how the verdict and the governance-local risk are separated (two ratings, two questions); how the compliance scorecard blends formula and judgment; how the dynamic mapping stays current; how findings-vs-gaps are two views of the same underlying facts.
- **NB05** — how a cross-validation qualifies its own claim (SMALL_DIRECTIONAL, not "generalizes"); how a coverage ledger prevents "we tested all proxies" when only one was tested; how explicit non-comparison (Lending Club post-mitigation = None) prevents a fabricated magnitude comparison.

---

## German Credit cross-validation (Notebook 05)

**Why a second dataset?** The Lending Club finding (region proxy for protected class → severe disparate impact) is dataset-specific. A validator should ask: does the mitigation technique generalize? German Credit is a structurally different dataset (1,000 records, 1970s–80s, direct `Sex` attribute — no geography). If removing proxy-like features improves fairness there too, the technique has some cross-population validity.

**What was tested.** Three candidate proxies: `Housing`, `Job`, `Purpose`. All three were tested (no silent coverage gaps; the coverage ledger's assertion `len(tested) + len(untested) == len(CANDIDATE_PROXIES)` passed).

| Dataset | Baseline DI | Status | Mitigation | Post-mitigation DI | Status | Δ DI |
|---|---|---|---|---|---|---|
| Lending Club | 0.5414 | SEVERE | Not applied in NB03 | not_assessed | — | — |
| German Credit | 0.9375 | OK | Removed Housing | 0.9740 | OK | +0.0365 |

**Classification:** `SMALL_DIRECTIONAL`. The technique improves DI in the *same direction* on both datasets, but the effect on German Credit is small. **The study claims *direction*, not *magnitude*.** The narrative says: *"Do NOT claim the technique generalizes in magnitude — the effect size is not comparable."*

### What the cross-validation's most important limitation is

**German Credit's baseline DI is 0.9375 — already OK by the FD-01 standard.** The Lending Club baseline is SEVERE (0.5414). The two datasets are in different fairness regimes *before mitigation even begins*.

**What this means for the claim.** The test asks: *does removing a proxy feature improve DI on a dataset whose DI is already acceptable?* The answer is yes, and small. That is **not** the same question as: *does removing a proxy feature fix a severe disparity?* The Lending Club finding was severe; the German Credit baseline is not. The cross-validation shows that proxy removal *improves* an OK outcome — not that it *remediates* a SEVERE one.

**The direction claim is supported. The magnitude claim is not.** A stronger cross-validation would require a second dataset whose baseline DI is in the SEVERE regime. German Credit cannot supply that dataset.

### What the Lending Club's "not_assessed" means

The Lending Club's post-mitigation DI is `None`, and the notebook says so explicitly in both the JSON and the narrative. This is a governance act, not a data-handling choice.

**What the `None` prevents.** If the notebook filled Lending Club's post-mitigation DI with any number — a hardcoded value, a reasonable guess, a re-derivation from a metric that isn't quite the same thing — the comparison table would show two complete rows. A reader would conclude: *the Lending Club mitigation was applied, the DI moved from 0.5414 to some value, and the German Credit result can be compared to it.* Every one of those conclusions would be false.

**The `None` is the honesty.** It records that the comparison is not one-to-one. German Credit's +0.0365 cannot be compared to "Lending Club's mitigation effect" because Lending Club has no mitigation effect measured. The comparison is **directional**, not **magnitudinal**.

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

### `governance/` — 12 project-governance artifacts

The `governance/` folder contains the project-governance artifacts that support the verdict. A reader who wants the full evidence trail follows the links below.

| Artifact | File | Description |
|---|---|---|
| Problem Definition Record | `governance/problem_definition.md` | Decision supported, population, target, constraints |
| Analytical Specification | `governance/analytical-specification.md` | Research questions, hypotheses, variables, success criteria |
| Data Specification | `governance/data-specifications.md` | Data sources, lineage, missingness, leakage audit |
| Feature Decision Log | `governance/features-decision-log.md` | Feature selection criteria, transformations, proxies |
| Methodology Document | `governance/methodology-document.md` | Candidates, selection, frozen parameters |
| Risk & Validation Plan | `governance/risk-validation-plan.md` | Thresholds, stability, drift, fairness, decision rules |
| Decision Criteria Register | `governance/decision-criteria-register.md` | Acceptable performance, drift, disparity, investigation triggers |
| Consolidated Findings Register | `governance/consolidated-findings-register.md` | The 11-item findings register, cross-referenced to the notebook phases |
| Remediation Plan | `governance/remediation-plan.md` | Sequenced actions with owners, deadlines, and success criteria |
| Approval Record | `governance/approval-record.md` | Sign-off chain, monitoring conditions, next review |
| Analysis Code & Results | `governance/analysis-code-results.md` | Index to the notebooks and a summary of their results |
| Validation Opinion | `reports/validation-opinion.md` | **Draft, pending analyst signature** — the signed verdict document |

**On the Validation Opinion's location.** The Validation Opinion is in `reports/` (not `governance/`), because it is the document that carries the verdict — and the verdict is what the `reports/` folder is for. Its **status is draft pending analyst signature**; the analyst sign-off block in the document has the `[ANALYST: ...]` placeholders that the claim-authorization discipline requires. When the analyst signs it, the status changes to "signed" and the "pending analyst signature" note is removed.

### `docs/` — narrative documents

| Document | Description |
|---|---|
| `docs/intended_use_statement.md` | Model purpose, intended use, documented limitations |
| `docs/data_dictionary.md` | Feature definitions, sources, exclusion rationale |
| `docs/regulatory_mapping.md` | Full OSFI E-23 / SR 11-7 / ECOA / IFRS 9 mapping |

### `outputs/reports/` — pipeline outputs

| Document | Description |
|---|---|
| `outputs/reports/validation_report_v2.txt` | Full independent validation report |
| `outputs/reports/validation_findings_register.csv` | 11-item findings register (machine-readable) |
| `outputs/reports/fairness_validation.csv` | Region-level fairness metrics |
| `outputs/reports/performance_summary_v2.json` | Authoritative validation verdict (`authored_by: "NB03"`) |
| `outputs/reports/german_credit_cross_validation.json` | Cross-population test results |

**On the Validation Opinion's placement.** The Validation Opinion is the only governance artifact in `reports/` rather than `governance/`. This is deliberate: the Validation Opinion is the document that **carries the verdict**, and it belongs alongside the other report outputs, not alongside the governance artifacts that support it. The LC `governance/consolidated-findings-register.md` and the LC `reports/validation-opinion.md` are the two faces of the same verdict — the register records what was found; the opinion records the analyst's signed conclusion.

---

## Related repositories

- [model-analysis-package](https://github.com/MiltonGreat/model-analysis-package) — the MAP anchor: the governance framework (AI Governance Policy, Regulatory Gap Analysis, Vendor Governance Review Template) and the package deliverables
- [fannie-mae-ifrs9-framework](https://github.com/MiltonGreat/credit-risk-model-validation) — the companion IFRS 9 stress-testing framework

## License

MIT
