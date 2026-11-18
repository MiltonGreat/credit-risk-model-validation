---
Artifact ID: LC-A03-DS
Version: 1.1
Run ID: 20261118_134600
Document Title: Data Specification
Project: Lending Club Consumer Credit Risk Model Validation
---

# Data Specification (A03)

## 1. Where does the data come from?
* **Source name:** Lending Club Loan Data snapshot (`loans_full_schema.csv`, hash `52872e7d4f8a388b`).
* **Access method:** Public download from Lending Club historical data repositories, secured and locked for this project.
* **Format:** CSV.
* **Available records:** Exactly 10,000 loan records representing a curated sample of the broader historical Lending Club book.
* **Licence:** Public dataset; terms permit analytical and model validation use.
* **Substitution constraint:** No substitution of the data file is permitted (DS-001 §2.1).

## 2. What was included, and what was excluded?
* **Population:** U.S. consumer loan applicants who were approved and funded by Lending Club in the observed sample window.
* **Unit of analysis:** One loan (one borrower-loan pair). A borrower with multiple loans contributes multiple rows.
* **Selection & Coverage:** The 10,000-row file serves as the authoritative sample for this validation framework. Its exact historical draw mechanism is undocumented in the public release, constituting a documented limitation (L8) regarding population representativeness.
* **Exclusions at source:** Applications that were declined, withdrawn, or never funded are absent from the dataset. The analysis is therefore conditional on prior credit approval—a inherent selection bias and documented limitation of using the Lending Club public sample as a credit proxy (L6).

## 3. How was the sample drawn and mapped to the target?
Unlike historical default models built on fully matured, closed-out 36-month portfolios, this collections prioritization framework evaluates a snapshot where loans exist across multiple active and distressed states.
* **Target Mapping (per A01 §5):**
  * **Class 1 (Default / Distress):** Charged Off, Late (31–120 days), In Grace Period, Late (16–30 days). Threshold set at 16+ days for operational collections prioritization.
  * **Class 0 (Performing):** Fully Paid, Current.
  * **Exclusions:** Any status falling outside these categories is excluded from training and evaluation.
* **Class Balance:** Approximately 1.78% default rate (178 defaults out of 10,000 records), requiring evaluation metrics specialized for severe class imbalance (PR-AUC, precision/recall) rather than naive accuracy.

## 4. How is the data split for development and validation?
* **Split Type:** Random hold-out split (e.g., 80% training / 20% validation, or equivalent partition as executed in NB02).
* **Random Seed:** Governed strictly by project-wide `RANDOM_STATE = 42` (AS-001 §8) to ensure full reproducibility.
* **Rationale & Scope Limitation:** Per A01 §7 and temporal audit requirements, time-based out-of-time (OOT) validation is **excluded** because the `issue_month` column was removed during leakage detection to prevent data leakage. Consequently, model generalization is tested via random hold-out partition rather than vintage time splits, carrying a formal model risk limitation regarding economic cycle stability.

## 5. What could leak between training and evaluation?
To prevent target leakage and post-origination hindsight bias, the following fields are strictly excluded from feature sets during NB01 and NB02:
* **Payment & Recovery Fields:** `total_pymnt`, `total_rec_prncp`, `total_rec_int`, `total_rec_late_fee`, `recoveries`, `collection_recovery_fee`, `last_pymnt_d`, `last_pymnt_amnt`, `last_credit_pull_d`, `last_fico_range_high`, `last_fico_range_low`.
* **Temporal / Identification Fields:** `issue_month` (removed to eliminate structural leakage, impacting temporal validation capability).
* **Rationale:** These fields are only observable post-origination or post-default resolution. Including them would artificially inflate model performance metrics and invalidate the prioritization use case.
