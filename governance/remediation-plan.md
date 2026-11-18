# Artifact A16: Remediation Plan
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Scope:** Lending Club & German Credit Datasets (Leakage-Fixed Execution)  
**Date:** November 18, 2026  

---

## Corrective Action & Risk Mitigation Roadmap

### Phase 1: Model Calibration & Probability Refinement
* **Action:** Implement post-hoc probability calibration (Isotonic Regression and Platt Scaling) on all classification models to ensure predicted default probabilities align with true empirical default rates.
* **Timeline:** Pre-Deployment (Milestone 1)
* **Owner:** Quantitative Risk Modeling Team

### Phase 2: Granular Fair Lending & Disparate Impact Redesign
* **Action:** Disaggregate regional disparate impact monitoring from Census Region level down to State-level and MSA-level granularity to satisfy strict ECOA and state-level fair lending guidelines. Implement in-processing adversarial debiasing or constrained optimization for proxy features.
* **Timeline:** Pre-Deployment (Milestone 2)
* **Owner:** Model Risk Management (Second Line) & Fair Lending Officer

### Phase 3: Out-of-Time (OOT) & Stress Testing Validation
* **Action:** Construct rigorous out-of-time (OOT) validation splits covering multiple macroeconomic vintage cycles to evaluate coefficient stability and performance degradation under stress scenarios (aligned with OSFI Guideline E23).
* **Timeline:** Ongoing / Post-Validation Review
* **Owner:** Validation Team

### Phase 4: Production Monitoring & Drift Governance
* **Action:** Establish automated continuous monitoring pipelines tracking Population Stability Index (PSI) for input features and Characteristic Stability Index (CSI) for predicted scores, with mandatory model re-calibration triggers when PSI > 0.25.
* **Timeline:** Post-Deployment Operationalization
* **Owner:** Enterprise Model Operations / Risk IT
