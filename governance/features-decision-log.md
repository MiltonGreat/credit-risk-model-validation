---
Artifact ID: LC-A04-FDL
Version: 1.2
Run ID: 20261118_135400
Document Title: Feature Decision Log
Project: Lending Club Consumer Credit Risk Model Validation
---

# Feature Decision Log (A04)

This is the document that describes what is derived from the data, and why. It answers questions:

## 1. Purpose and scope of the log

This log records, for every feature in the Lending Club dataset, the decision to include, exclude, transform, or construct it — and the reason. It is the contract between NB01 (which produced the feature inventory) and NB02 (which trains on the surviving features). Any change to a decision is a version bump and a re-run.

* **What this log does not do:** It does not document how the model uses a feature (that is LC-MD-001), and it does not compute feature importance (that is NB03's SHAP analysis). It records the decision.

---

## 2. Which features are included, and which are excluded?

* **Excluded features:** `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, `issue_month` (post-origination or temporal leakage).
* **Rationale:** These fields are only known after the loan has been repaid, charged off, or dynamically progressed post-origination. Including them would introduce data leakage.

### 2.1. Inclusion criteria and rules
A feature is included if it survives the four ordered exclusion rules in §2.2. The default decision is `INCLUDE` — the analyst can override, but the default reflects "no rule excluded it." This is deliberate: the burden is on the exclusion rule, not on the analyst.

The one positive criterion: the feature must be available at origination (i.e., not post-origination information). This is the leakage rule, and it is the most important inclusion criterion.

### 2.2. Exclusion criteria and rules
Applied in order. Once a rule fires, the feature is excluded and later rules do not matter.

| Order | Rule | What it catches | Example |
|---|---|---|---|
| 1 | Leakage | Post-origination information; knowing it would invalidate the model | `balance`, `paid_total`, `months_since_last_delinq` |
| 2 | Identifier | Non-predictive; would add noise or break encoding | `Unnamed: 0`, `emp_title` |
| 3 | High missingness | >50% missing; imputation would introduce bias | `verification_income_joint` (85%) |
| 4 | Low IV | Univariate IV heuristic < 0.02 for numeric; weak predictive signal | Low-signal credit lines and accounts |

*Note on high-missingness joint fields:* Applied uniformly to `verification_income_joint`, `annual_income_joint`, `debt_to_income_joint`, `months_since_90d_late`, and `months_since_last_delinq`. Treated as a documented limitation rather than a defect.

---

## 3. How is each feature transformed?

### Engineered features (NB02 Phase 6)

| Feature | Construction | Rationale |
|---|---|---|
| `dti_high` | `debt_to_income > 36` (binary) | 36% is the mortgage-industry DTI threshold; captures the "high-DTI" regime. |
| `int_rate_high` | `interest_rate > 15` (binary) | 15% is the high-cost threshold; captures the "high-cost" regime. |
| `loan_to_income` | `loan_amount / annual_income`, capped at 5 | Relative loan size; cap prevents extreme values from dominating. |
| `emp_years` | String → numeric (`"10+ years"` → 10, `"< 1 year"` → 0.5) | Employment length credit signal; standard conversion. |

* **The `emp_years` guard:** If `emp_length` is all-NaN, `emp_years` cannot be populated. The v2.1 guard drops the feature and records a MEDIUM finding (F-02.03). In this run, `emp_length` had 10,000 non-NaN values, so the guard fired the OK branch and no finding was created.

---

## 4. Which features are proxies, and what gap do they introduce?
* **Proxy:** `grade` and `sub_grade`. 
* **Intended concept:** Credit risk. 
* **Gap:** Grades are assigned by Lending Club's proprietary scoring model, not by the analyst's model. The model therefore partially replicates the platform's underwriting. 
* **Mitigation:** The model is scoped as a validation of the platform's risk assessment (see A01 D1/D4); results are reported with and without `grade`/`sub_grade` to show the effect.

---

## 5. Why does each feature matter (business meaning)?

### 5.1 Included (10 raw features — survived all four exclusion rules, with explicit overrides)

| Feature | Type | IV | Reason & Inclusion Status |
|---|---|---|---|
| `loan_amount` | Numeric | 0.012 | **Analyst Override:** Origination size; foundational credit risk factor retained despite mechanical IV < 0.02. |
| `term` | Ordinal | 0.018 | **Analyst Override:** Loan structure/duration; mandatory underwriting dimension retained despite mechanical IV < 0.02. |
| `interest_rate` | Numeric | 0.145 | Origination info; strong direct signal. |
| `grade` | Ordinal | 0.162 | Lending Club's own risk grade; strong signal. |
| `emp_length` | Categorical → `emp_years` | 0.038 | Borrower stability info; legitimate signal. |
| `homeownership` | Categorical | 0.021 | Origination info; legitimate signal. |
| `annual_income` | Numeric | 0.015 | **Analyst Override:** Borrower capacity metric; core affordability factor retained despite mechanical IV < 0.02. |
| `debt_to_income` | Numeric | 0.031 | Origination info; legitimate signal. |
| `delinq_2y` | Numeric | 0.012 | **Analyst Override:** Recent delinquency history; critical credit risk indicator retained despite mechanical IV < 0.02. |
| `inquiries_last_12m` | Numeric | 0.038 | Credit history at origination; legitimate signal. |

### 5.2 Excluded — leakage (9)
`balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, `issue_month`. (Excluded on Rule 1).

### 5.3 Excluded — identifier (2)
* `Unnamed: 0` (Rule 2: row index artefact)
* `emp_title` (Rule 2: free-text job title, non-stable categorical)

### 5.4 Excluded — high missingness (>50%, Rule 3)
* `verification_income_joint` (85.45% missing)
* `annual_income_joint` (85.05% missing)
* `debt_to_income_joint` (85.05% missing)
* `months_since_90d_late` (77.15% missing — also leakage)
* `months_since_last_delinq` (56.58% missing — also leakage)

### 5.5 Excluded — low IV (Rule 4, IV < 0.02 for numeric, excluding overridden core factors)
*Note: IV values reflect the simplified screening heuristic from NB01 (`|corr| × 0.3`, capped at 0.5), not textbook WOE-based IV.*

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
| `num_accounts_120d_past_due` | 0.003 | Severe delinquency count — weak signal |
| `num_accounts_30d_past_due` | 0.005 | Delinquency count — weak signal |
| `num_active_debit_accounts` | 0.003 | Debit account count — weak signal |
| `total_debit_limit` | 0.004 | Debit limit — weak signal |
| `num_total_cc_accounts` | 0.006 | Credit card count — weak signal |
| `num_open_cc_accounts` | 0.005 | Open credit card count — weak signal |
| `num_cc_carrying_balance` | 0.007 | Credit card carrying balance — weak signal |
| `num_mort_accounts` | 0.002 | Mortgage account count — weak signal |
| `account_never_delinq_percent` | 0.008 | Delinquency-free percentage — weak signal |
| `tax_liens` | 0.009 | Tax liens — weak signal |
| `public_record_bankrupt` | 0.011 | Bankruptcy — weak signal |
| `earliest_credit_line` | 0.006 | Credit history start — weak signal |

---

## 6. How stable is each feature?

### Nine leakage features (frozen list)
* **`balance`**: Current UPB — post-origination outcome.
* **`paid_total`**: Total paid — future information.
* **`paid_principal`**: Principal repaid — future information.
* **`paid_interest`**: Interest paid — future information.
* **`paid_late_fees`**: Late fees — future information.
* **`months_since_last_delinq`**: Time since last delinquency — updated after origination.
* **`months_since_90d_late`**: Time since 90+ delinquency — updated after origination.
* **`months_since_last_credit_inquiry`**: Time since last inquiry — updated after origination.
* **`issue_month`**: Temporal leakage — knowing origination timing lets you infer survival. *Note: Unlike post-origination outcomes, `issue_month` is a temporal timestamp letting survival be inferred; both constitute leakage.*

---

## 7. What fairness implications does each feature carry?
`homeownership` and `emp_length` may correlate with protected attributes, introducing indirect disparate impact risk. Fair lending testing is required. *Note: The Lending Club dataset does not contain direct protected-class attributes; regional proxies are tested per framework standards.*
