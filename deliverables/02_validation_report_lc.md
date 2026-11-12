# Model Risk Validation Report — Lending Club

**Document ID:** MAP-VR-LC-001  
**Version / Status:** 0.1 / Draft — pending analyst completion of the marked sections  
**Model under review:** Lending Club 4-model ensemble (best: Random Forest, AUC 0.6307)  
**Validation type:** Independent validation (second line)  
**Regulatory basis:** SR 11-7 / OSFI E-23 (reference only; no legal conclusion drawn)  
**Owner:** `[ANALYST: name, role]`  
**Companion documents:** [`../reports/validation-opinion.md`](../reports/validation-opinion.md) · [`../governance/consolidated-findings-register.md`](../governance/consolidated-findings-register.md) · [`../governance/decision-criteria-register.md`](../governance/decision-criteria-register.md) · [`../governance/risk-validation-plan.md`](../governance/risk-validation-plan.md)

---

## At a glance

| Field | Value |
|---|---|
| **Model** | Lending Club collections-prioritization model (4-model ensemble) |
| **Best model** | Random Forest (AUC 0.6307) |
| **Developer-claimed AUC** | 0.865 (leakage-included) |
| **Realistic AUC (leakage-free)** | 0.631 |
| **Leakage-inflation** | 27.1% (0.234 AUC points) |
| **Findings** | 11 total: 2 CRITICAL, 7 HIGH, 2 MEDIUM; 1 RESOLVED, 10 OPEN |
| **Verdict** | See [`../reports/validation-opinion.md`](../reports/validation-opinion.md) — **this report does not state the verdict** |
| **What this report is** | The evidence base for the Validation Opinion |
| **What this report is not** | The Validation Opinion itself; the verdict is the analyst's signed determination |

> **Report purpose.** This report documents the evidence the independent validation produced: the scope, the data, the methodology, the outcome analysis, the findings, the regulatory mapping, and the report's own limitations. It is the document the Validation Opinion is written *on top of*.
>
> **On the separation from the Validation Opinion.** The Validation Report states the evidence. The Validation Opinion states the verdict. The two are different documents with different jobs, and the separation is deliberate — the report is the evidence base; the opinion is the analyst's signed determination. **A reader who wants the verdict reads the Validation Opinion; a reader who wants the evidence reads this report.** The distinction is the same as the FM pipeline's claim-authorization discipline: the tool produces evidence, the analyst produces the verdict.
>
> **On the report's voice.** The report states what the validation found. It does not state what the findings *mean* — that is the Validation Opinion's job. Where the report has to characterize a finding (e.g., "CRITICAL" severity), the characterization is the DCR's (the Decision Criteria Register's), not the analyst's.

---

## 1. Scope and evidence base

### 1.1 — In scope

| Item | What the validation tested | Evidence artifact |
|---|---|---|
| Data quality | Missingness, leakage, duplicates, quality score | `outputs/reports/data_quality_summary.json`; `data/feature_inventory.csv` |
| Model construction | Four model classes, hyperparameters, split method | `models/*.pkl`; `performance_summary_v2.json` |
| Performance | AUC, KS, precision, recall, PR-AUC on held-out set | `outputs/reports/performance_metrics.csv`; `outputs/reports/ks_metrics.csv` |
| Fairness | FD-05 4/5ths rule at region level | `outputs/reports/fairness_validation.csv` |
| Calibration | HL test + slope/intercept + degeneracy check | `models/calibration_metrics.csv` |
| Stability | PSI attempt (result: `not_assessed` — see §3.5) | `performance_summary_v2.json` (`psi_status`) |
| Explainability | SHAP top-10 + monotonicity check | `outputs/reports/monotonicity_check.csv` |
| Cross-population | German Credit direction-only comparison | `outputs/reports/german_credit_cross_validation.json` |

### 1.2 — Out of scope

| Item | Reason | Documented as |
|---|---|---|
| Production scoring | No deployed service exists | Out of scope |
| IFRS 9 provisioning | Target is 16+ days, not 30+ days | F-03.10 (scope restriction) |
| Canadian applicability | U.S. data only | F-03.09 (limitation) |
| Direct race/gender fairness | No protected attributes in dataset | L3 in `governance/data-specifications.md` |
| Macroeconomic stress | No macro factors in dataset | F-03.08 (limitation) |
| Time-based stability | No usable date column after leakage removal | F-03.02 (split exception) |

**The exclusions are as important as the inclusions.** A reader who reads only §1.1 would think the model is comprehensively validated. The exclusions in §1.2 make the scope honest.

### 1.3 — Evidence review

| Artifact | Source | Purpose |
|---|---|---|
| `data_quality_summary.json` | NB01 v2.1 | Data quality score 58.9/100 (POOR) |
| `feature_inventory.csv` | NB01 v2.1 | Feature inclusion decisions; 9 leakage features identified |
| `leakage_findings.csv` | NB01 v2.1 | Leakage detection evidence |
| `findings_register_v2.csv` | NB02 v2.1 | 7 model development findings |
| `performance_summary_v2.json` | NB03 v2.1 | Validation verdict — authoritative (`authored_by: "NB03"`) |
| `validation_findings_register.csv` | NB03 v2.1 | 11 validation findings |
| `fairness_validation.csv` | NB03 v2.1 | Region-level DI = 0.5414 |
| `compliance_scorecard_v2.json` | NB04 v2.1 | Governance metrics |
| `german_credit_cross_validation.json` | NB05 v2.2 | Cross-population directional evidence |

**Verdict-ownership chain verified:** the `performance_summary_v2.json` artifact carries `authored_by: "NB03"`. NB04 and NB05 both assert this field before reading. **The chain is intact.** The chain is the mechanism by which the verdict has an auditable author; it is documented in the LC RVP's §5.2.

---

## 2. Data

### 2.1 — Source and sample

| Field | Value |
|---|---|
| **Source dataset** | Lending Club public loan data |
| **Source file** | `loans_full_schema.csv` (hash `52872e7d4f8a388b`) |
| **Records** | 10,000 |
| **Columns** | 56 (raw); after feature engineering and target definition, the model trains on the feature set in `data/feature_inventory.csv` |
| **Random seed** | 42 (project-wide) |
| **Default rate** | 1.78% |

**On the sample.** The 10,000-row sample is a random sample of the Lending Club public data. The sample's default rate (1.78%) is the rate the model is trained and evaluated on. **The sample is not stratified**; the LC pipeline's sampling method is documented in `governance/data-specifications.md` §3.

### 2.2 — Data quality

**Data quality score: 58.9/100 (POOR).** The score is a composite of four penalties: missingness (21.1), leakage (15), macro gap (5.0), and duplicates (`[ANALYST: insert from NB01 output — the duplicate penalty is not shown in the outputs I have]`). The bands (GOOD ≥ 80, MODERATE ≥ 60, POOR ≥ 40) are descriptive conventions, not DCR thresholds.

**The score is POOR by the pipeline's own measure.** The score's meaning is in the LC Data Specification (`governance/data-specifications.md` §7), which records what the score does and does not measure.

### 2.3 — Leakage

**Nine features were identified and removed:** `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, `issue_month`.

**The measured cost of the leakage is 0.234 AUC points (0.865 → 0.631), or 27.1% of the developer-claimed figure.** The leakage is documented as F-03.01 (CRITICAL, RESOLVED) in the findings register.

**The leakage list is pre-registered.** It was frozen in the LC Data Specification §7 before the data was inspected. **The pre-registered list is what makes the finding a finding rather than a post-hoc rationalization** — the list is not tuned to produce a particular answer. The LC Feature Decision Log (`governance/features-decision-log.md` §6) documents the pre-registration and the reason it matters.

### 2.4 — Missingness

**Ten columns have missing values; five columns exceed 20% missingness.** The high-missingness columns are documented in the LC Data Specification (`governance/data-specifications.md` §4.2). The imputation strategy is median (numeric) / mode (categorical), with missing indicators created *before* imputation.

**Zero missing values in the clean frame does not mean the missingness was harmless.** The missing indicators are the mitigation, not the remedy. The LC Data Specification §4.4 documents this distinction.

---

## 3. Outcome analysis

### 3.1 — Discrimination

| Model | AUC-ROC | PR-AUC | Precision | Recall | F1 |
|---|---|---|---|---|---|
| Logistic Regression | 0.536 | 0.036 | 0.039 | 0.333 | 0.070 |
| XGBoost | 0.604 | 0.034 | 0.029 | 0.444 | 0.055 |
| Random Forest | **0.631** | 0.059 | 0.085 | 0.333 | 0.135 |
| LightGBM | 0.621 | 0.029 | 0.029 | 0.694 | 0.056 |

**Best by held-out AUC:** Random Forest (0.6307). **KS statistic:** 0.2671 (Weak). **PR-AUC:** 0.059.

**The discrimination result is a MODERATE-band finding per DCR §1.** AUC 0.631 is between the "weak" floor (0.55) and the "good" floor (0.65). The DCR's bands are the standard; the discrimination findings are F-03.04 (poor calibration — separate from discrimination) and F-03.06 (weak discrimination, KS = 0.2671).

**AUC 0.6307 is the realistic performance after leakage removal.** It is not comparable to the developer-claimed 0.865, which was produced with the nine leakage features present. **The 0.234-point gap is the leakage premium** — the performance the model would have appeared to have if the leakage features were available at decision time.

**The report does not compare 0.631 to a reasonable alternative benchmark** (a logistic regression baseline, published Lending Club benchmarks, an industry median). That comparison would tell a reader whether 0.631 is *good enough* for the model's intended use, not just whether it is *worse than the inflated claim*. **It is a real analytical gap, and it belongs to the Validation Opinion, not to this report.** The Validation Opinion's §A.7 documents the gap as a limitation of the evidence.

### 3.2 — Calibration

| Model | HL p | Slope | Intercept | Status |
|---|---|---|---|---|
| Logistic Regression | 1.000 | 0.629 | −4.280 | SUSPICIOUS |
| XGBoost | 0.000 | 1.225 | −4.241 | POOR + SUSPICIOUS |
| Random Forest | 1.000 | 1.418 | −4.487 | SUSPICIOUS |
| LightGBM | 0.729 | 1.078 | −4.244 | OK |

**The HL degeneracy is a known artifact.** P-values of exactly 0.0 or 1.0 are numerical artifacts of the HL test's chi-square computation when a decile has zero observed events or zero predicted probability. **Three of four models show this.** Only LightGBM has a valid HL result.

**XGBoost's HL p = 0.000 is both a failed calibration result and a degenerate one.** F-03.04 (poor calibration) and F-03.05 (suspicious HL) both fire. **The poor calibration is not yet confirmed** until the HL implementation is verified; the degeneracy flag is the reason.

**The report states the evidence; it does not state the calibration's consequence.** The consequence — whether a miscalibrated model is usable for collections prioritization — is the Validation Opinion's determination. The Opinion's §A.8 states it.

### 3.3 — Fairness

**The test applied:** FD-05 4/5ths rule at region level. **State aggregated to 4 Census regions** (Northeast, Midwest, South, West) after FD-03 exclusion of "Other" (N=6 < 30). **Cardinality 4**, within FD-05's ≤ 50 constraint.

**Result:** DI = min(approval) / max(approval) = **0.5414**.

**Per-region breakdown:**

| Region | N | Approval rate | Default rate | TPR |
|---|---|---|---|---|
| South | 736 | 6.5% | 1.9% | 35.7% |
| West | 526 | 9.9% | 1.1% | 33.3% |
| Northeast | 377 | 9.6% | 3.2% | 33.3% |
| Midwest | 355 | 5.3% | 1.1% | 25.0% |

**The disparity is a two-tier approval pattern** — West/Northeast (~9.6–9.9%) vs. South/Midwest (~5.3–6.5%). The DI is below the 0.60 SEVERE threshold.

**On the per-state DI.** The v2.0 notebook computed DI on per-state one-hot columns, which produced degenerate comparisons at cardinality 50. **That was not the 4/5ths test.** The v2.1 notebook aggregates the 50 states to 4 regions, which is the FD-05-compliant test. **The 0.5414 result is the FD-05-compliant finding; the per-state values would be statistical artifacts of the rule's misapplication.** The FD-05 rationale is documented in the LC Decision Criteria Register (`governance/decision-criteria-register.md`) §3.3.

**The protection is a proxy.** Region is a geographic proxy for protected-class composition; the dataset has no direct race or ethnicity field. **The finding is indicative, not definitive.** The report states the finding; the analyst's reading of the finding's regulatory significance is the Validation Opinion's determination (see the Opinion's §A.6).

### 3.4 — Explainability

**SHAP computed on 100 validation rows** (indicative, not exhaustive). **Top-10 features:** `interest_rate`, `total_debit_limit`, `debt_to_income`, `loan_to_income`, `inquiries_last_12m`, `total_collection_amount_ever`, `installment`, `annual_income`, `accounts_opened_24m`, `total_credit_limit`.

**Leakage check:** no leakage features in top-20 SHAP — clean.

**Counterintuitive features (4):**

- `loan_to_income` — expected negative, actual positive
- `total_collection_amount_ever` — expected positive, actual negative
- `annual_income` — expected negative, actual positive
- `total_credit_limit` — expected positive, actual negative

**The SHAP direction for these four features contradicts credit-risk theory.** Either the feature encodes something unexpected, or the model has learned a spurious interaction. **This is F-03.07 (HIGH).** The finding does not directly affect the score, but it must be addressed in the Validation Opinion's discussion (see the Opinion's §A.10).

### 3.5 — Stability

**PSI:** `not_assessed`. The split is random-stratified, not time-based. **PSI on index-split halves of a random split measures within-distribution noise, not population stability.** A proper PSI requires time cohorts.

**The stability score component is 0.** This is not evidence of stability — it is the DCR's treatment of "not assessed" as 0 points. **The limitation is that PSI could not be computed on this data.** It is a data limitation, not a model property. The finding is F-03.02 (walk-forward split not implemented).

---

## 4. Findings

**11 validation findings: 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 RESOLVED (leakage removed), 10 OPEN.**

| ID | Severity | Finding | Status |
|---|---|---|---|
| F-03.01 | CRITICAL | 9 leakage features removed | RESOLVED |
| F-03.02 | HIGH | Walk-forward split not implemented | OPEN |
| F-03.03 | CRITICAL | Severe disparate impact (DI = 0.5414) | OPEN |
| F-03.04 | HIGH | Poor calibration in XGBoost | OPEN |
| F-03.05 | MEDIUM | Suspicious HL in 3 models | OPEN |
| F-03.06 | HIGH | Weak discrimination (KS = 0.2671) | OPEN |
| F-03.07 | HIGH | Counterintuitive SHAP (4 features) | OPEN |
| F-03.08 | HIGH | No macro indicators | OPEN |
| F-03.09 | HIGH | U.S. data only | OPEN |
| F-03.10 | HIGH | IFRS 9 staging misalignment | OPEN |
| F-03.11 | MEDIUM | Low precision (0.029) | OPEN |

**The findings register is the machine-readable version** — `outputs/reports/validation_findings_register.csv`. The register above is the human-readable summary. **The LC Consolidated Findings Register** (`governance/consolidated-findings-register.md`) is the cross-referenced version; it is the document a reader consults if they want the finding-to-notebook-phase mapping.

**On the findings' severity.** The severities are the DCR's (the Decision Criteria Register's), not the analyst's. The severity is determined by the finding's category (e.g., a SEVERE fairness finding is CRITICAL; a WARNING-level calibration finding is MEDIUM). **The severity column is a factual classification, not a judgment.**

---

## 5. Regulatory mapping

| Framework | Compliant | Total | % |
|---|---|---|---|
| OSFI E-23 | 5 | 7 | 71% |
| SR 11-7 | 4 | 6 | 67% |
| ECOA/CFPB | 0 | 2 | 0% |
| IFRS 9 | 0 | 2 | 0% |

**Key non-compliance:**

- **ECOA Fair Lending** — NON-COMPLIANT (severe disparate impact, DI 0.5414)
- **ECOA Protected Classes** — PARTIALLY COMPLIANT (region proxy only, no direct race/gender)
- **IFRS 9 Staging** — NON-COMPLIANT (16+ days vs. 30+ days) — **scope restriction, not defect**
- **IFRS 9 Provisioning** — NON-COMPLIANT (not designed for provisioning) — **scope restriction, not defect**
- **OSFI E-23 3.2 Monitoring** — PARTIALLY COMPLIANT (PSI not assessed)

**On the "0 of 2" percentages.** A reader seeing "0% compliant" for ECOA and IFRS 9 might conclude the model is a complete failure on both frameworks. **The correct reading is more nuanced.** The ECOA 0/2 decomposes to one NON-COMPLIANT (fair lending) and one PARTIALLY COMPLIANT (protected classes); the two are different kinds of non-compliance with different remediations. The IFRS 9 0/2 decomposes to two NON-COMPLIANT scope restrictions — the model correctly does not claim to be an IFRS 9 model. **The percentage is not the story; the decomposition is.** The LC README's "Reading 0 of 2 (0%) compliant" section documents the decomposition.

**The mapping is the LC Governance team's** (NB04). The regulatory mapping's full evidence — the specific requirements, the evidence artifacts, the gaps — is in `outputs/reports/regulatory_mapping_v2.csv` and `outputs/reports/compliance_report_v2.txt`.

---

## 6. Report limitations

**The validation has six process limitations** — things a reader should know about the validation itself, not about the model or the data.

| ID | Limitation | Impact |
|---|---|---|
| VL-1 | **Fairness test uses a region proxy, not a direct protected attribute** | Fairness findings are indicative, not definitive |
| VL-2 | **PSI is `not_assessed` on a random split** | Stability not tested; 0 stability points reflect absence of evidence, not evidence of stability |
| VL-3 | **SHAP is on a 100-row sample, not the full validation set** | Feature importance is indicative, not exhaustive |
| VL-4 | **HL test produces degenerate p-values for 3 of 4 models** | Calibration uncertain for those models; only LightGBM has a valid result |
| VL-5 | **Single-threshold fairness** | Approval rate measured at 0.5 only; the DI may differ at a different operating point |
| VL-6 | **The validation is performed by the same analyst who built the pipeline** | The independence claim rests on the NB02/NB03 separation and the `authored_by` assertion, not on organizational separation |

**On VL-6.** The Lending Club project is a portfolio artifact, and the same analyst built the pipeline and wrote the validation. **The independence claim rests on the technical separation** (NB02 vs NB03; `authored_by` assertion) and the **decision authority matrix** (§6.2 of the LC RVP), not on organizational independence. In a production deployment, the validator would be a separate person in a separate function. **The report states this clearly** — a reader who assumes organizational independence has misunderstood the artifact.

**The six limitations are the validation's process limitations.** The model's data limitations (L1–L8) are in `governance/data-specifications.md` §8. The methodology's limitations (M1–M6) are in `governance/methodology-document.md` §8. **The three registers (data, methodology, validation) are separate** — they describe different kinds of limitations.

---

## 7. Report completeness and hand-off

### 7.1 — What this report contains

- The evidence base: scope, data, methodology, outcome analysis, findings, regulatory mapping.
- The 11 findings with their severities, statuses, and evidence artifacts.
- The regulatory mapping with the framework-level compliance percentages.
- The validation's six process limitations.

### 7.2 — What this report does not contain

- **The verdict.** The verdict (6/10 → HIGH → REJECT) is in `reports/validation-opinion.md`, not in this report.
- **The analyst's reading of the findings.** The report states the evidence; the analyst's reading is the Validation Opinion.
- **The remediation plan.** The report states the findings; the remediation is in `governance/remediation-plan.md`.
- **The comparison to a reasonable alternative benchmark.** The report states the realistic AUC; the comparison is a gap the Validation Opinion documents (§A.7).

### 7.3 — Hand-off

**The Validation Opinion (`reports/validation-opinion.md`) is the analyst's signed determination on this evidence.** A reader who has this report and the Opinion has the full picture: the evidence and the verdict.

**The LC Decision Criteria Register (`governance/decision-criteria-register.md`)** is the criteria the report's findings are measured against. A reader who wants to know "was this finding severe enough to be CRITICAL?" reads the DCR's severity bands.

**The LC Consolidated Findings Register (`governance/consolidated-findings-register.md`)** is the cross-referenced version of the 11 findings. A reader who wants the finding-to-notebook-phase mapping reads the CFR.

**The LC Remediation Plan (`governance/remediation-plan.md`)** is the plan for closing the findings. A reader who wants to know "what happens next?" reads the remediation plan.

---

## 8. Document control

| Field | Value |
|---|---|
| **Document ID** | MAP-VR-LC-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | [`../reports/validation-opinion.md`](../reports/validation-opinion.md) · [`../governance/consolidated-findings-register.md`](../governance/consolidated-findings-register.md) · [`../governance/decision-criteria-register.md`](../governance/decision-criteria-register.md) · [`../governance/risk-validation-plan.md`](../governance/risk-validation-plan.md) · [`../governance/remediation-plan.md`](../governance/remediation-plan.md) |
| **Next review** | At the next re-validation |

---

## 9. What this report contains, and what it does not

### What it contains (sourced from the notebook outputs and the LC `governance/` folder)

- The evidence base: scope, data, methodology, outcome analysis, findings, regulatory mapping.
- The 11 findings with their severities, statuses, and evidence artifacts.
- The regulatory mapping with the framework-level compliance percentages.
- The validation's six process limitations.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§2.2** — the duplicate penalty in the data quality score (`[ANALYST: insert from NB01 output]`).
- **§8** — the document owner's name and role.
- **The report's narrative voice.** The report states the evidence; the analyst's framing of *why* the evidence matters is in the Validation Opinion, not in this report. **The report's narrative voice is intentionally neutral** — it states what the validation found, and leaves the interpretation to the Opinion.

**When the markers are filled, the report is complete.** It is the evidence base the Validation Opinion sits on top of; the two documents together are the full LC validation package.
