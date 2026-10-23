# Lending Club Credit Default Model — Approval Record

**Document ID:** LC-AR-001  
**Version / Status:** 2.2 / Draft pending MRC decision  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Approval Record (AR)  
**Decision-maker:** Model Risk Committee Chair  
**Frozen at:** NB03 v2.1 execution (run_id `20261021_145017`)  
**Companion documents:** LC-PDR-001 · LC-DS-001 · LC-FDL-001 · LC-MD-001 · LC-RVP-001 · LC-DCR-001 · LC-CFR-001 · LC-RMP-001 · MAP-VR-LC-001 (Validation Report)

---

## Record status at a glance

| Field | Value |
|---|---|
| **Record state** | DRAFT — awaiting MRC Chair signature |
| **Validation Opinion verdict** | HIGH / REJECT (risk score 6/10) |
| **Validator's position** | REJECT (recommendation to the MRC Chair) |
| **MRC Chair's decision** | `[ANALYST: decision — APPROVE REJECT / CONDITIONAL / OVERRIDE]` |
| **When decision is signed** | Record becomes binding; deployment status is determined by the decision |
| **Next review** | At completion of LC-R-01 and LC-R-03 (target: month 3) |

> **On the record's state.** This record is a **draft** until the MRC Chair's signature is applied. The Validation Opinion's verdict is a *recommendation* to the MRC Chair, not a *decision* by the MRC Chair. The record becomes binding when the signature block (§6) is complete. A reader who sees "REJECT" in §3 is seeing the validator's recommendation; a reader who sees a signature in §6 is seeing the MRC Chair's decision.

---

## 1. Purpose and scope of the record

This record documents the formal approval or rejection decision for the Lending Club Credit Default Model. It records the Validation Opinion's verdict, the MRC's decision, any conditions attached to the decision, and the signatures of the accountable parties.

The record is the **binding artifact**. The Validation Opinion reports the verdict; the Approval Record makes it official. Without this record, the verdict is a recommendation, not a decision.

---

## 2. Record structure and schema

The Approval Record has six sections:

| Section | Purpose | Populated at |
|---|---|---|
| §3 — Validation Opinion summary | What is being decided (the verdict and its drivers) | Drafting |
| §4 — Approval chain | Who has authority, and what each role's position is | Drafting |
| §5 — Decision and conditions | The MRC Chair's decision, any conditions, and the rationale | Signature |
| §6 — Signature block | Who signed, when, and with what authority | Signature |
| §7 — Document control | Version, status, supersedes, next review | Drafting + signature |
| §8 — Open items | What is pending, what is deferred | Drafting |

The record is short by design. It is a summary document, not a re-statement of the Validation Opinion. The Validation Opinion is 6 pages; the Approval Record is 2 pages. The record points to the opinion; the opinion provides the evidence.

---

## 3. Validation Opinion summary

**The Validation Opinion's verdict is HIGH / REJECT, risk score 6/10.** This is the **validator's recommendation** to the MRC Chair.

**The primary driver:** Fairness — region-level DI 0.5414 (SEVERE, below 0.60 threshold; DCR §3.3 FD-01; CFR F-03.03).

**The secondary drivers:**

- Calibration: XGBoost HL p = 0.000 (poor; DCR §3.2; CFR F-03.04)
- Performance: AUC 0.631 (MODERATE band, 1 of 6 points; DCR §3.1)
- Split method: random, not walk-forward (disclosed exception; DCR §3.1; CFR F-03.02)

**The leakage premium:** The developer's claimed AUC of 0.865 was produced with 9 leakage features present. After leakage removal, realistic AUC is 0.631 — a 27.1% relative degradation (DCR §4.1; CFR F-03.01).

**The conditions for approval:** 8 conditions specified in the Validation Report (MAP-VR-LC-001 §6), with a 3-month critical path (LC-RMP-001 §5).

**The Validation Opinion is the evidence base.** This record does not re-litigate the verdict; it records the MRC Chair's decision *on* the verdict.

---

## 4. Approval chain and decision authority

### 4.1 — The approval chain

Developer (NB02) → Validator (NB03) → Governance (NB04) → MRC Chair (final decision)


### 4.2 — Positions and authority

The table below records each role's **position** (what they recommend, attest, or acknowledge) and **authority** (what they are authorized to do). The "Decision" column is filled only by the MRC Chair, after signature.

| Role | Name | Authority | Position | Decision |
|---|---|---|---|---|
| Model Developer | `[ANALYST: name]` | Implements model; cannot self-approve | **Attests** — the model is as described in NB02; remediation ownership accepted | — |
| Independent Validator | `[ANALYST: name]` | Authors verdict; cannot deploy | **Recommends** — REJECT (Validation Opinion §1) | — |
| Governance Team Lead | `[ANALYST: name]` | Documents compliance; cannot override | **Attests** — the compliance report is complete as documented | — |
| Model Risk Committee Chair | `[ANALYST: name]` | Final approval or rejection | **Decides** — see §5 | `[ANALYST: decision — APPROVE REJECT / CONDITIONAL / OVERRIDE]` |

**On the "Position" vs "Decision" distinction.** The Developer, Validator, and Governance Team do not make decisions about deployment — they make *positions* (attestations and recommendations) that the MRC Chair considers. Only the MRC Chair has *decision* authority. The record's v2.1 "Decision" column conflated the two: it recorded `REJECT` for the Validator, which is the Validator's recommendation, not a decision by the Validator. The v2.2 table separates them.

### 4.3 — The three options available to the MRC Chair

The MRC Chair has three options, each with different consequences:

| Option | Meaning | Consequences |
|---|---|---|
| **APPROVE REJECT** | Accept the validator's verdict | The model is not deployed. The remediation plan (LC-RMP-001) proceeds. A new Approval Record is required when re-validation produces a new verdict. |
| **CONDITIONAL** | Approve with conditions (e.g., "deploy with fairness monitoring") | The model is deployed subject to the stated conditions. The conditions must be specified in §5. A monitoring plan is required. |
| **OVERRIDE** | Override the verdict (e.g., "deploy despite the REJECT") | The model is deployed. The override requires documented rationale (§5) and triggers re-validation on the affected dimension (LC-RVP-001 §7.3). |

**A note on the "not recommended" language in the v2.1 record.** The v2.1 record said CONDITIONAL was "not recommended given the severity of F-03.03" and OVERRIDE was "generally discouraged for fairness findings." The v2.2 record does not include that language. **The MRC Chair's authority is to decide; the record's role is to present the options and their consequences, not to advise on which option to choose.** If the analyst wants to make a recommendation, it belongs in the Validation Report (which recommends REJECT), not in the Approval Record (which records the MRC Chair's decision).

---

## 5. Decision and conditions

**Record state:** This section is empty until the MRC Chair's decision is recorded and signed. The fields below are populated by the MRC Chair at signature time.

**The MRC Chair's decision:** `[ANALYST: decision — APPROVE REJECT / CONDITIONAL / OVERRIDE]`

**The rationale:** `[ANALYST: rationale — required for all decisions; critical for OVERRIDE]`

**The conditions (if CONDITIONAL or OVERRIDE):** `[ANALYST: conditions — specified in full; each condition must have an owner, a due date, and a monitoring plan]`

**The conditions for re-validation (if APPROVE REJECT):** The 8 conditions from the Validation Report (MAP-VR-LC-001 §6) apply. The re-validation trigger is Phase 2 completion (target: month 3; LC-RMP-001 §5.1).

**The decision is the record's binding content.** Everything else is context. If §5 is empty, the record is a draft; if §5 is filled and §6 is signed, the record is binding.

---

## 6. Signature block

**The signature block records who signed, when, and with what authority.** The "Position" column repeats the position from §4.2; the "Signed" column records the signature. The "Decision" column is filled only by the MRC Chair.

| Role | Name | Position | Signature | Date | Decision |
|---|---|---|---|---|---|
| Model Developer | `[ANALYST: name]` | Attests | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| Independent Validator | `[ANALYST: name]` | Recommends REJECT | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| Governance Team Lead | `[ANALYST: name]` | Attests | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| Model Risk Committee Chair | `[ANALYST: name]` | Decides | `[ANALYST: signature]` | `[ANALYST: date]` | `[ANALYST: decision]` |

**The signature block is the record's formal completion.** Without signatures, the record is a draft. The signatures convert the verdict into a decision.

---

## 7. Document control

| Field | Value |
|---|---|
| Version | 2.2 |
| Status | Draft pending MRC decision |
| Supersedes | None (first version) |
| Next review | At completion of LC-R-01 and LC-R-03 (target: month 3) |
| Related documents | MAP-VR-LC-001 (Validation Report), LC-CFR-001 (Findings Register), LC-RMP-001 (Remediation Plan) |
| Record ID | `[ANALYST: record_id]` |

---

## 8. Open items and dependencies

### 8.1 — Open items

| # | Item | Owner | Due | Routing |
|---|---|---|---|---|
| 1 | **MRC decision** | MRC Chair | `[ANALYST: date]` | MRC meeting |
| 2 | **Remediation plan approval** | MRC Chair | `[ANALYST: date]` | The plan can be approved even if the model is rejected |
| 3 | **Re-validation trigger** | Validator | Month 3 | When LC-R-01 and LC-R-03 complete (LC-RMP-001 §5.1) |

**On the relationship between the model decision and the plan decision.** The MRC's decision on the model (this record) and its decision on the remediation plan (LC-RMP-001) are separate. The plan can be approved even if the model is rejected — the plan is the path from REJECT to APPROVE, and it needs approval to begin regardless of whether the model is deployed. The record notes this so that a reader does not assume the plan's approval is contingent on the model's approval.

### 8.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | MAP-VR-LC-001 (source of the verdict), LC-RMP-001 (source of the conditions) |
| Downstream | Re-validation (triggered by the decision), deployment (authorized by the decision) |
| Reproducibility | The record's version is recorded in the MRC's decision log |

---

*End of document.*

