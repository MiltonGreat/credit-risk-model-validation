# Analytical Specification (AS)

**Document ID:** LC-AS-001  
**Version / Status:** 2.2 / Frozen at NB02 v2.1 execution (run_id `20261021_145000`)  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Analytical Specification (AS)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · LC-CFR-001 (Consolidated Findings Register) · LC-RMP-001 (Remediation Plan) · MAP-VR-LC-001 (Validation Report)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Specify what the Lending Club analysis must produce — artifacts, schemas, acceptance criteria, and the pipeline that produces them |
| **Notebooks specified** | NB01–NB05 |
| **Contract pattern** | Every stage writes a machine-readable contract that the next stage reads; a missing contract raises `FileNotFoundError` |
| **Reproducibility requirements** | Seed 42; content hash; per-notebook manifest; per-run audit log; `run_id` and `notebook_version` on every artifact |
| **Where results live** | In the notebooks' output artifacts and in LC-DCR-001 §4 — **not in this specification** |

> **Document purpose.** This specification defines what the Lending Club analysis must produce: the artifacts, their schemas, their acceptance criteria, and the pipeline that produces them. It is the specification against which the notebooks are validated. Every artifact in the pipeline traces to a specification item here.
>
> **On the relationship between this specification and the results.** The AS states what the artifacts must contain and what criteria they must satisfy. **The measured results** — whether the criteria were met, and what values the artifacts contain — are in the notebooks' outputs and in LC-DCR-001 §4 (results against criteria). The AS does not contain the current run's values; the analysis reports them.

---

## 1. Purpose and scope

This specification defines what the Lending Club analysis must produce: the artifacts, their schemas, their acceptance criteria, and the pipeline that produces them. It is the specification against which the notebooks are validated. Every artifact in the pipeline traces to a specification item here.

**In scope:** five notebooks (NB01–NB05), their outputs, the contracts between them, and the acceptance criteria.

**Out of scope:** the MAP documents (which are separate deliverables), the production deployment (which does not exist), and the German Credit cross-validation's own specification (which is in NB05's own spec).

---

## 2. Stage specification

The pipeline has five stages, each implemented by one notebook. This section specifies, for each stage: the questions it answers, the inputs it requires, the outputs it produces, and the acceptance criteria its outputs must satisfy.

**This is the specification's core content.** A developer reading this section knows what to build; a validator reading it knows what to check. A reader who wants to know "what must NB03 produce?" finds the answer in one row of the table below, not in three sections.

### 2.1 — Stage overview

| Stage | Notebook | Questions answered | Inputs | Outputs | Key contract for next stage |
|---|---|---|---|---|---|
| 1 | NB01 | Is the data clean? What is the leakage? What features are available? | `loans_full_schema.csv` | `data_quality_summary.json`, `feature_inventory.csv`, `leakage_findings.csv`, `findings_register.csv`, `temporal_audit.csv` | `feature_inventory.csv` (NB02's input) |
| 2 | NB02 | What model best predicts default? What is the leakage-free performance? | NB01 artifacts, raw data | 4 model pickles, `predictions.pkl`, `test_data.pkl`, `performance_summary_v2.json` (no verdict), `findings_register_v2.csv`, `performance_metrics.csv`, `cv_results.csv`, `calibration_metrics.csv` | `predictions.pkl`, `test_data.pkl`, `performance_summary_v2.json` (NB03's input) |
| 3 | NB03 | Does the model pass performance, fairness, calibration? | NB02 artifacts | `performance_summary_v2.json` (with verdict), `validation_findings_register.csv`, `fairness_validation.csv`, `ks_metrics.csv`, `monotonicity_check.csv`, `leakage_comparison_validation.csv` | `performance_summary_v2.json` with verdict (NB04's and NB05's input) |
| 4 | NB04 | Is the model compliant with OSFI, SR 11-7, ECOA, IFRS 9? | NB03 artifacts | `compliance_scorecard_v2.json`, `regulatory_mapping_v2.csv`, `compliance_report_v2.txt` | Terminal (no downstream) |
| 5 | NB05 | Does the mitigation technique generalize? | NB03 artifacts, German Credit data | `german_credit_cross_validation.json`, `german_credit_cross_validation.csv` | Terminal (no downstream) |

**On the ordering.** NB01 must complete before NB02 (data quality feeds feature decisions). NB02 must complete before NB03 (the models and predictions are the validation's input). NB03 must complete before NB04 (the verdict feeds the compliance mapping). NB05 is **parallel to NB04** — it does not depend on NB04's output, and it does not depend on NB03's verdict; it reads NB03's artifacts and tests a different question (does the mitigation technique generalize to German Credit?). NB05 can run in parallel with NB04.

**On the contract pattern.** Every stage writes a machine-readable contract that the next stage reads. The contract is the specification's enforcement mechanism: a missing or malformed contract raises `FileNotFoundError` or a validation error at the next stage's start, so the pipeline fails loudly at the first broken contract rather than silently continuing with stale data.

### 2.2 — Acceptance criteria

The acceptance criteria for each stage are stated below. **The measured results against each criterion are in LC-DCR-001 §4 (results against criteria); this section states the criteria only.**

**NB01 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Leakage count | The `leakage_count` field in `data_quality_summary.json` must match the number of leakage features present in the actual dataset |
| Feature inventory completeness | Every raw column must have a row in `feature_inventory.csv` |
| Leakage findings coverage | At least one row per leakage feature in `leakage_findings.csv` |
| Findings register | One row per finding raised in the notebook's scan, with all required columns populated |
| Temporal audit | Every raw feature must be classified in `temporal_audit.csv` |

**NB02 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| All four models trained | All four pickles (`logistic_regression.pkl`, `xgboost.pkl`, `random_forest.pkl`, `lightgbm.pkl`) must exist and be loadable |
| No verdict in summary | `performance_summary_v2.json` must not contain any of the `does_not_contain` fields (`risk_score`, `risk_rating`, `recommendation`, `validation_verdict`, `fairness_status`) |
| Predictions completeness | `predictions.pkl` must contain a key for each model name, with a 1D `np.ndarray` of length `len(X_val)` |
| Test data provenance | `test_data.pkl` must contain `X_val`, `y_val`, `features`, `split_type`, `split_disclosure`, `run_id`, `notebook_version` |
| Calibration computed | `calibration_metrics.csv` must contain `HL_p_value` for each model |

**NB03 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Verdict authorship | `performance_summary_v2.json` must contain `authored_by: "NB03"` |
| Component sum | `sum(risk_score_components.values())` must equal `risk_score` |
| Fairness cardinality | The fairness test must be at FD-05-compliant cardinality (≤ 50 groups) |
| Fairness group sufficiency | Groups with N < 30 must be excluded from the DI calculation (FD-03) |
| KS computed from ROC | `ks_metrics.csv` must contain `KS_Statistic` computed from the ROC curve |
| SHAP monotonicity | Every top-10 feature must have a row in `monotonicity_check.csv` |
| Leakage comparison labels | `leakage_comparison_validation.csv` must use neutral labels (not INVALID/VALID) |

**NB04 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Verdict inheritance | `compliance_scorecard_v2.json` must contain `verdict_inherited_from: "NB03"` |
| Verdict match | The `validation_verdict` in the scorecard must match NB03's `performance_summary_v2.json` |
| Regulatory status format | `regulatory_mapping_v2.csv` must use exact-match status values (`COMPLIANT`, `PARTIALLY COMPLIANT`, `NON-COMPLIANT`) |
| No verdict claims in percentages | The compliance report must not use `[OK]` or `[!]` tokens on percentages |

**NB05 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| LC post-mitigation is not assessed | `german_credit_cross_validation.json` must contain `lending_club.post_mitigation_DI: null` (not a fabricated value) |
| LC post-mitigation CSV is NaN | `german_credit_cross_validation.csv` must contain `LC post_mitigation_DI` as NaN, not a fabricated value |
| Coverage ledger | Every candidate proxy must be either tested or recorded as untested in the coverage ledger |
| Classification rule | The direction classification (`SMALL_DIRECTIONAL`, `LARGE_DIRECTIONAL`, etc.) must be derived from the delta-DI rule, not from a hardcoded value |

**On the criteria's role.** These are the criteria a developer implements against. A notebook that meets them is a notebook that satisfies the AS. A notebook that fails them is a notebook that needs re-work — but the *finding* that the notebook failed is recorded in the CFR (as a documentation finding or a remediation finding), and the *current values* against the criteria are recorded in LC-DCR-001 §4.

---

## 3. Artifact schemas

This section specifies each artifact's schema — the columns or fields it must contain, their types, and their required populated state. **This is what a developer implements against.** The current values in each artifact are not specified here; the schema is.

### 3.1 — NB01 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `data_quality_summary.json` | JSON | `run_id`, `notebook_version`, `records`, `features`, `default_rate`, `leakage_count`, `data_quality_score`, `data_quality_rating` | All fields populated; `leakage_count` matches actual leakage features present |
| `feature_inventory.csv` | CSV | `Feature`, `Data_Type`, `Missing_Pct`, `IV`, `Inclusion_Decision`, `Exclusion_Reason`, `Leakage_Flag` | One row per raw column |
| `leakage_findings.csv` | CSV | `feature`, `test`, `value`, `severity` | At least one row per leakage feature |
| `findings_register.csv` | CSV | `id`, `severity`, `finding`, `description`, `evidence_reference`, `evidence_hash`, `remediation`, `owner`, `timeline`, `status` | One row per finding raised in NB01's scan |
| `temporal_audit.csv` | CSV | `Feature`, `Category`, `Available_at_Origination`, `Leakage_Flag` | Every raw feature classified |

### 3.2 — NB02 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `logistic_regression.pkl`, `xgboost.pkl`, `random_forest.pkl`, `lightgbm.pkl` | pickle | Trained model objects | All four models trained and loadable |
| `test_data.pkl` | pickle | `X_val`, `y_val`, `features`, `split_type`, `split_disclosure`, `run_id`, `notebook_version` | Validation set matches the split disclosure (random or walk-forward); `features` is a list of column names |
| `predictions.pkl` | pickle | `{model_name: np.ndarray}` | One key per model; each array is 1D of length `len(X_val)` |
| `performance_metrics.csv` | CSV | `Model`, `AUC-ROC`, `PR-AUC`, `Brier`, `Log_Loss`, `Accuracy`, `Precision`, `Recall`, `F1`, `Optimal_Threshold` | One row per model; all metrics populated |
| `cv_results.csv` | CSV | `Model`, `CV_AUC_Mean`, `CV_AUC_Std` | 5-fold CV results for each model |
| `calibration_metrics.csv` | CSV | `Model`, `HL_Statistic`, `HL_p_value`, `Calibration_Slope`, `Calibration_Intercept`, `Classification`, `Diagnostic` | HL p-value present; diagnostic column flags suspicious values |
| `performance_summary_v2.json` | JSON | `run_id`, `notebook_version`, `authored_by`, `realistic_auc`, `developer_claimed_auc`, `leakage_delta`, `leakage_features_removed`, `best_model`, `all_models`, `calibration`, `split`, `does_not_contain`, `ownership_note` | `does_not_contain` fields absent; `authored_by` absent or null (NB02 must not author a verdict) |
| `findings_register_v2.csv` | CSV | Same schema as NB01 | One row per finding raised in NB02's scan |

### 3.3 — NB03 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `performance_summary_v2.json` | JSON | `run_id`, `notebook_version`, `authored_by: "NB03"`, `risk_score`, `risk_rating`, `recommendation`, `risk_score_components`, `verdict_owner`, `verdict_inherited_by` | `sum(risk_score_components.values()) == risk_score`; `authored_by == "NB03"` |
| `validation_findings_register.csv` | CSV | `id`, `severity`, `finding`, `description`, `evidence_reference`, `remediation`, `owner`, `status` | One row per finding raised in NB03's validation |
| `fairness_validation.csv` | CSV | `Region`, `N`, `Approval_Rate`, `Default_Rate`, `TPR`, `DI_ratio`, `DI_category` | DI computed per FD-05 4/5ths rule; cardinality ≤ 50; groups with N ≥ 30 |
| `ks_metrics.csv` | CSV | `Model`, `KS_Statistic`, `KS_Threshold`, `Interpretation` | KS computed from ROC curve |
| `monotonicity_check.csv` | CSV | `Feature`, `Correlation_with_SHAP`, `Expected_Direction`, `Actual_Direction`, `Status` | Every top-10 SHAP feature checked |
| `leakage_comparison_validation.csv` | CSV | `Version`, `AUC_ROC`, `Basis`, `Status` | Neutral labels (not INVALID/VALID) |

### 3.4 — NB04 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `compliance_scorecard_v2.json` | JSON | `authored_by: "NB04"`, `verdict_inherited_from: "NB03"`, `validation_verdict`, `governance_local`, `compliance` | Verdict matches NB03's; inheritance field present |
| `regulatory_mapping_v2.csv` | CSV | `Framework`, `Requirement_ID`, `Requirement`, `Status`, `Evidence`, `Gap`, `Mitigation` | Status is exact-match `COMPLIANT` / `PARTIALLY COMPLIANT` / `NON-COMPLIANT` |
| `compliance_report_v2.txt` | text | 7 sections (Executive Summary, Regulatory Compliance, Key Results, Limitations, Sign-off, etc.) | No `[OK]` or `[!]` tokens on compliance percentages |

### 3.5 — NB05 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `german_credit_cross_validation.json` | JSON | `run_id`, `notebook_version`, `lending_club`, `german_credit`, `coverage`, `claim_tested`, `direction_narrative`, `limitations` | `lending_club.post_mitigation_DI` is `null` (not assessed); coverage ledger complete |
| `german_credit_cross_validation.csv` | CSV | `dataset`, `protected_attribute`, `baseline_DI`, `mitigation`, `post_mitigation_DI`, `delta_DI` | LC `post_mitigation_DI` is NaN (not assessed) |

---

## 4. Contract enforcement

The contract chain is the specification's enforcement mechanism. Every stage asserts its upstream contracts at start; a missing or malformed contract raises an error before any analysis runs.

NB01 → data_quality_summary.json + feature_inventory.csv + findings_register.csv → NB02
NB02 → predictions.pkl + test_data.pkl + performance_summary_v2.json (no verdict) → NB03
NB03 → performance_summary_v2.json (with verdict) + fairness_validation.csv + validation_findings_register.csv → NB04, NB05
NB04 → compliance_scorecard_v2.json → terminal
NB05 → german_credit_cross_validation.json → terminal


**Every stage asserts its upstream contracts before reading.** A missing contract raises `FileNotFoundError`; a malformed contract raises a validation error; a verdict-ownership violation raises an assertion error. The specification is enforced at runtime, not in prose.

**A note on the contract pattern.** The contract is a *machine-readable* artifact (JSON or pickle), not a prose summary. A developer who changes the artifact's schema without updating the downstream notebooks breaks the contract — and the downstream notebooks' `FileNotFoundError` is the specification's protection against silent drift.

---

## 5. Reproduction and versioning

### 5.1 — Requirements

| Requirement | What the specification requires |
|---|---|
| Random seed | The pipeline uses a project-wide seed (`RANDOM_STATE = 42` per AS-001 §8) for all stochastic operations |
| Content hash | Each notebook logs a SHA-256-16 content hash of its final artifact, so a re-run can be compared to the prior run |
| Manifest | Each notebook writes a per-notebook manifest listing every artifact it produced, with hashes |
| Audit log | Each notebook writes a per-run audit log, timestamped and versioned |
| `run_id` | Every artifact carries a `run_id` (timestamp-based) |
| `notebook_version` | Every artifact carries a `notebook_version` (semantic version, e.g., `2.1`) — **the version is taken from the notebook header, not fixed in the AS** |

### 5.2 — On the reproducibility requirement

Any re-run with the same seed, same data, and same notebook versions produces the same content hash. A different content hash means the input data, the cleaning, or the code changed. **The content hash is the reproduction's fingerprint** — a reviewer comparing two runs can compare hashes to know whether the analysis is the same.

**On the `notebook_version` field.** The AS does not specify the version value — the version is a property of the notebook, not of the specification. A notebook that bumps to 2.2 changes the artifact's version field to 2.2; the AS's requirement is that the field exists and is populated, not what value it holds.

---

## 6. Open items and dependencies

### 6.1 — Open items (AS-specific)

These are items about the *specification itself* — gaps or imprecisions in the AS. They are distinct from findings (which are in the CFR) and from methodology limitations (which are in the MD).

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **AS-001 does not yet specify the schema for the `Evidence_Hash` column in NB01's findings register.** The column exists (from the FR-07 fix) but the AS doesn't require it. | Documented here; a future AS revision would add it to §3.1. | MINOR version bump |
| 2 | **AS-001 does not specify the schema for the `Diagnostic` column in NB02's `calibration_metrics.csv`.** The column exists but its values (e.g., "SUSPICIOUS — HL p-value is exactly 0 or 1") are not specified. | Documented here; a future AS revision would specify the allowed values. | MINOR version bump |
| 3 | **AS-001 does not specify the format of the `split_disclosure` field in `test_data.pkl`.** The field is a dict; the AS does not specify its required keys. | Documented here; a future AS revision would specify the dict schema. | MINOR version bump |

**On the distinction from findings and methodology limitations.** These items are gaps in *the specification*, not gaps in *the model* or *the methodology*. They do not appear in the CFR because the CFR tracks findings against the model; they do not appear in the MD's limitations because the MD tracks methodology limitations. The AS's open items are its own.

### 6.2 — Items in other documents (not AS open items)

The following items appeared in v2.1's §8 as "open items." They are not AS open items — they belong to other documents — but they are noted here so a reader who was looking for them in the v2.1 AS can find them:

| v2.1 item | Actual location |
|---|---|
| "NB01 data quality score is 58.9 (borderline)" | This is a *finding* (F-01.07 and F-01.08 are related); the score itself is in LC-DCR-001 §4. It is not an AS open item. |
| "NB02 perf_rating vs DCR §1 bands" | This is a *documentation gap* in the MD's use of bands; it is a candidate for a documentation finding in the CFR, not an AS open item. |
| "NB05 random_state=42 for its own train/test split" | This is a *methodology detail* — it belongs in the MD's cross-validation section, not the AS. |

### 6.3 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | LC-PDR-001, LC-DS-001, LC-FDL-001, LC-MD-001 (all referenced by the AS) |
| Downstream | All five notebooks implement the AS; the analysis code and results record what they produced |
| Reproducibility | AS version is recorded in every notebook's header |

---

*End of document.*

