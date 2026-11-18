# Artifact A15: Write the Challenge Record
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Scope:** Lending Club & German Credit Datasets (Leakage-Fixed Execution)  
**Date:** November 18, 2026  

---

## Potential Invalidation Vectors & Reviewer Challenges

1. **Class Imbalance & Probability Calibration (Lending Club):**
   * *Challenge:* With a default rate of ~1.78%, uncalibrated probability outputs from class-weighted models can severely overstate absolute default risk, distorting economic capital calculations and cut-off threshold selections.
   * *Invalidation Risk:* If probability outputs are used directly for Expected Loss (EL) pricing without Isotonic Regression or Platt Scaling adjustments, credit decisions will misallocate capital.

2. **Geographic Aggregation Abstraction (Disparate Impact):**
   * *Challenge:* Aggregating fairness metrics at the Census Region level masks localized state-level or metropolitan statistical area (MSA) economic variations.
   * *Invalidation Risk:* Regional aggregation may create false positives (masking localized compliance compliance) or false negatives (hiding state-specific disparate impact violations under ECOA).

3. **Cross-Jurisdictional Generalizability (German Credit):**
   * *Challenge:* Does validating proxy-removal mitigation on a 1,000-row European credit dataset ($N=1,000$) prove efficacy for a large-scale US retail credit portfolio ($N>100,000$)?
   * *Invalidation Risk:* Differences in underwriting criteria, regulatory definitions of default, and macroeconomic structures mean German Credit results establish conceptual proof rather than direct operational equivalence.

4. **Feature Staleness & Macroeconomic Shift:**
   * *Challenge:* Models trained on historical static snapshots do not account for post-model economic shocks (e.g., interest rate shifts or inflation cycles).
   * *Invalidation Risk:* Rapid degradation of model stability and rank-ordering power over out-of-time (OOT) validation periods.
