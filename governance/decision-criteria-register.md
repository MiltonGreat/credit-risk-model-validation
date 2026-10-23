# Lending Club Credit Default Model — Decision Criteria Register

**Document ID:** LC-DCR-001  
**Version / Status:** 2.2  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Decision Criteria Register (DCR)  
**Frozen at:** NB03 v2.1 execution (run_id `20261021_145017`)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Define every decision criterion used in the Lending Club pipeline, its threshold, its source, and its authority for change |
| **Criterion families** | Performance (§3.1), Calibration (§3.2), Fairness (§3.3), Stability (§3.4), Explainability (§3.5), Governance (§3.6) |
| **Verdict scale** | 0–10 risk score, four bands (LOW / MODERATE / HIGH / CRITICAL) |
| **This run's verdict** | 6/10 → HIGH → REJECT |
| **Authority for threshold changes** | MRC Chair (sole approver); developer proposes, validator reviews |
| **Version chain** | v2.1 → v2.2 (this revision) → v3.0 (on any MAJOR threshold change) |

> **Document purpose.** This register records every decision criterion used in the Lending Club pipeline: the thresholds for AUC, KS, DI, HL, PSI, SHAP direction, and the weights in the risk score. It specifies who set each threshold, why, and who has authority to change it.
>
> **Structure.** The document has four parts: (1) the **criterion catalogue** (§3) — thresholds and bands, no results; (2) the **results against criteria** (§4) — measured values, pass/fail, points; (3) the **verdict record** (§5) — the risk score, rating, and recommendation; (4) the **governance** (§6–§10) — guards, change control, authority, open items. A reader who wants to know "what was the threshold?" reads §3; a reader who wants to know "did the model pass?" reads §4; a reader who wants to know "what is the verdict?" reads §5.

---

## 1. Purpose and scope of the register

This register records every decision criterion used in the Lending Club pipeline: the thresholds for AUC, KS, DI, HL, PSI, SHAP direction, and the weights in the risk score. It specifies who set each threshold, why, and who has authority to change it.

**The distinction from the RVP:** the RVP says what will be tested; the DCR says what the thresholds are, and where they come from. The RVP is the plan; the DCR is the parameters.

**The distinction from the Validation Report:** the Validation Report reports what the validation found; the DCR defines what "found" means against the criteria. The DCR is the *basis* for the verdict; the Validation Report is the *statement* of the verdict against that basis.

**The distinction from the Problem Definition Record:** the PDR fixes the *success criteria* before the model is built (e.g., "AUC ≥ 0.65 will be the threshold for acceptable performance"). The DCR fixes the *criterion catalogue* — the full set of thresholds, bands, and points, including those that go beyond the PDR's pre-registered criteria (e.g., KS, PSI, SHAP direction). The PDR states what success looks like; the DCR states how success is measured and scored.

---

## 2. Criterion catalogue overview

The Lending Club DCR contains six criterion families:

| Family | Criteria | Source |
|---|---|---|
| §3.1 — Performance | AUC, KS, precision, recall | DCR §1 |
| §3.2 — Calibration | HL p-value, slope, intercept | DCR §2 |
| §3.3 — Fairness | DI, DPD, EOD | DCR §3 (FD-01, FD-03, FD-05) |
| §3.4 — Stability | PSI | DCR §4 |
| §3.5 — Explainability | SHAP direction consistency | DCR §5 |
| §3.6 — Governance | Compliance score, verdict bands | DCR §6–§9 |

The six families span every dimension of the validation. Each family has its own thresholds, its own pass/fail logic, and its own authority for change.

---

## 3. Criterion catalogue — thresholds and bands

This section defines the criteria and their bands. **It does not report this run's results** — the results are in §4. A reader who wants to know "what is the AUC threshold?" reads §3.1; a reader who wants to know "did this run meet it?" reads §4.

### 3.1 — Performance criteria

**AUC-ROC bands:**

| Criterion | Band | Points | Source |
|---|---|---|---|
| AUC-ROC | ≥ 0.65 | 0 | DCR §1 |
| AUC-ROC | 0.60–0.65 | 1 | DCR §1 |
| AUC-ROC | 0.55–0.60 | 2 | DCR §1 |
| AUC-ROC | < 0.55 | 3 | DCR §1 |

**KS bands:**

| Criterion | Band | Action | Source |
|---|---|---|---|
| KS | ≥ 0.40 | 0 (strong) | DCR §1 |
| KS | 0.30–0.40 | 1 (moderate) | DCR §1 |
| KS | < 0.30 | HIGH finding | DCR §1 |

**Precision:**

| Criterion | Band | Action | Source |
|---|---|---|---|
| Precision | Reviewed against business cost | — | DCR §1 (not a fixed threshold) |

**Rationale for the performance bands.** The AUC bands are the standard risk appetite. The 0.65 target is the DCR's minimum for "acceptable" performance; below 0.55 is "unacceptable." The three-point scale maps to the risk score's performance component (0–3).

**The AUC/KS asymmetry is deliberate.** AUC drives the risk score (0–3 points); KS drives a finding (HIGH if < 0.30, no direct points). The asymmetry reflects that AUC and KS answer different questions: AUC measures overall ranking ability; KS measures the maximum separation between the default and non-default distributions. A model can have a moderate AUC and a weak KS — the two tests are informative about different aspects of discrimination. The DCR scores AUC because it is the more stable overall measure; it flags KS because a weak KS is a specific concern for tail prioritization (the collections use case).

### 3.2 — Calibration criteria

| Criterion | Band | Action | Source |
|---|---|---|---|
| HL p-value | > 0.05 | GOOD | DCR §2 |
| HL p-value | ≤ 0.05 | POOR finding | DCR §2 |
| HL p-value | **exactly** 0.0 or 1.0 | SUSPICIOUS — flag | DCR §2 (E-M03-06) |
| Calibration slope | ~1.0 | Well-calibrated | DCR §2 |
| Calibration intercept | ~0.0 | Well-calibrated | DCR §2 |

**Rationale for the calibration bands.** The HL test is the standard calibration test; p > 0.05 is the conventional "do not reject the null of good calibration" threshold. The **degeneracy check** is a DCR requirement, not an optional diagnostic: a calibration finding based on p = 1.0 is not a finding — it is a numerical artefact (usually a bin with zero expected events, producing 0/0 in the HL statistic). The DCR requires the degeneracy check because the alternative — treating a p-value of 1.0 as good calibration — would produce false confidence in the model's probabilities.

**The slope and intercept criteria are diagnostic, not scored.** They report whether the calibration is well-behaved (slope ~1.0, intercept ~0.0), but they do not contribute points to the risk score. The HL test is the scored criterion; the slope and intercept are interpretive context.

### 3.3 — Fairness criteria (FD-01, FD-03, FD-05)

**FD-01 — Disparate Impact (DI) bands:**

| DI band | Category | Points | Source |
|---|---|---|---|
| DI ≥ 0.80 | OK | 0 | DCR §3 FD-01 |
| 0.60 ≤ DI < 0.80 | MODERATE | 1 | DCR §3 FD-01 |
| DI < 0.60 | SEVERE | 3 | DCR §3 FD-01 |

**FD-03 — Minimum group size:**

| Criterion | Value | Source |
|---|---|---|
| Minimum N per group | 30 | DCR §3 FD-03 |
| Groups with N < 30 | Excluded from DI calculation | DCR §3 FD-03 |

**FD-05 — Maximum cardinality:**

| Criterion | Value | Source |
|---|---|---|
| Maximum cardinality for 4/5ths test | 50 | DCR §3 FD-05 |
| Above 50 | Test not valid | DCR §3 FD-05 |

**Rationale for the fairness bands.** The three FD rules work together. FD-05 says "you can only test attributes with ≤ 50 groups"; FD-03 says "exclude groups smaller than 30"; FD-01 says "here is how to interpret the DI that results." A test that ignores any of the three is not FD-compliant.

**The most important DCR rule for Lending Club:** per-state one-hot DI (v2.0's approach) is **not FD-05 compliant** — it compares one state to the complement of 49 others, which produces a degenerate comparison at high cardinality (many states have zero defaults, so the minimum approval rate is 0, and the DI ratio is 0). The DCR says this explicitly, and NB03 enforces it via region grouping. A validator who re-introduces per-state DI is not applying the DCR.

### 3.4 — Stability criteria

| Criterion | Band | Points | Source |
|---|---|---|---|
| PSI | < 0.15 | 0 (STABLE) | DCR §4 |
| PSI | 0.15–0.25 | 1 (MODERATE) | DCR §4 |
| PSI | > 0.25 | 2 (MAJOR) | DCR §4 |
| PSI | `not_assessed` | 0 (no penalty) | DCR §4 (E-M03-05) |

**Rationale for the stability bands.** The PSI bands are the standard stability thresholds. The **`not_assessed` rule is a DCR decision, not a default**: the DCR could have said "not_assessed → 2 points" (penalize for missing evidence), but it says 0. The choice reflects the principle that a data limitation should not penalize the model — if the dataset cannot support a time-based PSI, the model should not be scored as if its stability were in question. A validator who disagrees with the principle can change the DCR — but the change is a **MAJOR** version bump.

### 3.5 — Explainability criteria

| Criterion | Value | Action | Source |
|---|---|---|---|
| Top-10 SHAP direction consistency | All consistent | No finding | DCR §5 |
| Counterintuitive features | ≥ 1 | HIGH finding | DCR §5 |
| Leakage features in top-20 SHAP | Any | CRITICAL finding | DCR §5 |

**Rationale for the explainability criteria.** The SHAP direction check is the DCR's most interpretive criterion. It relies on a heuristic ("higher FICO is lower risk") that is well-established in credit risk. A feature that contradicts the heuristic could be (a) a data quality issue, (b) a genuine pattern the heuristic does not capture, or (c) a spurious pattern. The DCR requires the finding; the analyst decides which explanation applies. The **CRITICAL severity for leakage features in the top-20** reflects that a leakage feature in the model's top drivers would mean the leakage removal was incomplete — a data-integrity failure, not an explainability concern.

### 3.6 — Governance and verdict bands

**Risk score components:**

| Component | Max points | Source |
|---|---|---|
| Performance | 3 | DCR §1 |
| Stability | 2 | DCR §4 |
| Calibration | 2 | DCR §2 |
| Fairness | 3 | DCR §3 FD-01 |
| Leakage | 3 | DCR §3 |
| **Total** | **10** | DCR §9 |

**Verdict bands:**

| Total points | Rating | Recommendation |
|---|---|---|
| 0–2 | LOW | APPROVE |
| 3–4 | MODERATE | APPROVE WITH CONDITIONS |
| 5–7 | HIGH | REJECT |
| 8–10 | CRITICAL | REJECT |

**The weights and bands are frozen in DCR §9.** Changing any of them is a MAJOR version bump. The DCR is the document that makes "the model is REJECT" verifiable — the arithmetic traces to the DCR, not to the analyst's judgment.

---

## 4. Results against criteria — this run

This section reports the measured values against the criteria in §3, the pass/fail determination, and the points contributed to the risk score. **The measured values are here, not in §3** — a reader who wants to know "what is the AUC threshold?" reads §3.1; a reader who wants to know "did this run meet it?" reads this section.

### 4.1 — Performance results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| AUC-ROC | 0.631 | 0.60–0.65 | **FAIL** | 1 | — |
| KS | 0.2671 | < 0.30 | **FAIL** | 0 | F-03.06 (HIGH) |
| Precision | 0.085 | Reviewed against business cost | — (no fixed threshold) | — | F-03.11 (MEDIUM) |

**AUC 0.631 → MODERATE band, 1 point.** The model's overall ranking ability is below the DCR's target (≥ 0.65) but above the "unacceptable" floor (0.55). The 1-point contribution is a moderate signal, not a driver of the verdict.

**KS 0.2671 → Weak, HIGH finding F-03.06.** The tail separation is insufficient for collections prioritization. No direct points (per the AUC/KS asymmetry in §3.1), but the finding is HIGH severity and appears in the findings register.

**Precision 0.085 → MEDIUM finding F-03.11.** The false-positive rate at the operating threshold is high. No fixed threshold; the analyst reviews against business cost.

### 4.2 — Calibration results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| HL p-value — LightGBM | 0.729 | > 0.05 | **PASS** | 0 | — |
| HL p-value — XGBoost | 0.000 | ≤ 0.05 | **FAIL (POOR)** | 2 | F-03.04 (HIGH) |
| HL p-value — Logistic Regression | 1.000 | exactly 1.0 | **SUSPICIOUS** | — | F-03.05 (MEDIUM) |
| HL p-value — Random Forest | 1.000 | exactly 1.0 | **SUSPICIOUS** | — | F-03.05 (MEDIUM) |
| Calibration slope — LightGBM | 1.078 | ~1.0 | PASS (diagnostic) | — | — |

**Summary:** 1 of 4 models has a valid HL result (LightGBM); 1 is poorly calibrated (XGBoost); 2 are degenerate (Logistic Regression, Random Forest). **The calibration component of the risk score is 2 (XGBoost poor).** The degenerate p-values are flagged as F-03.05 (MEDIUM), not scored — a degenerate HL p-value is not a calibration finding, per the §3.2 degeneracy rule.

### 4.3 — Fairness results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| Region-level DI | 0.5414 | SEVERE (< 0.60) | **FAIL** | 3 | F-03.03 (CRITICAL) |
| Cardinality | 4 (South, West, Northeast, Midwest) | ≤ 50 | PASS (FD-05) | — | — |
| Group sufficiency | "Other" (N=6) excluded | ≥ 30 per group | PASS (FD-03) | — | — |

**Region-level DI 0.5414 → SEVERE band, 3 points, CRITICAL finding F-03.03.** This is the **primary driver of the REJECT verdict.** The FD-05 cardinality check passes (4 groups), and the FD-03 group sufficiency check passes ("Other" excluded). The DI test is FD-compliant; the result is a real severe disparity.

### 4.4 — Stability results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| PSI | `not_assessed` | — | — (no penalty) | 0 | — |

**PSI `not_assessed` on a random split → 0 points.** The stability component is 0, which is **not** evidence of stability — it is the DCR's treatment of "not assessed" as no-penalty (per §3.4). Documented in the Validation Report as a limitation: PSI requires a time-based split, which the dataset does not support (F-03.02).

### 4.5 — Explainability results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| Top-10 SHAP direction consistency | 6 of 10 consistent | — | **FAIL** | 0 | F-03.07 (HIGH) |
| Counterintuitive features | 4 (`loan_to_income`, `total_collection_amount_ever`, `annual_income`, `total_credit_limit`) | ≥ 1 | **FAIL** | — | F-03.07 (HIGH) |
| Leakage features in top-20 SHAP | 0 | — | PASS | — | — |

**4 counterintuitive SHAP features → HIGH finding F-03.07, 0 points.** The finding is not scored directly (explainability is not a risk-score component), but it appears in the findings register and must be addressed before the model's drivers can be trusted. The leakage check is clean — no leakage features in the top-20.

### 4.6 — Governance results

| Criterion | Measured | Band | Pass/Fail | Points | Finding |
|---|---|---|---|---|---|
| Compliance score | 74.1 / 100 | ≥ 80 target | **FAIL** | — | See MAP-VR-LC-001 §4 |

**Compliance score 74.1 / 100 → below the DCR's 80 target.** This is a governance finding, not a risk-score component (governance is not scored separately; it is tracked in the compliance report and the Validation Report).

---

## 5. Verdict record — this run

### 5.1 — Risk score components

| Component | Points | Basis |
|---|---|---|
| Performance | 1 / 3 | AUC 0.631, MODERATE band |
| Stability | 0 / 2 | PSI `not_assessed` (no penalty per §3.4) |
| Calibration | 2 / 2 | XGBoost poor (HL p = 0.000) |
| **Fairness** | **3 / 3** | **DI 0.5414, SEVERE** |
| Leakage | 0 / 3 | All 9 leakage features removed (F-03.01 resolved) |
| **Total** | **6 / 10** | — |

### 5.2 — Rating and recommendation

| Total points | Rating | Recommendation |
|---|---|---|
| **6** | **HIGH** | **REJECT** |

### 5.3 — What "REJECT" means

**REJECT is a hard prohibition on use for the intended purpose.** It is not a recommendation, a soft flag, or a conditional approval. A model rated HIGH (5–7 points) or CRITICAL (8–10 points) may not be deployed for any consequential decision — in this case, collections prioritization — until the findings that drove the rating are closed and a re-validation demonstrates closure.

**What REJECT permits:**

- **Methodology demonstration** — the model may be used to illustrate the validation methodology, as long as every use is labelled as illustrative and the model is not the basis for any decision.
- **Remediation work** — the model may be modified as part of the remediation plan (MAP-RP-001); the modified model is re-validated before any use.

**What REJECT prohibits:**

- **Collections prioritization** — the model may not be used to rank accounts, allocate collections effort, or inform any customer-facing decision.
- **Any consequential use** — the model may not be the basis for any decision that affects a borrower, a portfolio, or a regulatory submission.

**Who can override the REJECT:** the MRC Chair, with documented rationale (per LC-RVP-001 §6.2). An override is itself a recorded decision and triggers re-validation on the affected dimension.

### 5.4 — Primary driver of the verdict

**The rejection is fairness-driven, not performance-driven.** AUC 0.631 contributes 1 of 6 points; fairness (3 points) and calibration (2 points) dominate. A reader who sees "AUC 0.631, REJECT" and concludes performance was the driver has misread the verdict. The fairness finding (F-03.03, region-level DI 0.5414, SEVERE) is the single largest contributor and the one that must be closed first for the verdict to change.

---

## 6. Automated threshold guards

Lending Club's DCR uses a set of automated guards that fire based on thresholds, not on analyst judgment. A threshold change is a DCR change, not a code change.

| Guard | Triggers when | Finding | Source |
|---|---|---|---|
| LC-G1 (leakage) | Leakage features present | F-03.01 CRITICAL | DCR §3 |
| LC-G2 (fairness) | DI < 0.60 | F-03.03 CRITICAL | DCR §3 FD-01 |
| LC-G3 (calibration) | HL p ≤ 0.05 | F-03.04 HIGH | DCR §2 |
| LC-G4 (KS) | KS < 0.30 | F-03.06 HIGH | DCR §1 |
| LC-G5 (SHAP direction) | Counterintuitive features | F-03.07 HIGH | DCR §5 |
| LC-G6 (split method) | Not time-based | F-03.02 HIGH | DCR §1 |

**On the naming.** The Lending Club guards use the `LC-GN` convention (Lending Club Guard, number N). This is a **local convention** with no cross-project meaning. The Fannie Mae pipeline uses `AP-02`/`AP-03`/`AP-04` for its register guards; those are unrelated to the Lending Club guards. A reader who sees `LC-G2` and `AP-02` should not assume they refer to the same thing — they do not. The two projects have separate guard numbering schemes, and the guards fire on different thresholds.

**The critical point:** register guards fire based on thresholds, not on analyst judgment. The DCR is what makes them automated — a threshold change is a DCR change, not a code change. A guard that fires without a corresponding DCR entry is a code bug; a DCR entry that does not fire is a data finding.

---

## 7. Change control and versioning

### 7.1 — Change control rules

| Change type | Version bump | Requires | Example |
|---|---|---|---|
| Threshold change (e.g., AUC ≥ 0.70 instead of 0.65) | MAJOR | Methodology Owner approval; full re-validation | Changing DCR §1 AUC band |
| Adding a new criterion | MINOR | Validator approval; documented in DCR | Adding a new fairness metric |
| Clarifying wording | PATCH | No approval; documented | Fixing a typo in DCR §3 |
| Changing the verdict bands (LOW/MODERATE/HIGH) | MAJOR | MRC approval; full re-validation | Changing §3.6 band boundaries |

**The rule:** any change to a threshold that affects the verdict is a **MAJOR** version bump. Adding a new criterion is **MINOR**. Clarifying is **PATCH**.

**The version chain:** DCR v2.1 → DCR v2.2 (this revision — structural reorganization, no threshold changes; **MINOR**) → DCR v3.0 (on any MAJOR threshold change). The version number is recorded in every artifact that references the DCR.

> **On this revision (v2.2).** The changes from v2.1 to v2.2 are **structural**: the criterion catalogue is separated from the results, the verdict record is consolidated, the register guards are renamed from AP-XX to LC-GN, and the "what REJECT means" definition is added. **No threshold values changed.** The version bump is MINOR because the criteria are identical to v2.1 — a reader can compare v2.1 and v2.2 and find the same AUC bands, the same DI bands, the same PSI bands, the same verdict bands. Only the organization changed.

### 7.2 — Version history

| Version | Date | Change | Type |
|---|---|---|---|
| 2.1 | (frozen at NB03 v2.1) | — | — |
| 2.2 | (this revision) | Structural reorganization; LC-GN naming; REJECT definition added; no threshold changes | MINOR |

---

## 8. Authority and approval

| Change | Proposer | Approver | Rationale required |
|---|---|---|---|
| AUC threshold | Developer | MRC Chair | Yes |
| Fairness threshold (DI ≥ 0.80) | Validator | MRC Chair | Yes |
| New fairness criterion (DPD, EOD) | Validator | MRC Chair | Yes |
| Verdict band boundaries | MRC Chair | MRC Chair | Yes |
| Calibration HL bands | Validator | MRC Chair | Yes |
| SHAP direction heuristic | Validator | MRC Chair | Yes |

**The MRC Chair is the sole authority for threshold changes.** This is the separation of duties: the developer implements, the validator proposes, the MRC decides. No single role can change a threshold unilaterally — the change requires both a proposer and an approver.

**The authority chain:**

Developer (proposes) → Validator (reviews) → MRC Chair (approves)


A threshold change that bypasses this chain is a **MAJOR finding** in the next validation.

---

## 9. Rating and verdict systems — the three scales

The DCR uses three rating systems, each with its own bands:

| System | Scale | Bands | Purpose |
|---|---|---|---|
| Risk score | 0–10 | LOW / MODERATE / HIGH / CRITICAL | Model deployability |
| Component scores | per criterion | 0–3 for AUC, 0–3 for fairness, etc. | Risk score inputs |
| Finding severities | qualitative | CRITICAL / HIGH / MEDIUM / LOW / RESOLVED | Finding-level classification |

**The three are related but distinct.** A model can have a HIGH risk score (6/10) while having no CRITICAL findings (if all findings are HIGH). The risk score is the composite; the findings are the individual issues. A reader who confuses the two may think "HIGH risk score" means "has CRITICAL findings" — it does not.

---

## 10. Open items and dependencies

### 10.1 — Open items

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **KS threshold integration** | KS < 0.30 is a HIGH finding but contributes 0 to the risk score. A future DCR might integrate KS into the score (e.g., 0–2 points). | MINOR version bump; proposer = Validator; approver = MRC Chair (per §8) |
| 2 | **Precision threshold** | "Reviewed against business cost" — no fixed threshold. A future DCR might set a precision floor (e.g., ≥ 0.10) for collections use. | MINOR version bump; proposer = Validator; approver = MRC Chair |
| 3 | **PSI `not_assessed` rule** | Currently 0 points. A future DCR might penalize `not_assessed` (e.g., 1 point) to encourage time-based validation. | MAJOR version bump; proposer = Validator; approver = MRC Chair (per §7.1) |
| 4 | **F1 threshold** | Currently not scored. A future DCR might include F1 in the performance band. | MINOR version bump; proposer = Validator; approver = MRC Chair |

**On the routing column.** Each open item is routed to the DCR's own change-control chain (§7, §8), not to a separate roadmap. A DCR open item is a *proposed DCR change*, not a *finding* — it does not belong in MAP-RP-001, and it does not need an owner outside the DCR's authority chain. The proposer/approver columns in §8 are the routing.

### 10.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | LC-MD-001 (Methodology), LC-RVP-001 (Risk & Validation Plan) |
| Downstream | NB03 (verdict computation), MAP-VR-LC-001 (Validation Report) |
| Reproducibility | The DCR version is recorded in `performance_summary_v2.json` |
| Cross-project | The Lending Club guards (`LC-GN`) are unrelated to the Fannie Mae register guards (`AP-XX`); see §6 |

---

*End of document.*
