# Risk & Validation Plan — Lending Club Credit Default Model

**Document ID:** LC-RVP-001  
**Version / Status:** 2.2  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Risk & Validation Plan (RVP)  
**Frozen at:** NB03 v2.1 execution (run_id `20261021_145017`)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-DCR-001 (Decision Criteria Register) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Fix, in advance, what the validation will test, how success will be judged, and who decides |
| **Scope** | Independent validation of the Lending Club model (NB02) — fairness, performance, calibration, stability, governance |
| **Prescribed fairness test** | FD-05 4/5ths rule at region level (lowest-cardinality attribute) |
| **Prescribed discrimination floor** | AUC ≥ 0.65, KS ≥ 0.30 (DCR §1) |
| **Prescribed calibration floor** | HL p > 0.05 for ≥ 3 of 4 models (DCR §2) |
| **Prescribed stability floor** | PSI < 0.15 on a time-based split (DCR §4) |
| **Authority** | MRC Chair is the only role with approval authority; validator's verdict is authoritative until MRC override |
| **Where results live** | MAP-VR-LC-001 (Validation Report) — **not in this plan** |

> **Document purpose.** This plan defines how the Lending Club Credit Default Model will be validated: what is in scope, what risks are being tested, what evidence will be produced, who has authority to sign off, and what happens if the validation finds the model unfit.
>
> **On the relationship between this plan and the Validation Report.** This plan specifies *what the validation will test* and *what success will look like*. The Validation Report (MAP-VR-LC-001) reports *what the validation found*. The measured values (AUC 0.631, DI 0.5414, HL p-values, etc.) live in the Validation Report, not in this plan. A reader who wants to know whether the model passed a criterion reads the Validation Report; a reader who wants to know what the criterion was reads this plan.

---

## 1. Purpose and scope of the plan

This plan defines how the Lending Club Credit Default Model will be validated: what is in scope, what risks are being tested, what evidence will be produced, who has authority to sign off, and what happens if the validation finds the model unfit.

It is the **contract between the model developer and the Model Risk Committee** — it says, in advance, "here is what we will test, here is what success looks like, and here is who decides." The Validation Report (MAP-VR-LC-001) reports what the validation found; this plan specifies what it was supposed to find.

**In scope:** independent validation of the Lending Club model as trained (NB02), fairness testing, calibration assessment, stability assessment, governance review.

**Out of scope:** production monitoring (post-deployment), IFRS 9 provisioning use, Canadian portfolio applicability, direct protected-attribute fairness testing (no data), macroeconomic stress testing (no macro data).

---

## 2. Validation activities

The validation has five activities, in order of priority. The ordering is deliberate: fairness is first because a model that is accurate but unfair cannot be deployed; performance is second because a model that is fair but useless has no business case; calibration is third because miscalibrated probabilities mislead collections teams; stability is fourth because on a snapshot dataset it cannot be tested but must be documented; governance is fifth because it is the evidence trail, not the substantive question.

| # | Activity | What the validation will test | Prescribed method | Prescribed criterion |
|---|---|---|---|---|
| 1 | **Fairness** | Whether the model produces disparate impact across region groups | FD-05 4/5ths rule at region level (lowest-cardinality attribute) | DI ≥ 0.80 (DCR §3 FD-01) |
| 2 | **Performance** | Whether the model discriminates well enough for collections prioritization | AUC, KS, precision, recall, PR-AUC on held-out set | AUC ≥ 0.65, KS ≥ 0.30 (DCR §1) |
| 3 | **Calibration** | Whether predicted probabilities are reliable for decision-making | HL test with degeneracy check; slope/intercept | HL p > 0.05 for ≥ 3 of 4 models (DCR §2) |
| 4 | **Stability** | Whether the model will perform consistently on new data | PSI on a time-based split | PSI < 0.15 (DCR §4) |
| 5 | **Governance** | Whether the model is documented, reproducible, and compliant with regulatory expectations | OSFI E-23, SR 11-7, ECOA mapping | Compliance score ≥ 80 (DCR §9) |

> **Where the measured results are reported.** This table specifies *what will be tested* and *against which criterion*. The measured results against each criterion — and the pass/fail determination — are recorded in **MAP-VR-LC-001** (Validation Report) and in **LC-DCR-001** (Decision Criteria Register). They are not in this plan, because a plan that contains its own results cannot demonstrate that the criteria were pre-registered.

---

## 3. Validation scope

### 3.1 What is validated

| Dimension | Scope | Notebook |
|---|---|---|
| Data quality | Missingness, leakage, duplicates, quality score | NB01 |
| Model construction | Four model classes, hyperparameters, split method | NB02 |
| Performance | AUC, KS, precision, recall, PR-AUC on held-out set | NB03 |
| Fairness | FD-05 4/5ths rule at region level | NB03 |
| Calibration | HL test + slope/intercept + degeneracy check | NB03 |
| Stability | PSI attempt (expected `not_assessed` — see §3.2) | NB03 |
| Explainability | SHAP top-10 + monotonicity check | NB03 |
| Cross-population | German Credit direction-only comparison | NB05 |

### 3.2 What is not validated

| Item | Reason | Documented as |
|---|---|---|
| Production scoring | No deployed service exists | Out of scope |
| IFRS 9 provisioning | Target is 16+ days, not 30+ days | F-03.10 (scope restriction) |
| Canadian applicability | U.S. data only | F-03.09 (limitation) |
| Direct race/gender fairness | No protected attributes in dataset | LC-DS-001 §8 L3 |
| Macroeconomic stress | No macro factors in dataset | F-03.08 (limitation) |
| Time-based stability | No usable date column after leakage removal | F-03.02 (split exception) |

**The exclusions are as important as the inclusions.** A validator who reads only §3.1 would think the model is comprehensively validated. The exclusions in §3.2 make the scope honest.

---

## 4. Risk assessment

### 4.1 Inherent risks — what the validation will look for

These are the risks the model may carry, which the validation is designed to detect. The validation **tests** for each; it does **not** assume the outcome. The measured findings (which risks materialised, and at what severity) are recorded in **MAP-VR-LC-001 §4** and in **LC-DCR-001**.

| Risk ID | Inherent risk | Why it matters | How the validation detects it |
|---|---|---|---|
| R-01 | **Leakage contamination** — post-origination features inflate performance | Invalidates the model if present | Pre-registered leakage list; reconciliation assertion; AUC drop measurement |
| R-02 | **Severe disparate impact** — region-level DI below 0.60 | Blocks deployment on fairness grounds | FD-05 4/5ths rule at region level |
| R-03 | **Poor calibration** — HL p ≤ 0.05 for one or more models | Misleading probabilities | HL test + degeneracy check; slope/intercept |
| R-04 | **Methodological deviation** — random split, not walk-forward | PSI not assessable; results optimistically biased | Disclosed in `split_disclosure`; finding if not formally amended |
| R-05 | **Weak discrimination** — KS below 0.30 | Limited ranking sharpness for collections | KS statistic from ROC curve |
| R-06 | **Counterintuitive SHAP** — features opposite to credit-risk theory | Spurious pattern risk | SHAP top-10 monotonicity check |
| R-07 | **No macro factors** | Procyclical blindness | Documented limitation; not testable on this data |
| R-08 | **U.S. data only** | Jurisdictional risk | Documented limitation; not testable on this data |
| R-09 | **IFRS 9 misalignment** — 16+ vs 30+ days | Scope restriction | Documented; out of scope for this validation |
| R-10 | **Low precision** at operating threshold | High false-positive rate | Precision/recall at the operating point |

> **On "present" vs "tested."** This register lists the risks the validation will test. It does not state whether each risk materialised — that is the Validation Report's job. A validation plan that says "leakage is present" before the validation has run is not a plan; it is a results summary. The "present" language was in the v2.1 register; it has been removed.

### 4.2 Validation risks — what could go wrong with the validation itself

Separate from the model's inherent risks, the validation process itself carries risks. These are risks *of the validation*, not *of the model*.

| Risk ID | Validation risk | Impact on the validation | Mitigation |
|---|---|---|---|
| VR-1 | **Fairness test applied at the wrong cardinality** | DI is meaningless if computed per-state one-hot | FD-05 cardinality constraint enforced; region-level test only |
| VR-2 | **HL degeneracy undetected** | Degenerate p-values treated as calibration results | Degeneracy check (p exactly 0.0 or 1.0) enforced |
| VR-3 | **PSI reported as STABLE on a random split** | Misleading stability evidence | PSI reported as `not_assessed` when split is not time-based |
| VR-4 | **Verdict authored outside the ownership chain** | Unauditable verdict | `authored_by` assertion in NB04/NB05 |
| VR-5 | **Reconciliation drift between NB02 and NB03** | NB03's recomputed metrics diverge from NB02's reported metrics | 1e-6 reconciliation assertion; WARNING if drift exceeds threshold |

> **Why this register is separate from §4.1.** The model's inherent risks are what the validation is *testing*; the validation's own risks are what could make the validation *unreliable*. A single register that mixes them cannot distinguish "the model might be unfair" from "the fairness test might be wrong." The separation is deliberate.

### 4.3 Residual risk acceptance conditions

The residual risk profile **after remediation** depends on the remediation being executed as specified in MAP-RP-001. This plan cannot forecast the residual risk state — it can only state the **conditions under which the residual risk would be acceptable**.

| Risk ID | Residual risk acceptance condition | Tracking |
|---|---|---|
| R-01 | **Leakage removed** — all nine pre-registered leakage features removed; AUC drop disclosed | LC-R-01 in MAP-RP-001 (resolved) |
| R-02 | **Fairness remediation** — region-level DI ≥ 0.80 on a properly held-out population, with the fairness test applied at FD-05-compliant cardinality | LC-R-01 in MAP-RP-001 |
| R-03 | **Calibration remediation** — poor models recalibrated (Platt/isotonic) or replaced; HL degeneracy resolved | LC-R-03 in MAP-RP-001 |
| R-04 | **Split amendment or walk-forward** — either MD-001 §2 amended to permit random splits (with DCR approval), or walk-forward implemented on a dataset with a usable date column | LC-R-02 in MAP-RP-001 |
| R-05 | **KS improvement** — either KS ≥ 0.30 achieved, or the business case for a weaker KS accepted by the MRC | LC-R-04 in MAP-RP-001 |
| R-06 | **SHAP investigation** — the four counterintuitive features investigated and either re-engineered or documented as acceptable | LC-R-04 in MAP-RP-001 |
| R-07 | **Macro addition** — macro factors added, or the model's procyclical blindness accepted as a scope limitation | LC-R-05 in MAP-RP-001 |
| R-08 | **Canadian validation** — model validated on Canadian data, or Canadian deployment restricted | LC-R-05 in MAP-RP-001 |
| R-09 | **Scope restriction or recalibration** — the model restricted to non-provisioning use, or recalibrated to 30+ days | LC-R-05 in MAP-RP-001 |
| R-10 | **Threshold tuning** — threshold tuned against business cost, or precision accepted with a documented rationale | Part of LC-R-03 |

> **On the relationship between this plan and the Remediation Plan.** This section states the *conditions* under which residual risk would be acceptable. The MAP-RP-001 tracks *whether* the conditions are met — owners, timelines, evidence. A reader who wants to know "is the fairness remediation done?" reads MAP-RP-001; a reader who wants to know "what would 'done' look like?" reads this section.

---

## 5. Validation methodology

### 5.1 What the validation will test — methods and criteria

| Activity | Notebook | Evidence artifact (to be produced) | Prescribed acceptance criterion |
|---|---|---|---|
| Data quality assessment | NB01 | `data_quality_summary.json`, `feature_inventory.csv` | Data quality score ≥ 60 |
| Leakage detection | NB01 | `leakage_findings.csv` | All leakage features identified; AUC drop measured |
| Model training | NB02 | Four model pickles, `predictions.pkl` | All four models trained with MD-001 §3.4 hyperparameters |
| Performance evaluation | NB03 | `performance_metrics.csv` | AUC ≥ 0.65, KS ≥ 0.30 (DCR §1) |
| Fairness testing | NB03 | `fairness_validation.csv` | DI ≥ 0.80 (DCR §3 FD-01) |
| Calibration testing | NB03 | `calibration_metrics.csv` | HL p > 0.05 for ≥ 3 models (DCR §2) |
| Stability testing | NB03 | PSI result | PSI < 0.15 (DCR §4) — or `not_assessed` with documented reason |
| Explainability | NB03 | `monotonicity_check.csv` | All top-10 SHAP directions consistent with credit-risk theory |
| Governance review | NB04 | `compliance_scorecard_v2.json` | Compliance score ≥ 80 (DCR §9) |
| Cross-population | NB05 | `german_credit_cross_validation.json` | Direction confirmed (SMALL_DIRECTIONAL or LARGE_DIRECTIONAL) |

> **Where the measured results live.** The rightmost column states the *acceptance criterion*. The *measured result* against each criterion — and the pass/fail determination — are recorded in **MAP-VR-LC-001 §4** and **LC-DCR-001 §3**. A reader who wants to know "did the model pass?" reads the Validation Report; a reader who wants to know "what was the criterion?" reads this table.

### 5.2 The verdict-ownership chain — a governance control

The validation methodology enforces a strict chain of authorship. **This is a governance control, not a methodology detail** — it is what makes the verdict auditable, and it maps directly to the decision authority matrix in §6.2.

NB02 (developer) → writes performance_summary_v2.json WITHOUT verdict
↓
NB03 (validator) → asserts no verdict, writes its own WITH verdict
↓
NB04 (governance) → asserts authored_by="NB03", inherits verdict
↓
NB05 (cross-validation) → asserts authored_by="NB03", reads verdict context


**No notebook can short-circuit the chain.** The chain enforces four properties:

| Property | What it guarantees | Where it maps to §6.2 |
|---|---|---|
| NB02 cannot author a verdict | Developer cannot self-approve | Developer row: "cannot self-approve" |
| NB03 must author the verdict | Only the validator writes the verdict | Validator row: "Authors the verdict" |
| NB04 cannot override | Governance documents, does not decide | Governance row: "does not override validator" |
| NB05 cannot short-circuit | Cross-validation reads context, does not re-derive | — (NB05 is terminal) |

**A note on what the chain does not do.** The chain does not prevent the MRC Chair from overriding the validator (that is §6.2's "Override the verdict" row). The chain prevents the *notebooks* from short-circuiting the verdict; the *governance* override is a separate, documented decision. The distinction is important: a chain that also bound the MRC would be a chain that bound the body the chain exists to serve.

### 5.3 Where the run results are reported

The measured results against each criterion — the "did the model pass?" question — are recorded in:

- **MAP-VR-LC-001 (Validation Report)** — §1 (executive summary), §4 (outcome analysis), §7 (findings summary)
- **LC-DCR-001 (Decision Criteria Register)** — §3 (criteria and pass/fail), §4 (decision record)
- **`outputs/reports/performance_summary_v2.json`** — the machine-readable verdict artifact, with `authored_by: "NB03"`

**The measured values are not in this plan**, because a plan that contains its own results cannot demonstrate that the criteria were pre-registered. The separation is deliberate.

---

## 6. Validation governance

### 6.1 Roles and responsibilities

| Role | Name | Responsibility | Authority |
|---|---|---|---|
| Model Developer | `[ANALYST: name]` | NB02 ownership; model training; remediation | Implements changes; cannot self-approve |
| Independent Validator | `[ANALYST: name]` | NB03 ownership; verdict authorship; Validation Opinion | Authors the verdict; cannot deploy |
| Governance Team | `[ANALYST: name]` | NB04 ownership; compliance report | Documents compliance; does not override validator |
| Model Risk Committee Chair | `[ANALYST: name]` | Final approval or rejection | Can approve, reject, or override (with documented rationale) |
| Data Owner | `[ANALYST: name]` | Custodian of `loans_full_schema.csv` | Controls data access and versioning |

**Separation of duties.**

- The developer cannot validate their own model (NB02 vs NB03 separation).
- The validator cannot approve their own verdict (NB03 vs MRC separation).
- The governance team documents but does not decide (NB04 is evidence, not verdict).
- The MRC chair is the only role with approval authority.

### 6.2 Decision authority matrix

| Decision | Developer | Validator | Governance | MRC Chair |
|---|---|---|---|---|
| Retrain the model | Yes | No | No | Notify |
| Write the Validation Opinion | No | Yes | No | Review |
| Override the verdict | No | No | No | Yes (with rationale) |
| Approve deployment | No | No | No | Yes |
| Condition the approval | Propose | Yes | Document | Approve |
| Deploy | Implement | No | No | Authorize |

The matrix makes the chain auditable. An interviewer who asks "who can override the validator?" gets a clear answer: **only the MRC Chair, with documented rationale.** The validator's verdict is authoritative until the MRC overrides it — and the override is itself a recorded decision.

---

## 7. Validation schedule and milestones

### 7.1 Phase plan

| Phase | Activity | Pipeline status | Validation status |
|---|---|---|---|
| Phase 1 — Development | NB01 + NB02 (data, features, training) | **Executed** | — |
| Phase 2 — Independent validation | NB03 (performance, fairness, calibration, SHAP) | **Executed** | **Pending MRC review** |
| Phase 3 — Governance | NB04 (compliance report) | **Executed** | **Pending MRC review** |
| Phase 4 — Cross-validation | NB05 (German Credit) | **Executed** | **Pending MRC review** |
| Phase 5 — Validation Opinion | Analyst-authored verdict document | **Authored** | **Pending signature** |
| Phase 6 — MRC review | MRC decision on the verdict | — | **Pending** |
| Phase 7 — Remediation | LC-R-01 through LC-R-10 (MAP-RP-001) | — | **Not started** |
| Phase 8 — Re-validation | NB03 re-run with new artifacts | — | **Not started** |

> **On the two status columns.** "Pipeline status" answers "did the notebook run?" "Validation status" answers "has the validation activity been signed off?" The two are different: a notebook can have executed (pipeline status: Executed) without the validation being complete (validation status: Pending MRC review). Conflating them — as the v2.1 document did — makes the plan look as if the validation is complete when only the pipeline has run.

### 7.2 Key milestones

| Milestone | Target date | Owner | Status |
|---|---|---|---|
| Validation Opinion signed | `[ANALYST: date]` | Independent Validator | Pending |
| MRC decision recorded | `[ANALYST: date]` | MRC Chair | Pending |
| Fairness remediation complete (LC-R-01) | 3 months | Developer | Not started |
| Calibration remediation complete (LC-R-03) | 1 month | Developer | Not started |
| Split method decision (LC-R-02) | Next version | Developer | Not started |
| Re-validation | Month 3 | Validator | Not started |

### 7.3 Escalation path

| Trigger | Escalation |
|---|---|
| CRITICAL finding in NB03 | Immediate escalation to MRC; no deployment until resolved |
| HIGH finding in NB03 | Escalate to MRC with timeline; deployment conditions attached |
| Remediation not on schedule | Escalate to MRC at 50% of timeline |
| New CRITICAL finding post-validation | Re-validation required before any deployment |
| MRC overrides validator verdict | Documented rationale required; re-validation on the affected dimension |

**The escalation path is the operational control.** A CRITICAL finding without an escalation path is a document; a CRITICAL finding with a defined escalation path is a process.

---

## 8. Validation-process limitations

### 8.1 On the relationship to other limitations

The model's data limitations (L1–L8) are in **LC-DS-001 §8**. The methodology's limitations (M1–M6) are in **LC-MD-001 §8**. The validation's findings against each criterion are in **MAP-VR-LC-001 §4** and **LC-DCR-001 §3**. **This section records only the limitations of the validation *process* — things a reader of the Validation Report should know about the validation itself, not about the model or the data.**

### 8.2 Validation-process limitations

| ID | Limitation | Impact on the validation | Mitigation |
|---|---|---|---|
| VL-1 | **Fairness test uses a region proxy, not a direct protected attribute** | Fairness findings are indicative, not definitive | Documented in the Validation Report; HMDA-equivalent data acquisition is a longer-horizon item |
| VL-2 | **PSI is `not_assessed` on a random split** | Stability not tested; 0 stability points reflect absence of evidence, not evidence of stability | Documented in the Validation Report; walk-forward implementation is LC-R-02 |
| VL-3 | **SHAP is on a 100-row sample, not the full validation set** | Feature importance is indicative, not exhaustive | Documented; full-population SHAP is a future-version candidate |
| VL-4 | **HL test produces degenerate p-values for 3 of 4 models** | Calibration uncertain for those models; only LightGBM has a valid result | Degeneracy check flags the issue; HL implementation verification is LC-R-03 |
| VL-5 | **Single-threshold fairness** | Approval rate measured at 0.5 only; the DI may differ at a different operating point | Documented; threshold sensitivity is a future-version candidate |
| VL-6 | **No V1.0 baseline** | The compliance scorecard cannot report a change-from-baseline | Documented in NB04; a V1.0 baseline can be authored in a future revision |
| VL-7 | **The validation is performed by the same analyst who built the pipeline** | The independence claim rests on the NB02/NB03 separation and the `authored_by` assertion, not on organizational separation | The verdict-ownership chain (§5.2) is the mitigation; organizational independence is a governance decision outside this plan |

> **On VL-7.** The Lending Club project is a portfolio artifact, and the same analyst built the pipeline and wrote the validation. The independence claim in this plan rests on the **technical** separation (NB02 vs NB03; `authored_by` assertion) and the **decision authority matrix** (§6.2), not on organizational independence. In a production deployment, the validator would be a separate person in a separate function. The plan should state this clearly — a reader who assumes organizational independence has misunderstood the artifact.

### 8.3 Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **MRC decision** | Pending. The Validation Opinion recommends REJECT; the MRC Chair must record the decision and rationale. | Phase 6 in §7.1 |
| 2 | **Split method amendment** | MD-001 §2 must be amended to permit random splits, or the pipeline must acquire a date column. Current status: disclosed exception, amendment pending. | LC-R-02 in MAP-RP-001 |
| 3 | **Fairness remediation plan** | LC-R-01 is drafted but not yet executed. The remediation will require re-validation of the fairness test. | LC-R-01 in MAP-RP-001 |
| 4 | **V1.0 baseline** | No V1.0 baseline exists. The NB04 compliance scorecard suppresses the V1.0 comparison. If a V1.0 baseline is authored, the scorecard will include the delta. | Not tracked — documentation-only gap |

---

*End of document.*
