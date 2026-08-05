---
layout: default
title: "Model Validation Framework | OSFI E-23 & SR 11-7"
---

# The Model Risk Validation Lifecycle

Model validation is not a one-time test; it is a continuous cycle. Under **SR 11-7** (US) and **OSFI E-23** (Canada), a model is only as good as its ongoing monitoring.

## The 4 Pillars of a Robust Validation Framework

### 1. Conceptual Soundness
Does the model theory make sense? 
- For credit risk, does the Probability of Default (PD) model logically link macro-economic variables (like GDP) to default rates?
- *Regulatory Focus:* You must document *why* you chose XGBoost over Logistic Regression, not just *how*.

### 2. Data Quality & Integrity
This is the #1 reason models fail validation.
- Are there missing values? Are the labels (Default / Non-Default) accurately mapped?
- **Key concept:** "Garbage in, Gospel out." I use Python’s `pandas-profiling` and `ydata-profiling` to automatically generate data quality reports for auditors.

### 3. Outcomes Analysis (Performance)
This is where we test predictive power, but with a twist:
- **Discrimination:** Can the model separate good vs. bad borrowers? (AUC-ROC, KS Statistic).
- **Calibration:** If the model says 5% of this group will default, does exactly 5% actually default? (Hosmer-Lemeshow test).
- **Fairness:** Are we disproportionately rejecting a protected group through proxy variables? (Demographic Parity).

### 4. Ongoing Monitoring (The "Drift" Pillar)
A model built on 2015-2020 data will fail on 2025 data. 
- **Population Stability Index (PSI):** Tracks changes in the input features.
- **Performance Drift:** Tracks changes in the model's accuracy over time.

---

## The "Gap" in the Industry
Many institutions have model inventories that are incomplete. Under E-23, if an Excel spreadsheet is used for credit pricing, **it is a model** and must be validated. The gap is in tracking these.

**My approach:** I use Python to create a simple Model Inventory tracker, linking each model to its validation status, owner, and next review date.

[View my Model Inventory Python script on GitHub](#) *(Link to your repo or a gist)*
