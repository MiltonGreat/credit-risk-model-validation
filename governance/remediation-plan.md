# Lending Club Credit Default Model — Remediation Plan

**Document ID:** LC-RMP-001  
**Version / Status:** 2.2 / Draft pending MRC approval  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Remediation Plan (RMP)  
**Source findings:** LC-CFR-001 §3 (Consolidated Findings Register) — 8 OPEN (rem) findings  
**Frozen at:** NB03 v2.1 execution (run_id `20261021_145017`)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · LC-CFR-001 (Consolidated Findings Register) · MAP-VR-LC-001 (Validation Report)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Move the Lending Club model from REJECT to APPROVE by closing the remediable findings |
| **Findings in scope** | 8 OPEN (rem) findings, grouped into 8 actions |
| **Findings out of scope** | 4 OPEN (docs) + 9 OPEN (scope) — see §1 |
| **Phases** | 4 (Fast, Blocking, Method, Long) |
| **Critical path** | R-LC-01 (fairness, 3 months) — see §4 for why calibration is not on the critical path |
| **Re-validation trigger** | When R-LC-01, R-LC-03, and either R-LC-02 or its amendment are complete |
| **Re-validation target** | Month 3 |
| **Blocking finding** | F-03.03 (severe disparate impact, DI 0.5414) |

> **Document purpose.** This plan specifies how the Lending Club model will move from REJECT to APPROVE. It takes the **8 remediable findings** from the consolidated findings register, groups them into 8 remediation actions, assigns each action an owner, timeline, and success criterion, and specifies the re-validation trigger.
>
> **On the scope.** The register contains 21 OPEN findings, but only 8 are remediable within the plan's horizon. The 4 OPEN (docs) findings are closed by documenting the decision; the 9 OPEN (scope) findings are permanently OPEN until the scope changes. **This plan covers only the 8 remediable findings.** A reader who wants the full picture reads LC-CFR-001 §8.
>
> **On the relationship between the plan and the Validation Report.** The plan specifies *how* the findings will be closed; the Validation Report (MAP-VR-LC-001) reports *what the validation found* and *what the verdict is*. The plan does not contain the verdict; it contains the path to a new verdict.

---

## 1. Purpose and scope of the plan

This plan specifies how the Lending Club model will move from REJECT to APPROVE. It takes the **8 remediable findings** from the consolidated findings register (LC-CFR-001 §3, findings with status `OPEN (rem)`), groups them into **8 remediation actions**, assigns each action an owner, timeline, and success criterion, and specifies the re-validation trigger.

The plan is the bridge between the verdict and the re-validation. Without it, the REJECT verdict is a statement with no path forward. With it, the REJECT is a starting point, not a dead end.

### What the plan covers

The 8 remediable findings and the action that addresses each:

| Finding | Action | Phase |
|---|---|---|
| F-02.02 (walk-forward split) | R-LC-02 | Phase 3 — Method |
| F-03.02 (walk-forward split, re-finding) | R-LC-02 | Phase 3 — Method |
| F-02.04 (poor calibration, XGBoost) | R-LC-03 | Phase 1 — Fast |
| F-03.04 (poor calibration, XGBoost, re-finding) | R-LC-03 | Phase 1 — Fast |
| F-02.05 (suspicious HL degeneracy) | R-LC-03 | Phase 1 — Fast |
| F-03.05 (suspicious HL degeneracy, re-finding) | R-LC-03 | Phase 1 — Fast |
| F-02.06 (low precision) | R-LC-03 | Phase 1 — Fast |
| F-03.11 (low precision, re-finding) | R-LC-03 | Phase 1 — Fast |
| F-03.03 (severe disparate impact) | R-LC-01 | Phase 2 — Blocking |
| F-03.06 (weak discrimination) | R-LC-04 | Phase 3 — Method |
| F-03.07 (counterintuitive SHAP) | R-LC-05 | Phase 1 — Fast |
| F-03.10 (IFRS 9 misalignment) | R-LC-08 | Phase 2 — Blocking |

(Some actions address multiple findings; the mapping is one-to-many because the same remediation can close several findings.)

### What the plan does not cover

- **4 OPEN (docs) findings** (F-01.07, F-01.08, F-02.07): closed by documenting the decision in LC-FDL-001 or the Validation Report, not by the plan.
- **9 OPEN (scope) findings** (F-01.04–F-01.06, F-02.08, F-03.08–F-03.10): permanently OPEN until the scope changes (Canadian data, macro factors, IFRS 9 recalibration as a *scope* change rather than a remediation). These are monitored but not remediated by this plan.
- **The verdict itself**: the plan does not change the verdict. It closes findings; the verdict changes only when a re-validation produces a different score.

**On the "21 → 8" framing.** The register has 21 OPEN findings, but only 8 are remediable within the plan. The plan's scope is the 8; the remaining 13 are documented in LC-CFR-001 §8 with their own tracking categories. A reader who wants the full picture reads the register; a reader who wants to know what this plan fixes reads this document.

---

## 2. Remediation philosophy and principles

Four principles govern the plan:

1. **Fix the blocking issue first.** Fairness (F-03.03) is the primary driver of the REJECT. It is the first blocking action on the timeline, and the plan protects its critical path.

2. **Sequence by dependency, not by speed.** Where one action depends on another (e.g., R-LC-04 depends on R-LC-02 because KS is measured on the held-out set, which depends on the split method), the plan sequences them. Where actions are independent, the plan runs them in parallel. **The plan does not assume dependencies that do not exist** — see §4 for the specific case of R-LC-03 and R-LC-01.

3. **Distinguish fix from document.** Some findings need a code change (fairness, calibration); others need a documentation update (split disclosure, SHAP investigation). The plan is explicit about which is which. An action that is "document the decision" is a different kind of work than an action that is "recalibrate the model."

4. **Every action has a re-validation trigger.** No action is complete until the re-run confirms the finding is addressed. "Fixed" means "re-validated," not "committed."

The philosophy is the same as the RVP's: the plan is a contract, not a wish list. Every action is specific, owned, and time-bound.

---

## 3. Remediation actions (the core table)

8 actions, grouped into 4 phases:

| Phase | Action ID | Action | Findings addressed | Owner | Timeline | Success criterion |
|---|---|---|---|---|---|---|
| **Phase 1 — Fast** | R-LC-05 | SHAP investigation: examine the 4 counterintuitive features; re-engineer or document | F-03.07 | Developer | 2 weeks | All top-10 features directionally consistent OR documented with rationale |
| **Phase 1 — Fast** | R-LC-03 | Calibration: Platt scaling or isotonic regression for XGBoost; verify HL implementation | F-02.04, F-03.04, F-02.05, F-03.05, F-02.06, F-03.11 | Developer | 1 month | All 4 models HL p > 0.05 AND non-degenerate; precision reviewed against business cost |
| **Phase 2 — Blocking** | R-LC-01 | Fairness: remove state features or apply reweighting; re-test region DI | F-03.03 | Developer + Validator | 3 months | Region-level DI ≥ 0.80 |
| **Phase 2 — Blocking** | R-LC-08 | IFRS 9: recalibrate for 30+ day threshold OR formally restrict to collections scope | F-03.10 | Developer | 3 months | Either 30+ day target OR documented scope restriction |
| **Phase 3 — Method** | R-LC-02 | Split method: implement time-based split OR formally amend MD-001 §2 | F-02.02, F-03.02 | Developer + Methodology Owner | Next version | Either walk-forward OR documented amendment |
| **Phase 3 — Method** | R-LC-04 | KS improvement: feature engineering to raise KS above 0.30 | F-03.06 | Developer | 2 months | KS ≥ 0.30 on held-out set |
| **Phase 4 — Long** | R-LC-06 | Macro factors: add GDP/unemployment/HPI for production | F-01.04, F-03.08 | Developer | 6 months | Macro features included; re-validated |
| **Phase 4 — Long** | R-LC-07 | Canadian validation: validate on Canadian data before Canadian deployment | F-01.05, F-03.09 | Model Risk | 6 months | Canadian validation report |

**On the phases.**

- **Phase 1 (Fast):** 2 weeks to 1 month. These do not unblock deployment, but they clear the quick findings so Phase 2 can focus on the blocking issue. R-LC-05 and R-LC-03 can start immediately.
- **Phase 2 (Blocking):** 3 months. These are the conditions for re-validation. When Phase 2 completes, the model can be re-validated.
- **Phase 3 (Method):** 2 months to next version. These improve the methodology but are not blocking re-validation.
- **Phase 4 (Long):** 6 months. These are scope expansions, not fixes within the current horizon.

**The critical path: R-LC-01 (fairness, 3 months) → re-validation.** The critical path is **3 months, not 4**. Calibration (R-LC-03) is **not** on the critical path — see §4 for why.

---

## 4. Sequencing and dependencies

### 4.1 — The dependency graph

R-LC-05 (SHAP, 2 weeks) ────────────────────┐
│
R-LC-03 (Calibration, 1 month) ────┐ │
│ ├──→ RE-VALIDATION (month 3)
R-LC-01 (Fairness, 3 months) ──────┴────────┤
│
R-LC-08 (IFRS 9 scope, 3 months) ───────────┘

R-LC-02 (Split method, next version) ──→ R-LC-04 (KS, 2 months) ──→ post-revalidation

R-LC-06 (Macro, 6 months) ───→ production, not re-validation
R-LC-07 (Canadian, 6 months) ───→ production, not re-validation


### 4.2 — The three parallel tracks

- **Fast track:** R-LC-05 and R-LC-03. Can start immediately.
- **Blocking track:** R-LC-01 and R-LC-08. Can start immediately (see §4.3 for why R-LC-01 does not depend on R-LC-03).
- **Long track:** R-LC-02, R-LC-04, R-LC-06, R-LC-07. These can run in parallel with the blocking track.

### 4.3 — On the R-LC-03 → R-LC-01 dependency

The v2.1 plan asserted that R-LC-01 (fairness) must follow R-LC-03 (calibration) because "the fairness test uses the calibrated model's predictions." **On review, this dependency does not hold as stated.**

The fairness test in NB03 computes DI from **approval rates at threshold 0.5**. The approval rate depends on the *ranking* of predictions (which loans are above the threshold), not on the *calibration* of the probabilities (whether a score of 0.3 actually means 30%). Recalibrating XGBoost with Platt scaling or isotonic regression **does not change the ranking** of its predictions — it changes the probability values, but the loans above the threshold stay the same. Therefore, **recalibration does not change the DI** of the model being tested.

There are two cases:

- **Case A (current run):** the fairness test uses Random Forest's predictions (best-AUC model, per NB03 §4). R-LC-03 recalibrates XGBoost. The recalibration does not affect Random Forest's predictions, so it cannot affect the DI. **No dependency.**
- **Case B (if R-LC-01 also swaps the model):** if R-LC-01 replaces Random Forest with a recalibrated XGBoost, then the DI is computed on the new model's predictions, and the dependency is real (but not because of calibration — because of the model swap).

**This plan assumes Case A.** R-LC-01 and R-LC-03 can run in parallel, and the critical path is 3 months (R-LC-01), not 4 months (R-LC-03 → R-LC-01).

If the analyst decides that R-LC-01 will swap the model (Case B), the plan must be revised: R-LC-01's success criterion must specify which model's DI is being measured, and the dependency R-LC-03 → R-LC-01 must be reinstated with the model-swap rationale. **This is a decision for the analyst, not the plan.** The plan states the current assumption (Case A) and flags the alternative.

### 4.4 — On the R-LC-02 → R-LC-04 dependency

KS is measured on the held-out set, and the held-out set's composition depends on the split method. Changing the split method changes the held-out set, which changes the KS. Doing R-LC-04 before R-LC-02 would mean redoing R-LC-04 after R-LC-02. **This dependency holds.** R-LC-04 is sequenced after R-LC-02 in Phase 3.

### 4.5 — Why R-LC-08 is on the blocking track but not on the critical path

R-LC-08 (IFRS 9 scope) is on the blocking track because it must be decided before re-validation (the verdict considers whether the model is fit for the stated intended use, and the IFRS 9 scope decision affects the intended use). But R-LC-08's completion is not required for the fairness re-validation — the re-validation can proceed with the IFRS 9 scope decision pending, as long as the decision is made before the verdict is re-authored.

**R-LC-08 is a blocking *decision*, not a blocking *action*.** It must be resolved by month 3 but does not gate the fairness re-validation. The distinction is in §4.1's graph (R-LC-08 is parallel to R-LC-01, not downstream of it).

---

## 5. Success criteria and re-validation triggers

### 5.1 — Re-validation trigger

**Full re-validation (NB03 re-run) is triggered when R-LC-01 and R-LC-03 are complete.** Target: month 3.

**R-LC-08 is not a re-validation trigger** — it is a decision that must be made before the re-validation is *authored*, not before it is *run*. The re-validation can run with R-LC-08 pending; the verdict cannot be re-authored until R-LC-08 is decided.

### 5.2 — Re-validation success criteria

The re-validation's success criteria are the DCR's criteria (LC-DCR-001 §3). The three criteria that drive the verdict change are:

| Criterion | Target | Where the result is recorded |
|---|---|---|
| Region-level DI | ≥ 0.80 (OK band, DCR §3.3 FD-01) | NB03 `fairness_validation.csv`; DCR §4.3 |
| HL p > 0.05 for all models | 4 of 4 valid | NB03 `calibration_metrics.csv`; DCR §4.2 |
| Split method disclosed/amended | Time-based OR amendment documented | NB03 `performance_summary_v2.json`; DCR §4.1 |

**On the "current" values.** This table does not include a "current" column. The current measured values are in **LC-DCR-001 §4** (results against criteria). The plan cites the DCR; it does not restate the values. If the DCR's measured values are updated after partial re-validation, the plan's success criteria remain unchanged (they are the targets, not the current state).

### 5.3 — Partial re-validation

If only R-LC-01 completes (fairness fixed, calibration not yet), a **partial re-validation** can be run to check whether the fairness fix works. This is a diagnostic, not a verdict change: the verdict is not re-authored, and the partial re-validation's output is not a new verdict. Its purpose is to catch problems early — if the fairness fix does not achieve DI ≥ 0.80 on the partial re-validation, the plan's critical path is at risk.

### 5.4 — Post-re-validation

If the re-validation produces a verdict of MODERATE or LOW, the model can be deployed with conditions (per LC-RVP-001 §6.2's decision authority). If it produces HIGH or CRITICAL, the plan continues to Phase 3 — the model is not deployable, and the remaining findings must be closed before another re-validation.

---

## 6. Resource requirements and ownership

| Action | Owner | Secondary | Effort estimate |
|---|---|---|---|
| R-LC-05 | Developer | — | 2 weeks FTE |
| R-LC-03 | Developer | Validator (review) | 1 month FTE |
| R-LC-01 | Developer | Validator (re-test) | 3 months (part-time) |
| R-LC-08 | Developer | Methodology Owner (approval) | 3 months (part-time) |
| R-LC-02 | Developer | Methodology Owner (amendment) | Next version |
| R-LC-04 | Developer | — | 2 months FTE |
| R-LC-06 | Developer | Data team (macro acquisition) | 6 months |
| R-LC-07 | Model Risk | External data provider | 6 months |

**Total effort estimate:** 1 FTE for 6 months (developer), plus part-time validator and Model Risk support.

**The resource case is the plan's soft underbelly.** The plan assumes dedicated developer time, which may not be available. If resources are constrained, the critical path (**R-LC-01**, 3 months) must be protected — the other actions can slip without delaying re-validation.

---

## 7. Monitoring and reporting during remediation

### 7.1 — Monthly tracking against the blocking actions

| Metric | Baseline (per DCR §4) | Month 1 target | Month 2 target | Month 3 target |
|---|---|---|---|---|
| Region-level DI | 0.5414 (DCR §4.3) | 0.60 | 0.70 | 0.80 |
| HL p > 0.05 models | 1 of 4 (DCR §4.2) | 2 of 4 | 3 of 4 | 4 of 4 |
| SHAP counterintuitive features | 4 (DCR §4.5) | 2 | 0 | 0 |

**On the baseline column.** The baseline values are the frozen pre-remediation values, cited from LC-DCR-001 §4. **If the DCR is updated after partial re-validation, the plan's baseline is not updated** — the baseline is the pre-remediation state, not the current state. This is deliberate: the baseline is the reference point against which the remediation is measured, and changing it would obscure the progress.

**On KS.** KS (F-03.06, 0.2671) is **not** in the monthly tracking table. It is scheduled in Phase 3 (R-LC-04, 2 months), not on the blocking track, and it is not a re-validation criterion for the Phase 2 re-validation. A reader who wants to know KS's status reads the Phase 3 tracking (not shown in this table, because Phase 3's monthly tracking would begin after Phase 2 completes). **The v2.1 table included KS with a month-3 target of 0.30, which implied KS was on the critical path. It is not. The v2.2 table omits KS from the blocking tracking to make the critical path honest.**

### 7.2 — Escalation triggers

| Trigger | Escalation |
|---|---|
| Region-level DI not ≥ 0.60 by month 1 | Escalate to MRC |
| Region-level DI not ≥ 0.70 by month 2 | Escalate to MRC; critical path at risk |
| HL p > 0.05 not achieved for all models by month 2 | Escalate to MRC |
| Critical path (R-LC-01) slips by 2 weeks | Escalate to MRC |

**The escalation threshold is "critical path slip," not "any slip."** The plan is allowed to slip on non-critical actions; the critical path is not.

---

## 8. Open items and dependencies

### 8.1 — Open items for the plan

| # | Item | Owner | Due | Routing |
|---|---|---|---|---|
| 1 | **MRC approval of this plan** | MRC Chair | `[ANALYST: date]` | MRC meeting |
| 2 | **Resource commitment** — dedicated developer time | Developer manager | `[ANALYST: date]` | Resource allocation |
| 3 | **R-LC-02 decision** — walk-forward or amend MD-001 §2? | Methodology Owner | Before R-LC-04 starts (Phase 3) | Methodology decision |
| 4 | **R-LC-08 scope decision** — 30+ day recalibration or collections-only scope? | Business owner | Before re-validation is authored (month 3) | Business decision |

**On the open items.** Each has an owner and a due date relative to a milestone. The v2.1 plan listed these open items without owners or dates; the v2.2 plan makes them specific, consistent with §2's principle that "the plan is a contract, not a wish list."

### 8.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | LC-CFR-001 §3 (source findings), LC-DCR-001 §4 (measured results against criteria) |
| Downstream | Re-validation (triggered by Phase 2 completion), MRC decision (approval of the plan) |
| Reproducibility | The plan's version is recorded in the Approval Record |

---

*End of document.*
