# Artifact A13: Interpretation Record
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Result ID:** R-01  
**Date:** November 18, 2026  

---

## Result Summary
* **Result:** Leakage-free AUC = **0.631** on the out-of-time test set. Developer's stated claim: **0.865**. Difference: **0.234 (27.1%)**.
* **What it measures:** The model's ability to rank-order Fully Paid vs. Charged Off loans on later vintages.
* **What it does not measure:** Calibration (whether predicted probabilities match observed frequencies); individual-loan decision quality; performance on vintages outside 2016–2018.
* **Benchmarks applied:** AP-01 tolerance: AUC $\ge 0.70$. The model fails AP-01 (0.631 < 0.70). The developer's claim of 0.865 is not a benchmark; it is the claim the validation was commissioned to test.
* **Uncertainty:** Moderate; the out-of-time test set is a single period. The standard error of the AUC is approximately 0.01; the difference from 0.70 is not within sampling error.
* **Contradictory evidence:** The developer's claim of 0.865. The difference is explained by the leakage exclusion: the developer's claim appears to include post-origination fields. This is the finding the validation was commissioned to produce.
* **Interpretation:** The model's leakage-free performance is below the pre-registered tolerance. The model is not fit for portfolio risk assessment as specified. The developer's claim is not supported by leakage-free evidence.
* **Limitations:** The model is not calibrated; calibration would require a separate analysis. The out-of-time test is a single period; a second period would strengthen the conclusion.
