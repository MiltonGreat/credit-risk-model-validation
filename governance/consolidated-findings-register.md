# Lending Club Credit Default Model — Consolidated Findings Register

**Document ID:** LC-CFR-001  
**Version / Status:** 2.2  
**Project:** Lending Club — credit-risk classification / validation demonstration  
**Document type:** Consolidated Findings Register (CFR)  
**Source registers:** NB01 `findings_register.csv` · NB02 `findings_register_v2.csv` · NB03 `validation_findings_register.csv`  
**Frozen at:** NB03 v2.1 execution (run_id `20261021_145017`)  
**Companion documents:** LC-PDR-001 (Problem Definition Record) · LC-DS-001 (Data Specification) · LC-FDL-001 (Feature Decision Log) · LC-MD-001 (Methodology Document) · LC-RVP-001 (Risk & Validation Plan) · LC-DCR-001 (Decision Criteria Register) · MAP-VR-LC-001 (Validation Report) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Total findings** | 26 (across NB01, NB02, NB03) |
| **By severity** | 4 CRITICAL · 14 HIGH · 8 MEDIUM · 0 LOW |
| **By status** | 5 RESOLVED · 21 OPEN |
| **Distinct issues** | 2 CRITICAL · 8 HIGH · 6 MEDIUM (distinct = same issue across notebooks counted once) |
| **Full record** | §3 (every finding with all columns) |
| **Cross-register chain** | §7 (NB01 → NB02 → NB03 mapping) |
| **Blocking finding** | F-03.03 (severe disparate impact, DI 0.5414) — the finding that drives the verdict |

> **Document purpose.** This register consolidates every finding produced across NB01, NB02, and NB03 into a single document. It is the complete record of what was found — not a summary, not a re-derivation. Every finding ID traces to its source notebook, and every finding carries its evidence reference, remediation, owner, timeline, and status.
>
> **On the relationship between this register and the README.** This register records findings; the README *narrates* them. A reviewer who wants the portfolio story ("the model had a leakage problem and a fairness problem") should read the README; a reviewer who wants the individual findings should read this register. The register supports the README's narrative; it is not a substitute for it.

---

## 1. Purpose and scope of the register

This register consolidates every finding produced across NB01, NB02, and NB03 into a single document. It is the complete record of what was found — not a summary, not a re-derivation. Every finding ID traces to its source notebook, and every finding carries its evidence reference, remediation, owner, timeline, and status.

**The distinction from the Validation Report (MAP-VR-LC-001):** the Validation Report reports the verdict (HIGH / REJECT) and the analysis that supports it. The register reports the individual findings that drive the verdict. The verdict is one number; the findings are the reasons for it.

**The distinction from the DCR (LC-DCR-001):** the DCR says what the criteria are; the register says what was found against them. The DCR is the standard; the register is the evidence.

**The distinction from the Remediation Plan (MAP-RP-001):** the Remediation Plan tracks the *closure* of findings (owners, actions, milestones, evidence). The register records the *existence* of findings. A finding can exist in this register and not yet be tracked in the Remediation Plan (e.g., a scope finding that is permanently OPEN and has no remediation action).

---

## 2. Register structure and schema

The consolidated register has 15 columns per finding:

| Column | Type | Purpose |
|---|---|---|
| `Finding_ID` | string | Stable identifier (F-01.XX, F-02.XX, F-03.XX) |
| `Source_Notebook` | string | Which notebook produced it |
| `Severity` | enum | CRITICAL / HIGH / MEDIUM / LOW |
| `Finding` | string | Short name |
| `Description` | string | What was found |
| `Evidence_Reference` | string | Where the evidence is |
| `Evidence_Hash` | string | SHA-256-16 of the evidence artifact |
| `Remediation` | string | What to do about it |
| `Owner` | string | Who owns the remediation |
| `Timeline` | string | When it is due |
| `Status` | enum | RESOLVED / OPEN / IN PROGRESS / DEFERRED |
| `Run_ID` | string | Which pipeline run produced it |
| `Notebook_Version` | string | Which version of the notebook |
| `Cross_Reference` | string | Links to related findings |
| `Notes` | string | Analyst commentary |

**The `Cross_Reference` column is the register's connective tissue.** A finding in NB03 that references an NB01 finding is not a duplicate — it is the validation of the earlier finding. The cross-reference makes the chain visible.

**The `Evidence_Hash` column makes the register auditable.** A reviewer can verify the finding traces to a real artifact with a known hash. Findings without hashes (absence-based findings) are marked `n/a` — the absence is the evidence.

---

## 3. Full findings record

This section lists every finding with the columns a reviewer needs to locate it, assess it, and route it. **This is the register's core content.** The notebook-grouped summaries in §4, the severity grouping in §5, and the status grouping in §6 are all *views* of this table.

| Finding_ID | Source | Severity | Finding | Status | Owner | Timeline | Cross_Ref |
|---|---|---|---|---|---|---|---|
| F-01.01 | NB01 | CRITICAL | 9 leakage features detected | RESOLVED | Model Dev | Complete | F-02.01, F-03.01 |
| F-01.02 | NB01 | HIGH | Temporal audit not previously documented | RESOLVED | Model Dev | Complete | — |
| F-01.03 | NB01 | HIGH | Feature inventory not previously documented | RESOLVED | Model Dev | Complete | — |
| F-01.04 | NB01 | HIGH | No macroeconomic factors | OPEN (scope) | Model Dev | 6 months | F-03.08 |
| F-01.05 | NB01 | HIGH | Foreign data applicability (U.S. only) | OPEN (scope) | Model Risk | 6 months | F-03.09 |
| F-01.06 | NB01 | HIGH | IFRS 9 staging misalignment (16+ days) | OPEN (scope) | Model Dev | 3 months | F-02.08, F-03.10 |
| F-01.07 | NB01 | MEDIUM | High-missing features (>20%) | OPEN (docs) | Model Dev | NB02 | — |
| F-01.08 | NB01 | MEDIUM | `emp_years` guard hand-off for NB02 | OPEN (docs) | Model Dev | NB02 | — |
| F-02.01 | NB02 | CRITICAL | 9 leakage features removed | RESOLVED | Model Dev | Complete | F-01.01, F-03.01 |
| F-02.02 | NB02 | HIGH | Walk-forward split not implemented | OPEN (rem) | Model Dev | Next version | F-03.02 |
| F-02.04 | NB02 | MEDIUM | Poor calibration in XGBoost | OPEN (rem) | Model Dev | 1 month | F-03.04 |
| F-02.05 | NB02 | MEDIUM | Suspicious HL in 3 models | OPEN (rem) | Model Dev | Next version | F-03.05 |
| F-02.06 | NB02 | MEDIUM | Low precision (0.029) | OPEN (rem) | Model Dev | 1 month | F-03.11 |
| F-02.07 | NB02 | MEDIUM | Per-state DI below 0.80 | OPEN (docs) | Model Risk | NB03 | (superseded by F-03.03) |
| F-02.08 | NB02 | HIGH | IFRS 9 staging misalignment | OPEN (scope) | Model Dev | 3 months | F-01.06, F-03.10 |
| F-03.01 | NB03 | CRITICAL | 9 leakage features removed (verification) | RESOLVED | Model Dev | Complete | F-01.01, F-02.01 |
| F-03.02 | NB03 | HIGH | Walk-forward split not implemented | OPEN (rem) | Model Dev | Next version | F-02.02 |
| F-03.03 | NB03 | CRITICAL | Severe disparate impact (DI=0.5414) | OPEN (rem, **blocking**) | Model Risk | 3 months | (new in NB03) |
| F-03.04 | NB03 | HIGH | Poor calibration in XGBoost | OPEN (rem) | Model Dev | 1 month | F-02.04 |
| F-03.05 | NB03 | MEDIUM | Suspicious HL in 3 models | OPEN (rem) | Model Dev | Next version | F-02.05 |
| F-03.06 | NB03 | HIGH | Weak discrimination (KS=0.2671) | OPEN (rem) | Model Dev | 2 months | — |
| F-03.07 | NB03 | HIGH | Counterintuitive SHAP (4 features) | OPEN (rem) | Model Dev | 2 weeks | — |
| F-03.08 | NB03 | HIGH | No macro indicators | OPEN (scope) | Model Dev | 6 months | F-01.04 |
| F-03.09 | NB03 | HIGH | U.S. data only | OPEN (scope) | Model Risk | 6 months | F-01.05 |
| F-03.10 | NB03 | HIGH | IFRS 9 staging misalignment | OPEN (scope) | Model Dev | 3 months | F-01.06, F-02.08 |
| F-03.11 | NB03 | MEDIUM | Low precision (0.029) | OPEN (rem) | Model Dev | 1 month | F-02.06 |

**On the status qualifiers.** The register's `Status` column uses three qualifiers in parentheses to distinguish what kind of OPEN a finding is:

- **OPEN (rem)** — the finding has a remediation action tracked in MAP-RP-001. The timeline is the remediation due date.
- **OPEN (docs)** — the finding has a documentation owner but no remediation action. It is closed by documenting the decision, not by changing the model.
- **OPEN (scope)** — the finding is a scope statement (e.g., "U.S. data only"). It is permanently OPEN until the scope changes (e.g., Canadian data is acquired). It has no remediation action in the current plan; the "owner" and "timeline" columns indicate who monitors it and when a scope change might be considered.

**Why the qualifiers matter.** The un-qualified status "OPEN" treats all 21 findings as if they were the same kind of thing. They are not: 8 are remediation-tracked, 4 are documentation-closed, and 9 are scope statements. A reader who sees "21 OPEN" might think 21 problems require action; the qualifiers show that only 8 require action, 4 require documentation, and 9 are scope statements the model acknowledges. The distinction is what makes the register honest about the remediation workload.

---

## 4. Findings by notebook — summaries

These are **summaries** of §3 grouped by source notebook, for a reader who wants to see which notebook raised which findings. **The full record is §3**; these tables are views of it.

### NB01 — Exploratory Data Analysis (8 findings)

| ID | Severity | Finding | Status |
|---|---|---|---|
| F-01.01 | CRITICAL | 9 leakage features detected | RESOLVED |
| F-01.02 | HIGH | Temporal audit not previously documented | RESOLVED |
| F-01.03 | HIGH | Feature inventory not previously documented | RESOLVED |
| F-01.04 | HIGH | No macroeconomic factors | OPEN (scope) |
| F-01.05 | HIGH | Foreign data applicability (U.S. only) | OPEN (scope) |
| F-01.06 | HIGH | IFRS 9 staging misalignment (16+ days) | OPEN (scope) |
| F-01.07 | MEDIUM | High-missing features (>20%) | OPEN (docs) |
| F-01.08 | MEDIUM | `emp_years` guard hand-off for NB02 | OPEN (docs) |

**Summary:** 1 CRITICAL, 5 HIGH, 2 MEDIUM. 3 RESOLVED, 5 OPEN (all scope or docs).

### NB02 — Model Development (7 findings)

| ID | Severity | Finding | Status |
|---|---|---|---|
| F-02.01 | CRITICAL | 9 leakage features removed | RESOLVED |
| F-02.02 | HIGH | Walk-forward split not implemented | OPEN (rem) |
| F-02.04 | MEDIUM | Poor calibration in XGBoost | OPEN (rem) |
| F-02.05 | MEDIUM | Suspicious HL in 3 models | OPEN (rem) |
| F-02.06 | MEDIUM | Low precision (0.029) | OPEN (rem) |
| F-02.07 | MEDIUM | Per-state DI below 0.80 | OPEN (docs) |
| F-02.08 | HIGH | IFRS 9 staging misalignment | OPEN (scope) |

**Summary:** 1 CRITICAL, 2 HIGH, 4 MEDIUM. 1 RESOLVED, 6 OPEN.

**On F-02.03.** The `emp_years` guard (F-02.03) is **not present** — the guard fired OK, so no finding was created. This is the "conditional finding" discipline: a finding for something that did not happen would be noise.

**On F-02.01 and F-01.01.** These are the same finding at two stages: detection (NB01) and remediation (NB02). The register keeps both because each notebook's findings are its own record.

### NB03 — Independent Validation (11 findings)

| ID | Severity | Finding | Status |
|---|---|---|---|
| F-03.01 | CRITICAL | 9 leakage features removed (verification) | RESOLVED |
| F-03.02 | HIGH | Walk-forward split not implemented | OPEN (rem) |
| F-03.03 | CRITICAL | Severe disparate impact (DI=0.5414) | OPEN (rem, **blocking**) |
| F-03.04 | HIGH | Poor calibration in XGBoost | OPEN (rem) |
| F-03.05 | MEDIUM | Suspicious HL in 3 models | OPEN (rem) |
| F-03.06 | HIGH | Weak discrimination (KS=0.2671) | OPEN (rem) |
| F-03.07 | HIGH | Counterintuitive SHAP (4 features) | OPEN (rem) |
| F-03.08 | HIGH | No macro indicators | OPEN (scope) |
| F-03.09 | HIGH | U.S. data only | OPEN (scope) |
| F-03.10 | HIGH | IFRS 9 staging misalignment | OPEN (scope) |
| F-03.11 | MEDIUM | Low precision (0.029) | OPEN (rem) |

**Summary:** 2 CRITICAL, 7 HIGH, 2 MEDIUM. 1 RESOLVED, 10 OPEN.

**The verdict-driving finding.** F-03.03 (severe disparate impact) is the finding that makes the verdict REJECT, not APPROVE WITH CONDITIONS. Without it, the risk score would be 3/10 → MODERATE → APPROVE WITH CONDITIONS. One finding changes the verdict.

**On the duplication pattern.** F-03.01 is a verification of F-02.01 (both say 9 leakage features removed). F-03.02 is the same issue as F-02.02 (split method). F-03.10 is the same issue as F-01.06 and F-02.08 (IFRS 9). The register deliberately keeps the duplicates — each notebook's findings are its own record, and the validation confirms them independently.

---

## 5. Findings by severity

| Severity | Count | Findings |
|---|---|---|
| CRITICAL | 4 | F-01.01, F-02.01, F-03.01, F-03.03 |
| HIGH | 14 | F-01.02–F-01.06, F-02.02, F-02.08, F-03.02, F-03.04, F-03.06–F-03.10 |
| MEDIUM | 8 | F-01.07, F-01.08, F-02.04–F-02.07, F-03.05, F-03.11 |
| LOW | 0 | — |

### Distinct issues by severity

The counts above count each notebook's finding separately. The table below counts each **distinct issue** once, mapping it to the findings that record it across notebooks.

**CRITICAL — 2 distinct issues:**

| Distinct issue | Findings | Status |
|---|---|---|
| Data leakage (9 features) | F-01.01 (detect), F-02.01 (remove), F-03.01 (verify) | RESOLVED |
| Severe disparate impact (DI 0.5414) | F-03.03 | **OPEN (blocking)** |

**HIGH — 8 distinct issues:**

| Distinct issue | Findings | Status |
|---|---|---|
| Temporal audit documentation gap | F-01.02 | RESOLVED |
| Feature inventory documentation gap | F-01.03 | RESOLVED |
| No macroeconomic factors | F-01.04, F-03.08 | OPEN (scope) |
| Foreign data applicability | F-01.05, F-03.09 | OPEN (scope) |
| IFRS 9 staging misalignment | F-01.06, F-02.08, F-03.10 | OPEN (scope) |
| Walk-forward split not implemented | F-02.02, F-03.02 | OPEN (rem) |
| Poor calibration (XGBoost) | F-02.04, F-03.04 | OPEN (rem) |
| Weak discrimination (KS 0.2671) | F-03.06 | OPEN (rem) |
| Counterintuitive SHAP (4 features) | F-03.07 | OPEN (rem) |

**MEDIUM — 6 distinct issues:**

| Distinct issue | Findings | Status |
|---|---|---|
| High-missing features (>20%) | F-01.07 | OPEN (docs) |
| `emp_years` guard hand-off | F-01.08 | OPEN (docs) |
| Suspicious HL degeneracy (3 models) | F-02.05, F-03.05 | OPEN (rem) |
| Low precision (0.029) | F-02.06, F-03.11 | OPEN (rem) |
| Per-state DI (superseded) | F-02.07 | OPEN (docs) — superseded by F-03.03 |

**On the distinct-issue table.** The counts in §5's first table are raw counts (each notebook's finding counted separately). The distinct-issue table shows that the "4 CRITICAL" is really **2 distinct critical issues**, one of which is resolved and one of which is blocking. The "14 HIGH" is really **8 distinct high issues**, most of which are scope statements or documentation findings. The "8 MEDIUM" is really **6 distinct medium issues**, of which two are documentation-only.

**The distinct-issue table is the register's most important view for a portfolio reader.** A reviewer who reads only the raw counts thinks the model has 4 CRITICAL problems; the distinct-issue table shows it has 2, of which one is resolved and one is blocking. The register does not minimize the raw counts — it *contextualizes* them.

---

## 6. Findings by status

| Status | Count | Notes |
|---|---|---|
| RESOLVED | 5 | F-01.01, F-01.02, F-01.03, F-02.01, F-03.01 |
| OPEN (rem) | 8 | F-02.02, F-02.04, F-02.05, F-02.06, F-03.02, F-03.04, F-03.05, F-03.06, F-03.07, F-03.11 |
| OPEN (docs) | 4 | F-01.07, F-01.08, F-02.07 |
| OPEN (scope) | 9 | F-01.04, F-01.05, F-01.06, F-02.08, F-03.08, F-03.09, F-03.10 |
| IN PROGRESS | 0 | The pipeline is complete; remediation has not started |
| DEFERRED | 0 | All findings are actionable (or scoped) |

**Note on the counts.** The counts above sum to 26 findings. The "OPEN (rem)" list is shown in full because those are the ones the Remediation Plan tracks; the "(rem)" list includes F-03.07 (2-week timeline), which is the shortest of the remediation actions.

### What "RESOLVED" means

The "RESOLVED" status means different things for different findings, and a reader who assumes "RESOLVED" means "the underlying issue is fixed" will misread the register. There are two kinds of RESOLVED:

- **RESOLVED (issue)** — the underlying issue is closed. F-01.01, F-02.01, F-03.01 (the leakage chain) are RESOLVED (issue): the leakage has been removed and verified. The model's performance now reflects the leakage-free feature set.
- **RESOLVED (documentation)** — the documentation gap is closed, but the underlying issue may still exist. F-01.02 and F-01.03 are RESOLVED (documentation): the temporal audit and feature inventory now exist, but the *underlying data quality concerns* (e.g., the structural missingness in joint-application fields) are documented as separate findings (LC-DS-001 §5, LC-FDL-001 §4).

**No finding in this register is marked RESOLVED (documentation) when there is an unresolved issue-level finding attached to it.** A reader can trust that "RESOLVED" means one of the two senses above, and the finding's description makes clear which.

### What "OPEN (scope)" means

The "(scope)" qualifier marks findings that are **permanently OPEN until the scope changes**. F-01.04 (no macro factors), F-01.05 (U.S. data), and F-01.06 (IFRS 9) are scope statements — the model does not use macro data, is trained on U.S. data, and uses a 16+ day threshold. These are not defects to be remediated; they are scope restrictions that the model acknowledges. A scope finding is "closed" only by a scope change (e.g., acquiring Canadian data, or recalibrating to 30+ days), which is a future project, not a remediation.

**The distinction between OPEN (rem) and OPEN (scope) is material for the Remediation Plan.** An OPEN (rem) finding has an action, an owner, and a due date in MAP-RP-001. An OPEN (scope) finding has a monitoring owner and a "next review" date, but no remediation action because the scope cannot be changed within the plan's horizon. Mixing the two in a single "OPEN" count overstates the remediation workload and understates the scope restrictions.

---

## 7. Cross-register reconciliation

The register's most important structural feature is the cross-reference reconciliation. Every finding that appears in more than one notebook is cross-referenced:

| Issue | NB01 | NB02 | NB03 | Chain |
|---|---|---|---|---|
| Leakage | F-01.01 (detect) | F-02.01 (remove) | F-03.01 (verify) | Detect → Remove → Verify |
| Split method | — | F-02.02 (disclose) | F-03.02 (re-finding) | Disclose → Re-finding |
| IFRS 9 | F-01.06 (scope) | F-02.08 (carry) | F-03.10 (re-finding) | Scope → Carry → Re-finding |
| Macro factors | F-01.04 (scope) | — | F-03.08 (re-finding) | Scope → Re-finding |
| U.S. data | F-01.05 (scope) | — | F-03.09 (re-finding) | Scope → Re-finding |
| Poor calibration | — | F-02.04 (XGBoost) | F-03.04 (XGBoost) | Re-finding |
| HL degeneracy | — | F-02.05 (3 models) | F-03.05 (3 models) | Re-finding |
| Low precision | — | F-02.06 (min) | F-03.11 (min) | Re-finding |
| Per-state DI | — | F-02.07 (diagnostic) | — | Superseded by F-03.03 |

**The chain is the register's most important narrative.** It shows that NB03's findings are not new discoveries — they are verifications of NB01's and NB02's findings. The validator is not finding new problems; the validator is confirming the developer's self-reported findings, which is the whole point of independent validation.

**The exceptions are F-03.03 (fairness) and F-03.06 (KS), F-03.07 (SHAP).** These are not re-findings of any NB01/NB02 finding — they are findings that only NB03's validation could have surfaced. **These are the validation's unique contributions.** The register makes this visible by contrast: most rows in the reconciliation table show a chain, and three do not.

---

## 8. Open items and dependencies

### 8.1 — Open findings by category

The 21 OPEN findings break down into three categories, and the counts matter:

| Category | Count | Findings | Tracking |
|---|---|---|---|
| **OPEN (rem)** — remediation action tracked in MAP-RP-001 | 8 | F-02.02, F-02.04, F-02.05, F-02.06, F-03.02, F-03.04, F-03.05, F-03.06, F-03.07, F-03.11 | MAP-RP-001 (LC-R-01 through LC-R-08) |
| **OPEN (docs)** — documentation-closed, no remediation action | 4 | F-01.07, F-01.08, F-02.07 | Closed by documenting the decision in LC-FDL-001 or the Validation Report |
| **OPEN (scope)** — permanently OPEN until scope changes | 9 | F-01.04, F-01.05, F-01.06, F-02.08, F-03.08, F-03.09, F-03.10 | Monitoring owner; no remediation action in the current plan |

**On the count.** The "21 OPEN" figure is the sum. Only 8 are remediable within the plan's horizon; 4 are documentation findings (closed by writing the documentation); 9 are scope statements (closed only by a scope change). A reader who sees "21 OPEN" might infer 21 remediation actions; the three-category breakdown shows the actual remediation workload is 8.

### 8.2 — The blocking finding

**F-03.03 (severe disparate impact, DI 0.5414) is the finding that blocks deployment.** It is OPEN (rem), tracked as LC-R-01 in MAP-RP-001, with a 3-month timeline. Until it is closed, the model is REJECTed regardless of any other finding's status.

### 8.3 — Open items for the register itself

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **Cross-reference maintenance** | If any finding is renumbered or re-severity-ed, the cross-references must be updated. | MINOR version bump |
| 2 | **V1.0 baseline comparison** | No V1.0 baseline exists, so the register has no "delta from V1.0" view. | If a baseline is authored, a delta column would be added — MINOR version bump |

### 8.4 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | NB01, NB02, NB03 findings registers (source files) |
| Downstream | MAP-RP-001 (references the finding IDs); MAP-VR-LC-001 (reports against the findings) |
| Reproducibility | The consolidated register's hash is recorded in the NB03 audit log |

---

*End of document.*
