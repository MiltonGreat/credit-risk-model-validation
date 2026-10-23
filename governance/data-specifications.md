# Lending Club Credit Default Model — Data Specification

**Document ID:** LC-DS-001  
**Version / Status:** 1.1 / Frozen at NB01 v2.1 execution (run_id `20261021_144955`)  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Data Specification (DS)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Source** | Lending Club public loan dataset (published, de-identified) |
| **File** | `loans_full_schema.csv` |
| **Records** | 10,000 loans |
| **Raw columns** | 56 |
| **SHA-256-16 hash** | `52872e7d4f8a388b` (recorded in NB01 audit log) |
| **Observed default rate** | 1.78% (178 of 10,000) |
| **Target threshold** | 16+ days past due — **not** IFRS 9's 30+ day threshold |
| **Leakage features** | 9 identified and removed in NB02 (F-03.01, RESOLVED) |
| **Missingness** | 42,191 values across 10 columns; structural (joint fields) and semantic (delinquency) distinguished in §5 |
| **Data quality summary** | Pipeline composite score 58.9/100 — see §5 for what this measures and does not |

> **Document purpose.** This specification describes what the dataset contains and how it was prepared for modelling. It does **not** evaluate model performance, reach a validation verdict, or recommend remediation. Those belong to the Validation Report (MAP-VR-LC-001) and the Remediation Plan (MAP-RP-001).

---

## 1. Data sources and provenance

| Field | Value |
|---|---|
| Source | Lending Club public loan dataset (published, de-identified) |
| File | `loans_full_schema.csv` |
| Records | 10,000 |
| Raw columns | 56 |
| SHA-256-16 hash | `52872e7d4f8a388b` (recorded in NB01 audit log) |
| Access | Local file; no external API; no subscription |
| Licensing | Public dataset, no restriction for methodology demonstration |
| Vintage | 2018 snapshot (loans originated ~2015–2018) |

**Provenance discipline.** The hash is recorded in NB01's audit log at load time. Any re-run with a modified file produces a different hash and the difference is visible in the log. There is **no synthetic fallback** — the notebook raises `FileNotFoundError` if the file is missing (DS-001 §2.1).

---

## 2. Population and coverage

- **Population:** U.S. consumer loan applicants approved and funded by Lending Club, observed at a single snapshot.
- **Coverage:** 10,000 records is a **sample** of the full Lending Club book. Not the full population.
- **Selection:** The 10,000-row file is the sample. Its selection mechanism is not documented in the source materials; this is treated as a **limitation** (L8, §8), because a non-random or unexplained draw may not be representative of the full book. The file is treated as the authoritative sample for this project, not as a reconstruction of how the full book was drawn.
- **Exclusions at source:** applications that were declined (no loan), withdrawn, or never funded are **not in the dataset**. The dataset is therefore **conditional on approval** — a selection bias that is a documented limitation of using Lending Club as a proxy (L6, §8).

> **Why this matters.** A model trained on approved-and-funded loans has never seen a declined application. Its predicted default probabilities are conditional on the applicant having been approved by Lending Club's own underwriting — which is itself a selection mechanism. Any deployment to a different underwriting population requires revalidation.

---

## 3. Unit of analysis and schema

**Unit:** one loan (borrower-loan pair). 10,000 rows = 10,000 loans.

**Primary key:** implicit. No loan ID survives — `Unnamed: 0` is a row index artefact and is dropped in cleaning. The implication is that **individual loans cannot be re-identified** across runs; if loan-level traceability is required (e.g., for a specific defaulted loan), it must be reconstructed from the row position in the frozen sample, which is fragile. This is a documented limitation of the sample, not a defect of the pipeline.

### Schema (56 raw columns)

| Group | Columns | Count |
|---|---|---|
| Identifiers | `Unnamed: 0`, `emp_title` | 2 |
| Target | `loan_status` | 1 |
| Origination characteristics | `loan_amount`, `term`, `interest_rate`, `grade`, `sub_grade`, `issue_month`, `loan_purpose`, `application_type`, `initial_listing_status`, `disbursement_method` | 10 |
| Borrower characteristics | `emp_length`, `state`, `homeownership`, `annual_income`, `verified_income`, `debt_to_income`, `annual_income_joint`, `verification_income_joint`, `debt_to_income_joint` | 9 |
| Credit history | `delinq_2y`, `months_since_last_delinq`, `earliest_credit_line`, `inquiries_last_12m`, `total_credit_lines`, `open_credit_lines`, `total_credit_limit`, `total_credit_utilized`, `num_collections_last_12m`, `num_historical_failed_to_pay`, `months_since_90d_late`, `current_accounts_delinq`, `total_collection_amount_ever`, `current_installment_accounts`, `accounts_opened_24m`, `months_since_last_credit_inquiry`, `num_satisfactory_accounts`, `num_accounts_120d_past_due`, `num_accounts_30d_past_due`, `num_active_debit_accounts`, `total_debit_limit`, `num_total_cc_accounts`, `num_open_cc_accounts`, `num_cc_carrying_balance`, `num_mort_accounts`, `account_never_delinq_percent`, `tax_liens`, `public_record_bankrupt` | 28 |
| Loan-term characteristic | `installment` (scheduled monthly payment — an origination-time term, not a performance field) | 1 |
| Post-origination (leakage) | `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees` | 5 |

**Note on `installment`.** The field is the scheduled monthly payment amount, fixed at origination. It is a **loan-term characteristic**, not a post-origination performance field. A reader scanning the schema for leakage should not flag it — it is available at decision time, and it carries no information about the borrower's actual repayment behaviour.

---

## 4. Target variable and its construction

**Raw column:** `loan_status` (categorical, 6 observed values).

**Plain-language rule.** A loan is classified as **default (1)** if its status is any of: Charged Off, Late (31–120 days), In Grace Period, or Late (16–30 days). It is classified as **performing (0)** if its status is Fully Paid or Current. Any other status is excluded.

| Status | Class |
|---|---|
| Charged Off, Late (31–120), In Grace Period, Late (16–30) | **1 — default** |
| Fully Paid, Current | **0 — performing** |
| Anything else | Excluded from training and evaluation |

**Threshold:** 16+ days past due — deliberately not 30+. IFRS 9 would require 30+ for Stage 3 (see §8, L5). The 16+ threshold is operationally relevant for collections prioritisation; it is **not** an IFRS 9 definition and is not intended to be one.

**Construction (as implemented in NB01 Phase 3):**

```python
bad_statuses  = ['Charged Off', 'Late (31-120 days)', 'In Grace Period', 'Late (16-30 days)']
good_statuses = ['Fully Paid', 'Current']

df['default'] = df['loan_status'].apply(
    lambda x: 1 if x in bad_statuses else (0 if x in good_statuses else np.nan)
)
df_clean = df[df['loan_status'].isin(good_statuses + bad_statuses)]
