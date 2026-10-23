# Lending Club Credit Default Model — Problem Definition Record

**Document ID:** LC-PDR-001  
**Version / Status:** 0.2 / Draft — reconstructed from implementation  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Problem Definition Record (PDR)  
**Document purpose:** Fix the business problem, intended use, population, target, and pre-registered success criteria *before* the analytical work is evaluated against them.  
**Companion documents:** LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · MAP-VR-LC-001 (Validation Report)

---

## At a glance

| Field | Value |
|---|---|
| **Business problem** | Collections prioritization — rank current loans by near-term default probability so effort goes where it helps most |
| **Intended use** | Operational prioritization only — which accounts get a call, letter, or restructure offer |
| **Explicitly not for** | Credit approval, pricing, provisioning, capital, or regulatory submission |
| **Unit of analysis** | One loan (one borrower-loan pair) |
| **Target** | Binary default — 1 if 16+ days past due, in grace period, or charged off |
| **Pre-registered success criteria** | 5 criteria across discrimination, fairness, and calibration (see §5) |
| **What this PDR does NOT contain** | Measured results, the verdict, or the fairness determination — see §9 |

> **Provenance note.** This PDR was reconstructed after the code was written, from the implementation and the pipeline outputs. It is a **frozen historical record of what the project intended**, not evidence of what it achieved. Where the PDR states a criterion, the criterion is what the project set out to meet; whether the model met it is recorded in the DCR (measured thresholds) and the Validation Report (verdict), not here. A reconstructed PDR is a legitimate artifact provided it is honest about its provenance; this one is.

---

## 1. Business problem

The bank's collections team has limited resources and a large book of current loans. It needs a prioritization model: **rank current loans by the probability they will become seriously delinquent in the near term**, so that collections effort goes to the accounts most likely to benefit. The model does not decide whether to lend — origination is historical and fixed in this dataset. It decides **where to look first among loans that already exist.**

**What goes wrong without it:** collections effort is allocated by days-past-due buckets alone, which react late. A model that ranks by predicted default probability lets collections intervene **before the first missed payment**, when recovery is cheaper.

---

## 2. Intended use and users

| Role | Description |
|---|---|
| **Primary user** | Collections operations — daily/weekly ranking of the current book |
| **Secondary user** | Model Risk Committee — receives the Validation Opinion and approves or rejects deployment |

**Intended use:** Operational prioritization. Rankings inform which accounts get a call, a letter, or a restructure offer.

**Explicitly not intended for:** credit approval, pricing, provisioning, capital, or any regulatory submission.

---

## 3. Population, unit, and target

**Population:** U.S. consumer loan applicants who were approved and funded by Lending Club in the observed window (2018 snapshot in this sample).

**Unit of analysis:** One loan (one borrower-loan pair). **Not the borrower** — a borrower with two loans contributes two rows.

**Target variable:** `default` (binary) — 1 if the loan is 16+ days past due, in grace period, or charged off; 0 if current or fully paid.

---

## 4. Target definition

| Status | Class |
|---|---|
| Charged Off, Late (31–120), In Grace Period, Late (16–30) | **1 — default** |
| Fully Paid, Current | **0 — performing** |
| Anything else | Excluded from training and evaluation |

**Threshold:** 16+ days, deliberately not 30+. **Rationale:** collections prioritization acts at the first sign of distress; 30+ is an IFRS 9 provisioning threshold, not an operational one.

**Class balance (this sample):** 1.78% default (178 of 10,000). Severe imbalance — the model must be evaluated on precision/recall and PR-AUC, **not accuracy**.

**Documented gap:** the 16+ day threshold is **not IFRS 9-compliant**. Recorded as Finding F-03.10 and treated as a **scope restriction, not a defect** — the model never claimed to be an IFRS 9 model.

> **Note on the class balance figure.** The 1.78% figure is the observed rate in the sampled dataset. It is stated here as a *dataset characteristic* relevant to target definition (the model must handle severe imbalance), not as a *success measurement*. The model's performance on the imbalanced target is recorded in the DCR and the Validation Report.

---

## 5. Pre-registered success criteria

These criteria were set as the standard against which the model would be judged. **The measured results against each criterion are recorded in the DCR (LC-DCR-001) and in the Validation Report (MAP-VR-LC-001), not in this PDR.** The separation is deliberate: a PDR that contains its own results cannot demonstrate that the criteria were pre-registered.

| Criterion | Target | Source | What "fail" would mean |
|---|---|---|---|
| AUC-ROC | ≥ 0.65 | DCR §1 | Model discrimination below the pre-registered floor; deployment not justified on performance grounds alone |
| KS statistic | ≥ 0.30 (strong discrimination) | DCR §1 | Tail separation insufficient for the stated prioritization use |
| Region-level DI | ≥ 0.80 | DCR §3 FD-01 | Fairness finding; deployment blocked pending remediation |
| Calibration (HL p) | > 0.05 for ≥ 3 of 4 models | DCR §2 | Predicted probabilities not reliable for threshold-based decisions |
| Precision at operating threshold | Reviewed against business cost, not a fixed number | DCR §1 | No fixed threshold; the analyst reviews precision against the cost of false positives and false negatives at the chosen operating point |

> **Why the criteria are stated without measured values.** A PDR's function is to fix the standard in advance. The standard is only meaningful if it is separable from the outcome. The measured values, the pass/fail determination per criterion, and the overall verdict are recorded in the DCR and the Validation Report, where they can be evaluated against this PDR without circularity.

---

## 6. Scope and exclusions

### In scope

- Binary default prediction on the historical Lending Club sample
- Four model classes compared under identical hyperparameters
- Fairness testing on region as a protected-attribute proxy
- Calibration and stability assessment on the held-out set
- SHAP-based direction check against credit-risk theory

### Excluded

| Exclusion | Reason |
|---|---|
| IFRS 9 provisioning | Target threshold misaligned (16 vs 30 days); model not designed for ECL |
| Canadian portfolios | U.S. data only; no validation on Canadian population |
| Direct race/gender fairness testing | Dataset has no protected attributes; region is a proxy |
| Macroeconomic stress testing | No macro factors in dataset |
| Time-based validation | No usable date column after leakage removal (`issue_month` removed per DS-001 §7) |
| Production scoring | The model is not approved; no scoring service exists |

---

## 7. Key assumptions and dependencies

### Assumptions

- Historical default patterns in the Lending Club sample are **representative of the operational book** (i.e., no distribution shift between training and deployment populations).
- **16+ days** is the operationally relevant distress threshold for collections.
- **Region** is an acceptable proxy for the protected attributes the model cannot observe.
- The four-model comparison is informative even though only one model (Random Forest) is selected.

### Dependencies

| Category | Dependency |
|---|---|
| **Data** | `loans_full_schema.csv` (hash `52872e7d4f8a388b`) — **no substitution permitted** (DS-001 §2.1) |
| **Upstream documents** | DS-001 (Data Specification), MD-001 (Methodology), AS-001 (Analytical Specification), DCR (Decision Criteria), FDL (Feature Decisions) |
| **Notebooks** | NB01 → NB02 → NB03 → NB04 → NB05 (in order; each notebook asserts its upstream contracts) |
| **External tools** | `fairlearn` (fairness metrics), `shap` (explainability), `scikit-learn`, `xgboost`, `lightgbm` |
| **Seed** | `RANDOM_STATE = 42` (AS-001 §8) — **changing it invalidates all reproducibility claims** |

---

## 8. Approval and ownership

| Role | Name | Responsibility |
|---|---|---|
| Model Developer | `[ANALYST: name]` | NB02 ownership; model training and calibration |
| Independent Validator | `[ANALYST: name]` | NB03 ownership; verdict authorship |
| Governance Team | `[ANALYST: name]` | NB04 ownership; compliance report |
| Model Risk Committee Chair | `[ANALYST: name]` | Final approval or rejection |
| Data Owner | `[ANALYST: name]` | Custodian of `loans_full_schema.csv` |

**Approval workflow:** Developer → Independent Validator (NB03 verdict) → Governance (NB04 compliance report) → Model Risk Committee (final decision recorded in the Approval Record).

> **Note on the placeholders.** The `[ANALYST: name]` fields are the analyst's contribution, not omissions. The PDR produces the structure and the roles; the analyst (or the project owner) assigns the names and records the approval.

---

## 9. What this PDR does not decide

This PDR fixes the problem, the intended use, the population, the target, and the pre-registered success criteria. It does **not** decide:

| Question | Where the answer lives |
|---|---|
| **Did the model meet the success criteria?** | LC-DCR-001 (measured thresholds, pass/fail per criterion) and MAP-VR-LC-001 (verdict) |
| **What is the model's risk rating and recommendation?** | MAP-VR-LC-001 (Validation Report, §1 and §7) |
| **Is the model fair across protected groups?** | MAP-VR-LC-001 (§4, fairness) and the fairness validation artifact (`outputs/reports/fairness_validation.csv`) |
| **Is the model IFRS 9-compliant?** | MAP-VR-LC-001 (§2, conceptual soundness) — and the answer is that it is not, by design, because the target threshold was set for collections, not provisioning |
| **What remediation is required, by whom, and by when?** | MAP-RP-001 (Model Risk Remediation Plan) |
| **Can the model be deployed?** | MAP-VR-LC-001 (§1, §6) — the Validation Report owns the deployability determination |

**The separation is deliberate.** A PDR that also contains the verdict is not a PDR; it is a PDR with a Validation Report stapled to it, and a reviewer cannot tell where the pre-registration ends and the post-hoc analysis begins. This PDR ends at the pre-registration. Everything downstream is in the documents listed above.

---

*End of document.*
