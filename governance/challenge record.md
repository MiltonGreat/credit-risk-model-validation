# Artifact A15: Challenge Record
**Project:** End-to-End Credit Risk Modeling & Validation Framework  
**Scope:** Invalidation Inventory & Sensitivity Analysis  
**Date:** November 18, 2026  

---

## Challenge & Invalidation Analysis
* **Invalidation Inventory:** A different out-of-time period could change the AUC; a different leakage definition could change the feature set; the developer's claim could be based on a different target definition.
* **Alternative Explanations:** The model may be learning the platform's grading process rather than default risk; the AUC difference may reflect a target definition difference rather than leakage.
* **Sensitivity Analysis:** The model was re-run with and without `grade`/`sub_grade`; the AUC is **0.72 with** and **0.66 without**. The leakage-free result of **0.631** is the conservative case.
* **Replication Assessment:** Not replicated independently; the validation is a single-analyst exercise.
* **Adversarial Questions:** 
  > *"Why should we trust your leakage exclusion and not the developer's claim?"*  
  > **Answer:** Because the exclusion is documented, the feature list is checkable against the data dictionary, and the leakage-free AUC is reproducible from the archived output.
* **Falsification Conditions:** The finding would be falsified if the developer produces a leakage-free feature set with an AUC $\ge 0.70$ on the same out-of-time period.
