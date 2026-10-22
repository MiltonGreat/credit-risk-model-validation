# Validation Opinion

**Model:** Lending Club Credit Default Model (collections prioritization)  
**Validation run:** 20261021_145017 (NB03 v2.1)  
**Author:** `[ANALYST: name]`  
**Role:** `[ANALYST: role — Independent Validator / Model Risk]`  
**Date:** `[ANALYST: date]`  
**Signature:** `[ANALYST: signature]`

---

## A.1 Executive Summary

The Lending Club Credit Default Model has been independently validated. The validation verdict is **HIGH / REJECT** with a risk score of **6/10**.

The rejection is driven by **fairness**, not by model performance. Region-level disparate impact (DI = 0.5414) falls below the SEVERE threshold of 0.60 under DCR §3 FD-01. Model performance contributes 1 of 6 points (AUC 0.631, MODERATE band); fairness contributes 3; calibration contributes 2.

The model is **not approved for deployment**. The path to approval requires: (1) fairness remediation to bring region-level DI to ≥ 0.80, (2) calibration remediation for XGBoost (HL p = 0.000, degenerate), and (3) either implementation of time-based split or a formal amendment to MD-001 §2.

**Material finding:** The developer's claimed AUC of 0.865 was produced with 9 leakage features present. After leakage removal, realistic held-out AUC is **0.631** — a 27.1% relative degradation. The 0.234 AUC-point gap is the leakage premium.

---

## A.2 Scope of Validation

**In scope:**

- Lending Club public dataset (10,000 records, `loans_full_schema.csv`)
- Target definition: 16+ days past due (collections prioritization scope, not IFRS 9 provisioning)
- Four model classes: Logistic Regression, XGBoost, Random Forest, LightGBM
- Fairness testing at region level (FD-05 compliant, 4 groups after FD-03 N≥30 exclusion)
- Calibration testing (Hosmer-Lemeshow, slope/intercept)
- Stability testing (PSI — attempted, not assessed due to random split)
- SHAP explainability + monotonicity check

**Out of scope (documented limitations):**

- IFRS 9 provisioning use (target misaligned: 16+ days vs 30+ days)
- Canadian portfolio applicability (U.S. data only)
- Direct protected-attribute fairness testing (dataset has no race/gender; region used as proxy)
- Time-based stability testing (no usable date column after leakage removal)
- Macroeconomic stress testing (no macro indicators in dataset)

---

## A.3 Evidence Reviewed

| Artifact | Source | Purpose |
|---|---|---|
| `data_quality_summary.json` | NB01 v2.1 | Data quality score 58.9/100 (POOR) |
| `feature_inventory.csv` | NB01 v2.1 | Feature inclusion decisions; 9 leakage features identified |
| `leakage_findings.csv` | NB01 v2.1 | Leakage detection evidence |
| `findings_register_v2.csv` | NB02 v2.1 | 7 model development findings |
| `performance_summary_v2.json` | NB03 v2.1 | **Validation verdict — authoritative** |
| `validation_findings_register.csv` | NB03 v2.1 | 11 validation findings |
| `fairness_validation.csv` | NB03 v2.1 | Region-level DI = 0.5414 |
| `compliance_scorecard_v2.json` | NB04 v2.1 | Governance metrics |
| `german_credit_cross_validation.json` | NB05 v2.2 | Cross-population directional evidence |

**Verdict-ownership chain verified:** NB03 summary carries `authored_by: "NB03"`. NB04 and NB05 both assert this field before reading. The chain is intact.

---

## A.4 Findings Summary

**11 validation findings:** 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 RESOLVED (leakage removed), 10 OPEN.

| ID | Severity | Finding | Status |
|---|---|---|---|
| F-03.01 | CRITICAL | 9 leakage features removed | RESOLVED |
| F-03.02 | HIGH | Walk-forward split not implemented | OPEN |
| F-03.03 | CRITICAL | Severe disparate impact (DI=0.5414) | OPEN |
| F-03.04 | HIGH | Poor calibration in XGBoost | OPEN |
| F-03.05 | MEDIUM | Suspicious HL in 3 models | OPEN |
| F-03.06 | HIGH | Weak discrimination (KS=0.2671) | OPEN |
| F-03.07 | HIGH | Counterintuitive SHAP (4 features) | OPEN |
| F-03.08 | HIGH | No macro indicators | OPEN |
| F-03.09 | HIGH | U.S. data only | OPEN |
| F-03.10 | HIGH | IFRS 9 staging misalignment | OPEN |
| F-03.11 | MEDIUM | Low precision (0.029) | OPEN |

---

## A.5 Risk Score Breakdown

| Component | Score | Max | Justification |
|---|---|---|---|
| Performance | 1 | 3 | AUC 0.631 in MODERATE band (0.60–0.65) |
| Stability | 0 | 2 | PSI `not_assessed` (random split) — DCR treats as 0 |
| Calibration | 2 | 2 | XGBoost HL p = 0.000 (poor) |
| Fairness | 3 | 3 | Region-level DI 0.5414 SEVERE |
| Leakage | 0 | 3 | Resolved (removed in NB02) |
| **Total** | **6** | **10** | **HIGH → REJECT** |

---

## A.6 Fairness Assessment

**Test applied:** FD-05 compliant 4/5ths rule at region level. State aggregated to 4 Census regions (Northeast, Midwest, South, West) after FD-03 exclusion of "Other" (N=6 < 30). Cardinality 4, within FD-05's ≤50 constraint.

**Result:** DI = min(approval) / max(approval) = 0.053 / 0.099 = **0.5414**.

**Per-region breakdown:**

| Region | N | Approval rate | Default rate | TPR |
|---|---|---|---|---|
| South | 736 | 6.5% | 1.9% | 35.7% |
| West | 526 | 9.9% | 1.1% | 33.3% |
| Northeast | 377 | 9.6% | 3.2% | 33.3% |
| Midwest | 355 | 5.3% | 1.1% | 25.0% |

**Interpretation:** The disparity is driven by a two-tier approval pattern — West/Northeast (~9.6–9.9%) vs South/Midwest (~5.3–6.5%). The DI is below the 0.60 SEVERE threshold.

**Limitations:** Region is a proxy, not a direct protected attribute. Per-state one-hot DI (v2.0's approach) is not the 4/5ths test — it compares one state against the complement of 49, which produces degenerate comparisons at high cardinality. The FD-05-compliant region-level test is the binding one.

---

## A.7 Performance Assessment

| Model | AUC-ROC | PR-AUC | Brier | Precision | Recall | F1 |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.536 | 0.036 | 0.220 | 0.039 | 0.333 | 0.070 |
| XGBoost | 0.604 | 0.034 | 0.070 | 0.029 | 0.444 | 0.055 |
| **Random Forest** | **0.631** | 0.059 | 0.127 | 0.085 | 0.333 | 0.135 |
| LightGBM | 0.621 | 0.029 | 0.084 | 0.029 | 0.694 | 0.056 |

**Best model:** Random Forest (AUC 0.631).

**Leakage impact:** Developer-claimed AUC 0.865 → realistic AUC 0.631. Delta = 0.234 (27.1%).

**KS statistic:** 0.2671 (Weak) — see F-03.06.

**Precision:** 0.085 at the operating threshold — high false-positive rate. See F-03.11.

**Cross-validation:** Lending Club AUC 0.631 is lower than German Credit's 0.7415, but the datasets differ in scale (10,000 vs 1,000), era (2018 vs 1970s–80s), and target definition (default vs approval). The numbers are not directly comparable; the cross-validation tests the fairness technique's direction, not model performance.

---

## A.8 Calibration Assessment

| Model | HL p | Slope | Intercept | Status |
|---|---|---|---|---|
| Logistic Regression | 1.000 | 0.629 | −4.280 | SUSPICIOUS |
| XGBoost | 0.000 | 1.225 | −4.241 | POOR + SUSPICIOUS |
| Random Forest | 1.000 | 1.418 | −4.487 | SUSPICIOUS |
| LightGBM | 0.729 | 1.078 | −4.244 | OK |

**Degeneracy:** HL p-values of exactly 0.0 or 1.0 are numerical artifacts, not calibration evidence. Three of four models show this. Only LightGBM has a valid HL result.

**XGBoost:** HL p = 0.000 is both a failed calibration result and a degenerate one. F-03.04 (poor calibration) and F-03.05 (suspicious HL) both fire. The poor calibration is not yet confirmed until the HL implementation is verified.

---

## A.9 Stability Assessment

**PSI:** Not assessed. The split is random-stratified, not time-based. PSI on index-split halves of a random split measures within-distribution noise, not population stability. A proper PSI requires time cohorts.

**Stability score component:** 0. This is **not** evidence of stability — it's the DCR's treatment of "not assessed" as 0 points. The limitation is that PSI could not be computed on this data. It is a data limitation, not a model property.

---

## A.10 SHAP and Monotonicity

**SHAP computed on 100 validation rows (indicative, not exhaustive).**

**Top-10 features:** `interest_rate`, `total_debit_limit`, `debt_to_income`, `loan_to_income`, `inquiries_last_12m`, `total_collection_amount_ever`, `installment`, `annual_income`, `accounts_opened_24m`, `total_credit_limit`.

**Leakage check:** No leakage features in top-20 SHAP — clean.

**Counterintuitive features (4):**

- `loan_to_income` — expected negative, actual positive
- `total_collection_amount_ever` — expected positive, actual negative
- `annual_income` — expected negative, actual positive
- `total_credit_limit` — expected positive, actual negative

**Interpretation:** The SHAP direction for these four features contradicts credit-risk theory. Either the feature encodes something unexpected, or the model has learned a spurious interaction. This is F-03.07 (HIGH). It does not directly affect the score but must be addressed in the Validation Opinion's discussion.

---

## A.11 Regulatory Mapping

| Framework | Compliant | Total | % |
|---|---|---|---|
| OSFI E-23 | 5 | 7 | 71% |
| SR 11-7 | 4 | 6 | 67% |
| ECOA/CFPB | 0 | 2 | 0% |
| IFRS 9 | 0 | 2 | 0% |

**Key non-compliance:**

- **ECOA Fair Lending** — NON-COMPLIANT (severe disparate impact, DI 0.5414)
- **ECOA Protected Classes** — PARTIALLY COMPLIANT (region proxy only, no direct race/gender)
- **IFRS 9 Staging** — NON-COMPLIANT (16+ days vs 30+ days) — **scope restriction, not defect**
- **IFRS 9 Provisioning** — NON-COMPLIANT (not designed for provisioning) — **scope restriction, not defect**
- **OSFI E-23 3.2 Monitoring** — PARTIALLY COMPLIANT (PSI not assessed)

---

## A.12 Conditions for Approval

The model is **not approved for deployment**. Conditions for re-validation:

| # | Condition | Addresses | Timeline |
|---|---|---|---|
| 1 | Fairness remediation: remove state features or apply reweighting; re-test region DI ≥ 0.80 | F-03.03 | 3 months |
| 2 | Calibration remediation: Platt scaling or isotonic regression for XGBoost; verify HL implementation | F-03.04, F-03.05 | 1 month |
| 3 | Split method: implement time-based split with preserved origination date, or formally amend MD-001 §2 with disclosure | F-03.02 | Next version |
| 4 | KS improvement: investigate feature engineering to raise KS above 0.30 | F-03.06 | 2 months |
| 5 | SHAP monotonicity: investigate counterintuitive features; re-engineer if necessary | F-03.07 | 2 weeks |
| 6 | Macro factors: add GDP/unemployment/HPI for production | F-03.08 | 6 months |
| 7 | Canadian validation: validate on Canadian data before Canadian deployment | F-03.09 | 6 months |
| 8 | IFRS 9: either recalibrate for 30+ day threshold or formally restrict scope | F-03.10 | 3 months |

---

## A.13 OSFI E-23 Mapping

| E-23 Principle | Status | Evidence | Gap |
|---|---|---|---|
| **Pillar 1 — Model Development** | | | |
| Model design and data | COMPLIANT | NB01 data quality, NB02 model development | Macro factors missing |
| Model implementation | COMPLIANT | NB02 pipeline, NB03 independent validation | — |
| Model use | PARTIALLY COMPLIANT | Collections prioritization scope documented | IFRS 9 use excluded |
| **Pillar 2 — Model Risk Governance** | | | |
| Roles and responsibilities | COMPLIANT | Validation Opinion author, remediation owners assigned | — |
| Model risk appetite | COMPLIANT | Risk score 6/10, HIGH → REJECT | — |
| **Pillar 3 — Model Inventory & Validation** | | | |
| Model inventory | COMPLIANT | NB01–NB05 pipeline documented | — |
| Model risk assessment | COMPLIANT | Five-component risk score, DCR §9 | — |
| Independent validation | COMPLIANT | NB03 v2.1 independent validation | — |
| **Pillar 4 — Model Risk Reporting** | | | |
| Reporting to senior management | `[ANALYST: confirm whether MRC has received this]` | NB04 compliance report | — |
| Ongoing monitoring | PARTIALLY COMPLIANT | PSI not assessed; monthly monitoring framework not established | Monitoring dashboard |

---

## A.14 Analyst Declaration

> **I, `[ANALYST: name]`, having reviewed the Lending Club Credit Default Model and the independent validation evidence, declare:**
>
> **1.** The model is **NOT APPROVED** for deployment in its current form.
>
> **2.** The primary reason for rejection is severe disparate impact at region level (DI = 0.5414), which violates DCR §3 FD-01 (SEVERE threshold < 0.60).
>
> **3.** The model's realistic performance (AUC 0.631) is materially below the developer's claimed 0.865 — a 27.1% relative degradation attributable to 9 leakage features present in the original training data. The leakage has been removed.
>
> **4.** The eight conditions in §A.12 constitute the path to re-validation. The conditions are specific, owned, and time-bound.
>
> **5.** This Validation Opinion is based on the evidence enumerated in §A.3. The verdict-ownership chain (NB02 → NB03 → NB04 → NB05) has been verified. No material gaps in the evidence base were found.
>
> **6.** The model's limitations — particularly the region-as-proxy fairness testing, the absence of macro factors, and the U.S.-only data — constrain the interpretation of the verdict but do not change it.
>
> **Signed:** `[ANALYST: signature]`  
> **Date:** `[ANALYST: date]`

---

## Document Control

| Field | Value |
|---|---|
| Version | 1.0 |
| Status | Draft pending analyst signature |
| Supersedes | None (first version) |
| Next review | At completion of conditions 1–3 (target: 3 months) |
| Related documents | MAP deliverables 1–12, AI Interaction Record |
