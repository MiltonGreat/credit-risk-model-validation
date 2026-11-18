# Artifact A14: Write the Findings
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Scope:** Lending Club & German Credit Datasets (Leakage-Fixed Execution)  
**Date:** November 18, 2026  

---

## Factual Audit Statements

1. **Pipeline Execution & Traceability:** 
   * The project successfully executed five discrete notebooks (Notebooks 01 through 05) across **59 cumulative operations**, with immutable Run IDs (`20261118_151827` through `20261118_170144`) cryptographically embedded in all output artifacts (`outputs/*.json`).

2. **Data Integrity & Leakage Resolution:** 
   * Pre-registered post-origination tracking columns (`balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `months_since_last_delinq`, `months_since_90d_late`, `months_since_last_credit_inquiry`, `issue_month`, `emp_title`, `loan_status`, `Unnamed: 0`) were successfully excluded from all Lending Club training pipelines.
   * Target leakage identified in initial German Credit cross-validation runs was fully resolved by explicitly isolating and dropping the raw `Risk` target column (Run ID `20261118_170144`), transitioning model AUCs from an erroneous 1.0000 to realistic out-of-sample metrics (0.7614 – 0.7776).

3. **Quantitative Model Performance:** 
   * Lending Club baseline models yield an out-of-sample AUC-ROC of ~0.635 under extreme class imbalance (~1.78% default rate).
   * German Credit leakage-fixed models yield out-of-sample AUC-ROC metrics of `0.7614` (Logistic Regression), `0.7749` (XGBoost), `0.7776` (Random Forest), and `0.7654` (LightGBM).

4. **Fairness & Compliance Evaluation:** 
   * Validation under DCR v2.2 established a regional disparate impact ratio of **0.5414**, falling strictly within the **SEVERE** disparity tier (< 0.60 threshold).
