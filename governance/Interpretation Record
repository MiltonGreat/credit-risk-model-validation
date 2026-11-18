# Artifact A13: Interpretation Record
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Scope:** Lending Club & German Credit Datasets (Leakage-Fixed Execution)  
**Date:** November 18, 2026  

---

## 1. Executive Summary of Quantitative Results
The empirical results across the model development and validation pipelines indicate that standard machine learning classifiers (`Logistic Regression`, `XGBoost`, `Random Forest`, `LightGBM`) achieve baseline predictive power but expose significant fairness and generalization vulnerabilities under institutional risk frameworks (OSFI E23, SR 11-7, and ECOA).

* **Lending Club Baseline & Fairness (Notebooks 01–03):** Models achieved an out-of-sample AUC-ROC of ~0.635 under severe class imbalance (~1.78% default rate). However, validation under DCR v2.2 revealed a **SEVERE regional disparate impact violation (DI 0.5414)**, triggering an automatic formal high/reject risk score (6/10).
* **German Credit Cross-Validation (Notebook 05, Run ID `20261118_170144`):** Following the removal of target leakage, leakage-free models achieved robust out-of-sample AUC-ROC performance ranging between **0.7614 and 0.7776** (Accuracy ~0.67–0.72), confirming that proxy-feature mitigation techniques generalize effectively without relying on ground-truth label contamination.

---

## 2. Economic & Risk Meaning of Findings

### A. Predictive Discrimination vs. Information Value
The modest AUC (~0.635) on Lending Club reflects the inherent noise in prime/near-prime retail credit portfolios where default events are rare. While tree-based ensembles and logistic regressions rank-order risk moderately better than random selection, their operational utility is strictly bounded by calibration drift and class imbalance distortions.

### B. Structural Fairness Risks
The disparate impact ratio (0.5414) in Census Region evaluations indicates that geographic economic structuring creates proxy discrimination against protected demographic groups. Ingestion of regional macro-indicators without explicit mitigation violates fair lending principles (ECOA) by penalizing borrowers based on geographic location rather than individual creditworthiness.

### C. Generalizability of Mitigation
The German Credit cross-validation results (~0.77 AUC) prove that proxy-feature removal (e.g., isolating housing, foreign worker status, and personal status attributes) preserves primary risk-ranking capability while decoupling models from discriminatory proxy channels. This validates the feasibility of fair-lending-compliant feature engineering across distinct regulatory jurisdictions.
