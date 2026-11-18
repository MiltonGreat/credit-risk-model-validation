# Artifact A14: Findings
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Finding ID:** F-01  
**Date:** November 18, 2026  

---

## Factual Audit Statement
* **Statement:** The model's leakage-free AUC is **0.631**, below the pre-registered tolerance of **0.70** and **0.234** below the developer's stated claim of **0.865**. The difference is explained by the leakage exclusion.
* **Evidence:** Out-of-time test set; leakage-free feature set; AP-01 and AP-08.
* **Source:** A08, A11, A13.
* **Limitation:** Single out-of-time period; model not calibrated.
* **Risk Significance:** High. The model's stated performance is not supported by leakage-free evidence.
* **Materiality:** Material.
* **Root Cause:** The developer's claim appears to include post-origination fields; the leakage exclusion removes them.
* **Recommendation:** Do not approve the model for portfolio risk assessment. Require the developer to re-state the performance claim on a leakage-free feature set.
* **Escalation:** ES-01, Model Risk Lead.
