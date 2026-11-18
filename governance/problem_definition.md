---
Artifact ID: LC-A01-PDR
Version: 2.1
Run ID: 20261118_132900
Document Title: Problem Definition Record
Project: Lending Club Consumer Credit Risk Model Validation
---

# Problem Definition Record (A01)

This is the document that states why the model exists. It answers seven questions:

**1. What business problem does the model address?**

The bank's collections team has limited resources and a large book of current loans. It needs a prioritization model: rank current loans by the probability they will become seriously delinquent in the near term, so that collections effort goes to the accounts most likely to benefit. The model does not decide whether to lend — origination is historical and fixed in this dataset. It decides where to look first among loans that already exist.

**2. What decisions does the model support?**

It states the decisions the validation supports:

| # | Decision | Decision-maker | Frequency |
|---|---|---|---|
| D1 | Whether the model may be used for portfolio risk assessment | Model Risk Committee | Annual |
| D2 | Whether the model's stated performance claim is accurate | Model owner | Once, at validation |
| D3 | Whether the model's disparate-impact exposure is acceptable | Compliance | Once, at validation |
| D4 | Whether the model may be used for individual loan decisions | Credit team | Not supported — see below |

**3. What decisions does the model not support?** 

"The validation does not support: individual loan approval or rejection (the model is not calibrated for individual decisions, and the finding recommends against it); loan pricing (set by the platform's risk-based pricing engine); regulatory capital calculations; compensation or performance evaluation; any decision affecting a specific borrower without human review."

**4. What is the population and unit of analysis?**

* Population: U.S. consumer loan applicants who were approved and funded by Lending Club in the observed window (2018 snapshot in this sample).
* Unit of analysis: One loan (one borrower-loan pair). Not the borrower — a borrower with two loans contributes two rows.

**5. What is the target?**

| Status | Class |
|---|---|
| Charged Off, Late (31–120), In Grace Period, Late (16–30) | 1 — default |
| Fully Paid, Current | 0 — performing |
| Anything else | Excluded from training and evaluation |

Threshold: 16+ days, deliberately not 30+. Rationale: collections prioritization acts at the first sign of distress; 30+ is an IFRS 9 provisioning threshold, not an operational one.

Class balance (this sample): 1.78% default (178 of 10,000). Severe imbalance — the model must be evaluated on precision/recall and PR-AUC, not accuracy.

Documented gap: the 16+ day threshold is not IFRS 9-compliant. Recorded as Finding F-03.10 and treated as a scope restriction, not a defect — the model never claimed to be an IFRS 9 model.

**6. What are the constraints?**

These criteria were set as the standard against which the model would be judged. The measured results against each criterion are recorded in the DCR (LC-DCR-001) and in the Validation Report (MAP-VR-LC-001), not in this PDR. The separation is deliberate: a PDR that contains its own results cannot demonstrate that the criteria were pre-registered.

| Criterion | Target | Source | What "fail" would mean |
|---|---|---|---|
| AUC-ROC | $\ge 0.65$ | DCR §1 | Model discrimination below the pre-registered floor; deployment not justified on performance grounds alone |
| KS statistic | $\ge 0.30$ (strong discrimination) | DCR §1 | Tail separation insufficient for the stated prioritization use |
| Region-level DI | $\ge 0.80$ | DCR §3 FD-01 | Fairness finding; deployment blocked pending remediation |
| Calibration (HL p) | $> 0.05$ for $\ge 3$ of 4 models | DCR §2 | Predicted probabilities not reliable for threshold-based decisions |
| Precision at operating threshold | Reviewed against business cost, not a fixed number | DCR §1 | No fixed threshold; the analyst reviews precision against the cost of false positives and false negatives at the chosen operating point |

Why the criteria are stated without measured values. A PDR's function is to fix the standard in advance. The standard is only meaningful if it is separable from the outcome. The measured values, the pass/fail determination per criterion, and the overall verdict are recorded in the DCR and the Validation Report, where they can be evaluated against this PDR without circularity.

**7. Scope and exclusions**

**In scope**
- Binary default prediction on the historical Lending Club sample
- Four model classes compared under identical hyperparameters
- Fairness testing on region as a protected-attribute proxy (accepted for internal compliance sign-off)
- Calibration and stability assessment on the held-out set
- SHAP-based direction check against credit-risk theory

**Excluded & Risk-Mitigation / Limitation Notes**

| Exclusion | Reason & Model Risk Limitation / Risk Mitigation |
|---|---|
| IFRS 9 provisioning | Target threshold misaligned (16 vs 30 days); model not designed for ECL. |
| Canadian portfolios | U.S. data only; no validation on Canadian population. |
| Direct race/gender fairness testing | Dataset has no protected attributes; region is a proxy. **Mitigation/Limitation:** Region-level disparate impact analysis serves as an initial screen. While deemed sufficient for internal compliance sign-off, residual fair lending risk remains due to the absence of direct race/gender observation or advanced imputation (e.g., BISG). |
| Macroeconomic stress testing | No macro factors in dataset. |
| Time-based validation | No usable date column after leakage removal (`issue_month` removed per DS-001 §7). **Mitigation/Limitation:** Because temporal out-of-time (OOT) validation is excluded, model performance cannot be verified against economic cycle shifts. This introduces residual temporal model risk, which carries a formal model risk rating limitation in the Validation Report (MAP-VR-LC-001) and must be monitored via post-deployment drift tracking. |
| Production scoring | The model is not approved; no scoring service exists. |

**8. What are the key assumptions and dependencies?**

**Assumptions**
- Historical default patterns in the Lending Club sample are representative of the operational book (notwithstanding the formal limitation regarding temporal out-of-time validation).
- 16+ days is the operationally relevant distress threshold for collections.
- Region is an acceptable operational proxy for protected attributes the model cannot observe, satisfying internal compliance sign-off requirements while retaining residual risk notation.
- The four-model comparison is informative even though only one model (Random Forest) is selected.

**Dependencies**

| Category | Dependency |
|---|---|
| Data | `loans_full_schema.csv` (hash `52872e7d4f8a388b`) — no substitution permitted (DS-001 §2.1) |
| Upstream documents | DS-001 (Data Specification), MD-001 (Methodology), AS-001 (Analytical Specification), DCR (Decision Criteria), FDL (Feature Decisions) |
| Notebooks | NB01 → NB02 → NB03 → NB04 → NB05 (in order; each notebook asserts its upstream contracts) |
| External tools | `fairlearn` (fairness metrics), `shap` (explainability), `scikit-learn`, `xgboost`, `lightgbm` |
| Seed | `RANDOM_STATE = 42` (AS-001 §8) — changing it invalidates all reproducibility claims |

**9. What happens if the model is wrong?**

**What goes wrong without it:** collections effort is allocated by days-past-due buckets alone, which react late. A model that ranks by predicted default probability lets collections intervene before the first missed payment, when recovery is cheaper. If the model is wrong due to temporal instability or proxy bias, collections resources may be misallocated, or disparate impact risks may go unmitigated in deployment.

**10. What this PDR does NOT contain**

This PDR fixes the problem, the intended use, the population, the target, and the pre-registered success criteria. It does not decide:

| Question | Where the answer lives |
|---|---|
| Did the model meet the success criteria? | `LC-DCR-001` (measured thresholds, pass/fail per criterion) and `MAP-VR-LC-001` (verdict) |
| What is the model's risk rating and recommendation? | `MAP-VR-LC-001` (Validation Report, §1 and §7 — including formal limitations for temporal validation and proxy fairness) |
| Is the model fair across protected groups? | `MAP-VR-LC-001` (§4, fairness) and the fairness validation artifact (`outputs/reports/fairness_validation.csv`) |
| Is the model IFRS 9-compliant? | `MAP-VR-LC-001` (§2, conceptual soundness) — and the answer is that it is not, by design, because the target threshold was set for collections, not provisioning |
| What remediation is required, by whom, and by when? | `MAP-RP-001` (Model Risk Remediation Plan) |
| Can the model be deployed? | `MAP-VR-LC-001` (§1, §6) — the Validation Report owns the deployability determination |

The separation is deliberate. A PDR that also contains the verdict is not a PDR; it is a PDR with a Validation Report stapled to it, and a reviewer cannot tell where the pre-registration ends and the post-hoc analysis begins. This PDR ends at the pre-registration. Everything downstream is in the documents listed above.
