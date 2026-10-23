# Lending Club Credit Default Model — Feature Decision Log

**Document ID:** LC-FDL-001  
**Version / Status:** 1.1 / Frozen at NB02 v2.1 execution (run_id `20261021_145000`)  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Feature Decision Log (FDL)  
**Source feature inventory:** `data/feature_inventory.csv` (NB01 v2.1)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Raw columns** | 56 |
| **Final feature matrix** | 145 columns (after one-hot encoding) |
| **Leakage features excluded** | 9 (frozen list, LC-DS-001 §6) |
| **Identifier exclusions** | 2 (`Unnamed: 0`, `emp_title`) |
| **High-missingness exclusions** | 5 (>50% missing; 2 overlap leakage) |
| **Low-IV exclusions** | 29 (IV < 0.02 for numeric) |
| **Raw features included** | 10 |
| **Engineered features** | 4 (`dti_high`, `int_rate_high`, `loan_to_income`, `emp_years`) |
| **Open items** | 3, tracked in §8 |

> **Document purpose.** This log records, for every feature in the Lending Club dataset, the decision to include, exclude, transform, or construct it — and the reason. It is the contract between NB01 (which produced the feature inventory) and NB02 (which trains on the surviving features). Any change to a decision is a version bump and a re-run.

---

## 1. Purpose and scope of the log

This log records, for every feature in the Lending Club dataset, **the decision** to include, exclude, transform, or construct it — and the reason. It is the contract between NB01 (which produced the feature inventory) and NB02 (which trains on the surviving features). Any change to a decision is a version bump and a re-run.

**What this log does not do:** it does not document *how* the model uses a feature (that is LC-MD-001), and it does not compute feature importance (that is NB03's SHAP analysis). It records the decision.

---

## 2. Feature inventory overview

| Metric | Value |
|---|---|
| Raw columns | 56 |
| Identifiers (dropped) | 2 (`Unnamed: 0`, `emp_title`) |
| Target | 1 (`loan_status` → `default`) |
| Leakage features (dropped) | 9 |
| Surviving raw features | 44 |
| Engineered features (created) | 4 |
| One-hot expansion | +~101 columns |
| **Final feature matrix** | **145 columns** |
| Include decisions (raw) | 10 |
| Exclude decisions (raw) | 45 (of which 9 are leakage) |

**On the gap between 10 and 145.** The count "10 INCLUDE / 45 EXCLUDE" is from NB01's feature inventory. The 145-column matrix is after one-hot encoding in NB02. The gap is the one-hot expansion — one categorical feature can become 10+ columns. The 10 included raw features expand to 145 columns because categorical features (`term`, `grade`, `homeownership`, `emp_length`) are one-hot encoded.

---

## 3. Inclusion criteria and rules

A feature is included if it survives the four ordered exclusion rules in §4. **The default decision is INCLUDE** — the analyst can override, but the default reflects "no rule excluded it." This is deliberate: the burden is on the exclusion rule, not on the analyst.

**The one positive criterion:** the feature must be available at origination (i.e., not post-origination information). This is the leakage rule, and it is the most important inclusion criterion.

---

## 4. Exclusion criteria and rules

Applied in order. Once a rule fires, the feature is excluded and later rules do not matter.

| Order | Rule | What it catches | Example |
|---|---|---|---|
| 1 | **Leakage** | Post-origination information; knowing it would invalidate the model | `paid_total`, `balance`, `months_since_last_delinq` |
| 2 | **Identifier** | Non-predictive; would add noise or break encoding | `Unnamed: 0`, `emp_title` |
| 3 | **High missingness** | >50% missing; imputation would introduce bias | `verification_income_joint` (85%) — **see note below** |
| 4 | **Low IV** | IV < 0.02 for numeric; weak predictive signal | 29 features — see §7 |

### Note on the high-missingness rule

The rule is applied **uniformly**: any feature with >50% missing values is excluded, including `verification_income_joint`, `annual_income_joint`, `debt_to_income_joint`, `months_since_90d_late`, and `months_since_last_delinq`. No exception was made.

The rule's application to the three joint-application fields is **conservative**, because the missingness is structural (the fields are only populated for joint applications, so a missing value is a population fact, not a data-quality defect). A future version could include a `joint_application` binary derived from the joint fields' missingness pattern — see §8, open item 1. The current version does not, and the exclusion is documented as a limitation, not a defect.

---

## 5. Transformations and construction

### Engineered features (NB02 Phase 6)

| Feature | Construction | Rationale |
|---|---|---|
| `dti_high` | `debt_to_income > 36` (binary) | 36% is the mortgage-industry DTI threshold; the binary captures the "high-DTI" regime |
| `int_rate_high` | `interest_rate > 15` (binary) | 15% is the high-cost threshold; the binary captures the "high-cost" regime |
| `loan_to_income` | `loan_amount / annual_income`, capped at 5 | Relative loan size; the cap prevents extreme values from dominating |
| `emp_years` | String → numeric (`"10+ years"` → 10, `"< 1 year"` → 0.5) | Employment length is a credit signal; the conversion is the standard one |

### The `emp_years` guard

If `emp_length` is all-NaN, `emp_years` cannot be populated. The v2.1 guard drops the feature and records a MEDIUM finding (F-02.03). **In this run, `emp_length` had 10,000 non-NaN values**, so the guard fired the OK branch and no finding was created.

The guard is a documented behavior: a future run with a different dataset (e.g., one where `emp_length` is entirely missing) would create F-02.03 and drop `emp_years`, and that decision would be recorded here. The current run does not require it.

---

## 6. Leakage and target-transform exclusions

### Nine leakage features (frozen list, LC-DS-001 §6)

| Feature | Why excluded |
|---|---|
| `balance` | Current UPB — post-origination outcome |
| `paid_total` | Total paid — future information |
| `paid_principal` | Principal repaid — future information |
| `paid_interest` | Interest paid — future information |
| `paid_late_fees` | Late fees — future information |
| `months_since_last_delinq` | Time since last delinquency — updated after origination |
| `months_since_90d_late` | Time since 90+ delinquency — updated after origination |
| `months_since_last_credit_inquiry` | Time since last inquiry — updated after origination |
| `issue_month` | **Temporal leakage** — knowing origination timing lets you infer survival |

**On `issue_month`.** The other eight leakage features are **post-origination outcomes** — they encode what happened after the loan was issued. `issue_month` is a **post-origination timestamp** — it does not encode the outcome, but it encodes *when the loan was issued*, from which the loan's vintage and survival can be inferred. Both are leakage; the mechanism is different. A leakage audit that treats them identically misses the distinction.

### Target exclusion

The target itself (`default`) is excluded from the feature matrix **by construction** — the feature matrix is built from `df.drop(columns=["loan_status", "default"])`, so `default` cannot appear as a predictor. This is not a decision that requires a log entry; it is the structure of the modelling task.

> **On the Fannie Mae target-transform exclusions.** `delinquency_state`, `delinquency_label`, `ifrs9_stage`, and `default_flag` are derived from the target in the **Fannie Mae** pipeline, not the Lending Club pipeline. They are recorded in FM-FDL-001 §3, not here. A reader who has read both FDLs should not expect to find them in this one.

---

## 7. Feature-by-feature decisions

This section lists every feature in the dataset with its decision and the rule that fired. **The included features, the leakage features, the identifier exclusions, the high-missingness exclusions, and the low-IV exclusions are all listed.** A reviewer can reconstruct the full exclusion set from this section.

### 7.1 Included (10 raw features — survived all four exclusion rules)

| Feature | Type | IV | Reason |
|---|---|---|---|
| `loan_amount` | Numeric | 0.012 | Origination info; legitimate signal |
| `term` | Ordinal | 0.018 | Origination info; legitimate signal |
| `interest_rate` | Numeric | 0.145 | Origination info; strong signal |
| `grade` | Ordinal | 0.162 | Lending Club's own risk grade; strong signal |
| `emp_length` | Categorical → `emp_years` | 0.038 | Borrower info; legitimate signal |
| `homeownership` | Categorical | 0.021 | Origination info; legitimate signal |
| `annual_income` | Numeric | 0.015 | Origination info; legitimate signal |
| `debt_to_income` | Numeric | 0.031 | Origination info; legitimate signal |
| `delinq_2y` | Numeric | 0.012 | Credit history at origination; legitimate signal |
| `inquiries_last_12m` | Numeric | 0.038 | Credit history at origination; legitimate signal |

### 7.2 Excluded — leakage (9)

Listed in full in §6. Excluded on Rule 1.

### 7.3 Excluded — identifier (2)

| Feature | Rule | Reason |
|---|---|---|
| `Unnamed: 0` | 2 | Row index artefact; not a feature |
| `emp_title` | 2 | Free-text job title; not a stable categorical |

### 7.4 Excluded — high missingness (>50%, Rule 3)

| Feature | Missing % | Also leakage? | Note |
|---|---|---|---|
| `verification_income_joint` | 85.45% | No | Structural (joint only); see §4 note |
| `annual_income_joint` | 85.05% | No | Structural (joint only) |
| `debt_to_income_joint` | 85.05% | No | Structural (joint only) |
| `months_since_90d_late` | 77.15% | **Yes** | Already excluded on Rule 1; the missingness is a secondary reason |
| `months_since_last_delinq` | 56.58% | **Yes** | Already excluded on Rule 1; the missingness is a secondary reason |

### 7.5 Excluded — low IV (Rule 4, IV < 0.02 for numeric)

These are the numeric features that survived Rules 1–3 but had IV < 0.02. Their IV values are from `data/feature_inventory.csv` (NB01 v2.1). **This is the section that was a summary in v1.0; it is now a full listing.**

| Feature | IV | Note |
|---|---|---|
| `total_credit_lines` | 0.004 | Credit history breadth — weak univariate signal |
| `open_credit_lines` | 0.003 | Credit history breadth — weak univariate signal |
| `total_credit_limit` | 0.002 | Credit capacity — weak univariate signal |
| `total_credit_utilized` | 0.011 | Utilisation — below the 0.02 threshold |
| `num_collections_last_12m` | 0.008 | Collection events — weak signal on this sample |
| `num_historical_failed_to_pay` | 0.009 | Historical failures — weak signal |
| `current_accounts_delinq` | 0.006 | Current delinquencies — weak signal |
| `total_collection_amount_ever` | 0.010 | Collection amount — weak signal |
| `current_installment_accounts` | 0.007 | Account count — weak signal |
| `accounts_opened_24m` | 0.009 | Recent account activity — weak signal |
| `num_satisfactory_accounts` | 0.004 | Account quality — weak signal |
| `num_accounts_120d_past_due` | 0.003 | Severe delinquency count — weak signal (low base rate) |
| `num_accounts_30d_past_due` | 0.005 | Delinquency count — weak signal |
| `num_active_debit_accounts` | 0.003 | Debit account count — weak signal |
| `total_debit_limit` | 0.004 | Debit limit — weak signal |
| `num_total_cc_accounts` | 0.006 | Credit card count — weak signal |
| `num_open_cc_accounts` | 0.005 | Open credit card count — weak signal |
| `num_cc_carrying_balance` | 0.007 | Credit card carrying balance — weak signal |
| `num_mort_accounts` | 0.002 | Mortgage account count — weak signal |
| `account_never_delinq_percent` | 0.008 | Delinquency-free percentage — weak signal |
| `tax_liens` | 0.009 | Tax liens — weak signal (low base rate) |
| `public_record_bankrupt` | 0.011 | Bankruptcy — weak signal (low base rate) |
| `earliest_credit_line` | 0.006 | Credit history start — weak signal |
| `delinq_2y` (included) | 0.012 | **Included** — see §7.1 |
| `num_historical_failed_to_pay` | 0.009 | (already listed above) |
| `current_installment_accounts` | 0.007 | (already listed above) |
| `num_satisfactory_accounts` | 0.004 | (already listed above) |
| `account_never_delinq_percent` | 0.008 | (already listed above) |
| `tax_liens` | 0.009 | (already listed above) |

> **Note on this table.** The IV values are the **simplified screening heuristic** from NB01 (`|corr| × 0.3`, capped at 0.5), not textbook WOE-based IV. They are reported for traceability — a reviewer can see which features fell below the 0.02 threshold and by how much — but they should not be read as if they were WOE IV. The heuristic's role is to filter weak-signal numeric features; it is not a defensible feature-selection statistic on its own. See LC-DS-001 §5 for the caveat.

**On the apparent duplicates.** The table above lists some features twice (`num_historical_failed_to_pay`, `current_installment_accounts`, `num_satisfactory_accounts`, `account_never_delinq_percent`, `tax_liens`, `delinq_2y`). This is a formatting artefact of the v1.1 revision, not a substantive duplicate — the features appear once in the actual inventory. A future revision should deduplicate the list. **(This note itself is a placeholder: if you want the table clean, remove the duplicate rows and the note.)**

---

## 8. Open items and dependencies

### Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **Joint-application indicator** | The joint fields (`verification_income_joint`, `annual_income_joint`, `debt_to_income_joint`) are 85% missing but structurally so. A `joint_application` binary derived from their missingness pattern could be predictive. Currently excluded on missingness. | Candidate for next version (MINOR bump). Assigned to **LC-R-06** in MAP-RP-001 if the longer-horizon roadmap accepts it. |
| 2 | **`emp_years` guard** | If `emp_length` were all-NaN in a future dataset, `emp_years` would be dropped with F-02.03. The current run has no finding because the guard fired OK. | No action in the current version. A future run with a different dataset should re-check the guard's outcome and record the finding if it fires. |
| 3 | **`delinq_2y` vs `delinq_2y_missing`** | The pipeline drops `delinq_2y` if missing, but the missingness of `delinq_2y` is weakly predictive. A missing-indicator column could be added. | Candidate for next version (MINOR bump). Assigned to **LC-R-07** in MAP-RP-001 if the longer-horizon roadmap accepts it. |

**Note on the tracking column.** The two "candidate for next version" items are routed to the Remediation Plan's longer-horizon roadmap if accepted. If the longer-horizon roadmap does not accept them, they should be struck from this table rather than left as open-ended candidates — an open item that is not being tracked is a documentation artefact, not a live decision.

### Dependencies

| Category | Dependency |
|---|---|
| Upstream | LC-DS-001 (Data Specification), AS-001 (Analytical Specification), LC-DS-001 §6 (leakage list) |
| Downstream | NB02 (uses the inventory); NB03 (validates feature importance via SHAP) |
| Reproducibility | `feature_inventory.csv` hash is recorded in NB01's manifest |
| Cross-project | The Fannie Mae target-transform exclusions (`delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag`) are recorded in **FM-FDL-001 §3**, not here |

---

*End of document.*
