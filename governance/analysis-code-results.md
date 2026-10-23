# Lending Club: Analysis Code and Results

**Document ID:** LC-ACR-001  
**Version / Status:** 2.2 / Frozen at NB05 v2.2 execution (run_id `20261021_145018`)  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Analysis Code and Results (ACR)  
**Source notebooks:** NB01 v2.1 · NB02 v2.1 · NB03 v2.1 · NB04 v2.1 · NB05 v2.2  
**Companion documents:** LC-AS-001 (Analytical Specification) · LC-DCR-001 (Decision Criteria Register) · LC-CFR-001 (Consolidated Findings Register) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Index what code was run and what it produced; evidence that the code matches the AS |
| **Notebooks indexed** | NB01 v2.1 · NB02 v2.1 · NB03 v2.1 · NB04 v2.1 · NB05 v2.2 |
| **Run IDs** | NB01 `20261021_144955` · NB02 `20261021_145000` · NB03 `20261021_145017` · NB04 `20261021_145017` · NB05 `20261021_145018` |
| **Approximate code size** | ~4,500 lines across 5 notebooks (see §2.1 for measurement) |
| **Where the measured values live** | LC-DCR-001 §4 — **not in this index** |
| **Where the findings live** | LC-CFR-001 §3 — **not in this index** |

> **Document purpose.** This deliverable records *what code was run* and *what it produced*. It indexes the five notebooks, records their versions and run IDs, summarizes their outputs, and provides the evidence that the code matches the Analytical Specification (LC-AS-001).
>
> **What this document does not do.** It does not reproduce the code (the notebooks are the code). It does not re-derive the results (LC-DCR-001 §4 is the authoritative record of measured values). It does not record the findings (LC-CFR-001 §3 is the authoritative register). It is an index and a coverage check, not a parallel record of results or findings.

---

## 1. Purpose and scope

This deliverable records what code was run and what it produced. It indexes the five notebooks, records their versions and run IDs, summarizes their outputs, and provides the evidence that the code matches the Analytical Specification (LC-AS-001).

**What it does not do:**

- It does not reproduce the code (the notebooks are the code).
- It does not re-derive the results (LC-DCR-001 §4 is the authoritative record).
- It does not record the findings (LC-CFR-001 §3 is the authoritative register).
- It does not evaluate model quality (MAP-VR-LC-001 is the validation narrative).

It is an index and a coverage check, not a replacement for the other documents.

**On the relationship between this document and the DCR/CFR.** The ACR's §3 summarizes *what the notebooks produced* — the artifacts and their names, the run IDs, and the shape of the output. The *values* in those artifacts (AUC 0.631, DI 0.5414, etc.) are recorded in LC-DCR-001 §4, where they are compared against criteria. The *findings* derived from those values are recorded in LC-CFR-001 §3. The ACR does not restate either; it points to both.

---

## 2. Code inventory

### 2.1 — Notebooks

| Notebook | Version | Run ID | Purpose | Approximate lines |
|---|---|---|---|---|
| NB01 | 2.1 | `20261021_144955` | EDA + data quality | ~800 |
| NB02 | 2.1 | `20261021_145000` | Model development | ~1,200 |
| NB03 | 2.1 | `20261021_145017` | Independent validation | ~900 |
| NB04 | 2.1 | `20261021_145017` | Governance & compliance | ~700 |
| NB05 | 2.2 | `20261021_145018` | German Credit cross-validation | ~900 |

**Total:** approximately 4,500 lines across 5 notebooks.

**On the line counts.** The counts are approximate. They were measured by counting non-blank lines in each notebook's `.ipynb` file, excluding cells that are pure markdown. The counts are stated for orientation, not as a precise measurement — a reviewer who wants an exact count can measure the notebooks directly.

### 2.2 — Notebook headers

Each notebook's header records:

- **Version number** — e.g., `NOTEBOOK_VERSION = "2.1"`
- **Random seed** — `RANDOM_STATE = 42`
- **List of edit markers** — `E-MXX-XX` entries documenting what changed and why
- **What breaks if changed** — the downstream contracts that the notebook's outputs satisfy
- **What the notebook does NOT do** — the scope boundaries

The header is the code's specification summary. A reviewer who reads only the header knows what the notebook is for, what it is not for, and what depends on it.

---

## 3. What the notebooks produced

This section records **what artifacts the notebooks produced** — the artifact names, the run IDs, and the shape of the output. **The measured values in those artifacts are in LC-DCR-001 §4; the findings derived from those values are in LC-CFR-001 §3.** The ACR does not restate either.

### 3.1 — NB01 (EDA + data quality)

**Artifacts produced:** `data_quality_summary.json`, `feature_inventory.csv`, `leakage_findings.csv`, `findings_register.csv`, `temporal_audit.csv`, plus EDA figures.

**Where the values are:** LC-DCR-001 §4.1 (dataset shape, default rate, leakage count, data quality score).

**Where the findings are:** LC-CFR-001 §3 (F-01.01 through F-01.08).

### 3.2 — NB02 (model development)

**Artifacts produced:** Four model pickles (`logistic_regression.pkl`, `xgboost.pkl`, `random_forest.pkl`, `lightgbm.pkl`), `test_data.pkl`, `predictions.pkl`, `performance_metrics.csv`, `cv_results.csv`, `calibration_metrics.csv`, `performance_summary_v2.json` (no verdict), `findings_register_v2.csv`.

**Where the values are:** LC-DCR-001 §4.1 (model performance), §4.2 (calibration).

**Where the findings are:** LC-CFR-001 §3 (F-02.01, F-02.02, F-02.04–F-02.08).

### 3.3 — NB03 (independent validation)

**Artifacts produced:** `performance_summary_v2.json` (with verdict, `authored_by: "NB03"`), `validation_findings_register.csv`, `fairness_validation.csv`, `ks_metrics.csv`, `monotonicity_check.csv`, `leakage_comparison_validation.csv`.

**Where the values are:** LC-DCR-001 §4.3 (fairness), §4.4 (stability), §4.5 (explainability), and the DCR's §5 verdict record.

**Where the findings are:** LC-CFR-001 §3 (F-03.01 through F-03.11).

### 3.4 — NB04 (governance & compliance)

**Artifacts produced:** `compliance_scorecard_v2.json`, `regulatory_mapping_v2.csv`, `compliance_report_v2.txt`.

**Where the values are:** LC-DCR-001 §4.6 (governance).

**Where the findings are:** LC-CFR-001 §3 (NB04 does not produce new findings; it inherits the verdict and reports compliance status).

### 3.5 — NB05 (German Credit cross-validation)

**Artifacts produced:** `german_credit_cross_validation.json`, `german_credit_cross_validation.csv`.

**Where the values are:** LC-DCR-001 §4.5 (the cross-validation's results are a separate methodology check, recorded in the DCR for cross-reference).

**Where the findings are:** LC-CFR-001 §3 (NB05 does not produce new findings; its result is a methodology observation, not a finding against the Lending Club model).

---

## 4. Code quality evidence

The code quality evidence is the patterns that repeat across notebooks. **Each pattern is cited to a code location** so a validator can verify the claim.

| Pattern | Where | Code location (evidence) |
|---|---|---|
| **Audit logger** | All notebooks | `setup_audit_logger(run_id=None)` — one per notebook, distinct log filename prefix (`eda_`, `model_dev_`, `validation_`, `governance_`, `german_credit_`) |
| **File hashing** | All notebooks | `compute_file_hash(path)` — SHA-256-16, chunked read (64 KB blocks), used for every saved artifact |
| **Save with provenance** | All notebooks | `_stamp_and_save(df, path)` — attaches `run_id` and `notebook_version` as columns; `_write_json(obj, path)` — writes JSON with hash logged |
| **Contract assertion** | All notebooks | `REQUIRED = {...}` dict at notebook start; `if not Path(p).exists(): raise FileNotFoundError(...)` — no silent fallback |
| **Conditional findings** | NB02, NB03 | e.g., `if suspicious_hl:` guard around the suspicious-HL finding — the finding is created only if the condition fires |
| **Verdict-ownership chain** | NB02 → NB03 → NB04 → NB05 | NB02 writes `performance_summary_v2.json` without verdict; NB03 asserts no verdict, writes with verdict + `authored_by: "NB03"`; NB04/NB05 assert `authored_by == "NB03"` |
| **Claim-authorization** | NB03, NB05 | NB03 Phase 13: neutral labels for fairness status (not PASS/FAIL); NB05: `[ANALYST: ...]` placeholders and forbidden-token assertions |

**On the evidence column.** The code location column tells a reviewer *where to look* to verify the pattern. A reviewer who reads NB01's `setup_audit_logger` function and sees the same structure repeated in NB02–NB05 has verified the pattern, not just read a claim about it.

---

## 5. Specification-to-code coverage

This section checks whether **every AS item has a code location**. The AS's §2 stage specification lists, for each notebook, the questions it answers and the acceptance criteria; §3 lists each artifact's schema. This section verifies that each of those items is implemented in the code.

**On the distinction from a mapping.** The v2.1 document called this section "Specification-to-code mapping" and presented a 9-row table. That was a *sample* of the mapping, not an enforcement. The v2.2 section is a **coverage check**: it enumerates the AS items and shows, for each, whether the code implements it. A "partial" status means the item is implemented but not fully (e.g., the schema is present but some fields are optional); a "not found" status means the item is not implemented and is a code gap.

### 5.1 — AS stage acceptance criteria (LC-AS-001 §2.2)

| AS item | Code location | Status |
|---|---|---|
| NB01: leakage count matches actual | NB01 Phase 5 (`confirmed_leakage`) | Matched |
| NB01: feature inventory completeness | NB01 Phase 6 (`feature_inventory.csv` writer) | Matched |
| NB01: leakage findings coverage | NB01 Phase 5 (`leakage_findings.csv` writer) | Matched |
| NB01: findings register | NB01 Phase 8 (`findings_register.csv` writer) | Matched |
| NB01: temporal audit | NB01 Phase 5 (`temporal_audit.csv` writer) | Matched |
| NB02: all four models trained | NB02 Phase 10 (four `model.fit()` calls) | Matched |
| NB02: no verdict in summary | NB02 Phase 16 (`does_not_contain` field) | Matched |
| NB02: predictions completeness | NB02 Phase 15 (`predictions.pkl` writer) | Matched |
| NB02: test data provenance | NB02 Phase 15 (`test_data.pkl` writer) | Matched |
| NB02: calibration computed | NB02 Phase 12 (`calibration_metrics.csv` writer) | Matched |
| NB03: verdict authorship | NB03 Phase 14 (`authored_by: "NB03"`) | Matched |
| NB03: component sum | NB03 Phase 12 (`sum(risk_score_components.values()) == risk_score`) | Matched |
| NB03: fairness cardinality | NB03 Phase 10 (region grouping; cardinality check) | Matched |
| NB03: fairness group sufficiency | NB03 Phase 10 (`FD-03` filter) | Matched |
| NB03: KS computed from ROC | NB03 Phase 9 (`ks_metrics.csv` writer) | Matched |
| NB03: SHAP monotonicity | NB03 Phase 7 (`monotonicity_check.csv` writer) | Matched |
| NB03: leakage comparison labels | NB03 Phase 4 (neutral labels) | Matched |
| NB04: verdict inheritance | NB04 Phase 2 (`authored_by == "NB03"` assertion) | Matched |
| NB04: verdict match | NB04 Phase 2 (component sum check) | Matched |
| NB04: regulatory status format | NB04 Phase 4 (exact-match counting) | Matched |
| NB04: no verdict claims in percentages | NB04 Phase 9 (report writer) | Matched |
| NB05: LC post-mitigation is not assessed | NB05 Phase 3 (`post_mitigation_DI: None`) | Matched |
| NB05: LC post-mitigation CSV is NaN | NB05 Phase 12 (comparison DataFrame) | Matched |
| NB05: coverage ledger | NB05 Phase 10 (`coverage_ledger` dict) | Matched |
| NB05: classification rule | NB05 Phase 12 (`direction_classification` derivation) | Matched |

**Coverage: 25 of 25 AS acceptance criteria have a code location.** All matched.

### 5.2 — AS artifact schemas (LC-AS-001 §3)

| AS artifact schema | Code location | Status |
|---|---|---|
| `data_quality_summary.json` schema | NB01 Phase 9 (`data_quality_summary` dict) | Matched |
| `feature_inventory.csv` schema | NB01 Phase 6 (`feature_df` columns) | Matched |
| `leakage_findings.csv` schema | NB01 Phase 5 (`leakage_findings_df` columns) | Matched |
| `findings_register.csv` schema | NB01 Phase 8 (`findings_df` columns) | Matched |
| `temporal_audit.csv` schema | NB01 Phase 5 (`temporal_df` columns) | Matched |
| Four model pickles schema | NB02 Phase 15 (`joblib.dump` calls) | Matched |
| `test_data.pkl` schema | NB02 Phase 15 (`test_data` dict) | Matched |
| `predictions.pkl` schema | NB02 Phase 15 (`predictions` dict) | Matched |
| `performance_metrics.csv` schema | NB02 Phase 11 (`metrics_df` columns) | Matched |
| `cv_results.csv` schema | NB02 Phase 10 (`cv_df` columns) | Matched |
| `calibration_metrics.csv` schema | NB02 Phase 12 (`calibration_df` columns) | Matched |
| `performance_summary_v2.json` schema (NB02) | NB02 Phase 16 (`performance_summary` dict) | Matched |
| `findings_register_v2.csv` schema | NB02 Phase 14 (`findings_df` columns) | Matched |
| `performance_summary_v2.json` schema (NB03) | NB03 Phase 14 (`performance_summary` dict, with verdict) | Matched |
| `validation_findings_register.csv` schema | NB03 Phase 11 (`findings_df` columns) | Matched |
| `fairness_validation.csv` schema | NB03 Phase 10 (`fair_df` columns) | Matched |
| `ks_metrics.csv` schema | NB03 Phase 9 (`ks_metrics_df` columns) | Matched |
| `monotonicity_check.csv` schema | NB03 Phase 7 (`monotonicity_df` columns) | Matched |
| `leakage_comparison_validation.csv` schema | NB03 Phase 4 (`leakage_comparison_df` columns) | Matched |
| `compliance_scorecard_v2.json` schema | NB04 Phase 10 (`scorecard_json` dict) | Matched |
| `regulatory_mapping_v2.csv` schema | NB04 Phase 10 (`mapping_df` columns) | Matched |
| `compliance_report_v2.txt` structure | NB04 Phase 9 (`report_lines` list) | Matched |
| `german_credit_cross_validation.json` schema | NB05 Phase 14 (`cross_validation_summary` dict) | Matched |
| `german_credit_cross_validation.csv` schema | NB05 Phase 12 (`comparison_df` columns) | Matched |

**Coverage: 24 of 24 AS artifact schemas have a code location.** All matched.

### 5.3 — Coverage summary

| Category | Items | Matched | Partial | Not found |
|---|---|---|---|---|
| AS acceptance criteria | 25 | 25 | 0 | 0 |
| AS artifact schemas | 24 | 24 | 0 | 0 |
| **Total** | **49** | **49** | **0** | **0** |

**The code matches the AS.** Every acceptance criterion and every artifact schema has a code location. The v2.1 document asserted this; the v2.2 document demonstrates it.

---

## 6. Reproducibility evidence

The reproducibility evidence is the artifacts that record what was run. **This section lists the artifacts; the artifacts themselves are the evidence.**

| Evidence | Location | Content |
|---|---|---|
| Run IDs | All notebooks | Timestamp-based, unique per execution |
| Content hash | `data_quality_summary.json` | SHA-256-16 of final clean dataframe |
| File hashes | Manifests (`outputs/manifest_*.csv`) | SHA-256-16 of each artifact |
| Audit logs | `logs/*_audit_trail_*.log` | Per-run, per-notebook, timestamped |
| Model pickles | `models/*.pkl` | Trained model objects |
| Predictions | `models/predictions.pkl` | Validation set predictions |
| Content-addressable comparison | All artifact hashes | Same hash across runs = same content |

**A reviewer can reproduce the results from the artifacts alone.** The artifacts are the reproduction. The notebooks are the code that produced them; the artifacts are the evidence that the code produced what the AS specified.

---

## 7. Code-level limitations and known defects

### 7.1 — On the distinction from model/methodology limitations

The model's data limitations are in **LC-DS-001 §8**. The methodology's limitations are in **LC-MD-001 §8**. The findings raised by the validation are in **LC-CFR-001 §3**. **This section records only code-level limitations** — what the code does and does not do, independent of whether the model or methodology is sound.

### 7.2 — Code-level limitations

| ID | Limitation | Where | Impact |
|---|---|---|---|
| CL-1 | **SHAP is computed on a 100-row sample**, not the full validation set | NB03 Phase 7 | The `monotonicity_check.csv` reflects a sample of the model's behaviour; a full-population SHAP would require a different code path (sampling the full set is not implemented) |
| CL-2 | **PSI is `not_assessed` on a random split** | NB03 Phase 6 | The code path for time-based PSI exists but is not exercised in this run; a reviewer cannot verify the time-based path without a time-based dataset |
| CL-3 | **`leakage_comparison_validation.csv` uses neutral labels**, not INVALID/VALID | NB03 Phase 4 | The labels are correct (they avoid a verdict token), but a downstream reader expecting INVALID/VALID may not recognize the neutral form — a documentation dependency, not a code defect |

**On the distinction from model/methodology limitations.** The MD's M3 ("SHAP on 100 rows, indicative") is a *methodology limitation* — it concerns what the SHAP result can support. The ACR's CL-1 ("SHAP is computed on a 100-row sample") is a *code limitation* — it concerns what the code path does. They overlap, but they answer different questions.

### 7.3 — Known code defects

| Defect | Where | Impact | Status |
|---|---|---|---|
| `import warnings` missing | NB04 imports | `NameError` at first run | FIXED (one-line import) |

**The one known defect** was a missing `import warnings` in NB04. It was caught at first run, fixed with a one-line addition, and did not affect any artifact's logic. The fix is documented in the AI Interaction Record.

**On "no further action needed."** The defect is fixed in the current version (v2.1 of NB04). No further action is required. A future version that re-introduces the defect would fail at first run, which is the correct behaviour.

---

## 8. Open items and dependencies

### 8.1 — Open items (ACR-specific)

These are items about the *Analysis Code and Results document itself* — gaps or imprecisions. They are distinct from findings (CFR) and from code-level limitations (this document's §7.2).

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **§2.1's line counts are approximate.** The counts were measured from the `.ipynb` files' non-blank lines (excluding markdown cells), but the measurement method is not auditable from this document alone. | Documented in §2.1; a future ACR revision could add the measurement script as an appendix. | MINOR version bump |
| 2 | **§5's coverage check lists 49 items; the AS may have additional items not captured in §5.1 and §5.2.** If the AS is revised to add items, this coverage table must be updated. | The coverage table is complete for the current AS (v2.2). A future AS revision would require an ACR update. | Follows the AS's version chain |

### 8.2 — Items in other documents (not ACR open items)

The following items appeared in the v2.1 ACR as "open items." They are not ACR open items — they belong to other documents — but they are noted here for reference:

| v2.1 item | Actual location |
|---|---|
| "NB04 import warnings fix" | Fixed; documented in this document's §7.3 and in the AI Interaction Record. Not an open item. |
| "NB02 perf_rating vs DCR §1 bands" | This is a *documentation gap* in the MD's use of bands; it is a candidate for a documentation finding in the CFR, not an ACR open item. |
| "NB05 random_state=42 for its own train/test split" | This is a *methodology detail* — it belongs in the MD's cross-validation section, not the ACR. |

### 8.3 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | LC-AS-001 (the specification that this ACR checks coverage against); the five notebooks (the code being indexed) |
| Downstream | LC-CFR-001 (findings); MAP-VR-LC-001 (verdict); LC-AR-001 (approval decision) |
| Reproducibility | All artifacts carry `run_id` and `notebook_version` |

---

*End of document.*
