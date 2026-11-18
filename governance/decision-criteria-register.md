---
Artifact ID: LC-A07-DCR
Version: 2.2
Run ID: 20261119_143800
Document Title: Decision Criteria Register (LC-DCR-001)
Project: Lending Club Consumer Credit Risk Model Validation
---

# Decision Criteria Register (A07)

This document pre-registers the thresholds that determine pass, investigate, and fail across all validation dimensions[cite: 2].

---

## 1. Purpose and scope of the register

This register records every decision criterion used in the Lending Club pipeline: the thresholds for AUC, KS, DI, HL, PSI, SHAP direction, and the weights in the risk score[cite: 2]. It specifies who set each threshold, why, and who has authority to change it[cite: 2].

* **Distinction from the RVP:** The RVP specifies *what* will be tested; the DCR specifies *what* the thresholds are and where they come from[cite: 2].
* **Distinction from the Validation Report:** The Validation Report reports *what* the validation found; the DCR defines what "found" means against the pre-registered criteria[cite: 2].
* **Distinction from the Problem Definition Record (PDR):** The PDR fixes the high-level success criteria before the model is built; the DCR establishes the full technical catalogue of thresholds, bands, and scoring points[cite: 2].

---

## 2. Core decision framework
### 2.1 What constitutes acceptable performance?
Acceptable performance requires meeting pre-registered minimum floors designed for collections prioritization:
* **Discrimination:** AUC-ROC $\ge 0.65$ (DCR §1)[cite: 2]; Kolmogorov-Smirnov (KS) statistic $\ge 0.30$ (strong separation at $\ge 0.40$).
* **Precision / Recall:** Evaluated against business cost at the operating threshold (with feasibility constraints recognizing that at a ~20% default rate, extreme precision/recall targets require higher AUC; metrics are thus evaluated at a stated cutoff).
* **Class Balance & Feasibility:** The default rate baseline is approximately 1.78% to 20% depending on sample scope, requiring specialized metrics over naive accuracy.

### 2.2 What constitutes material drift?
Material drift and stability degradation are captured via score and population shifts:
* **Stability Index (PSI):** $\text{PSI} < 0.15$ is stable; $0.15\text{–}0.25$ indicates moderate drift; $\text{PSI} > 0.25$ constitutes **major drift** (2 points on risk score, DCR §4)[cite: 2]. *(Note: In this snapshot run, PSI is `not_assessed` due to the lack of a usable date column after leakage removal).*
* **Calibration Breakdown:** Hosmer-Lemeshow (HL) $p \le 0.05$ indicates poor calibration drift or misaligned probability estimates, triggering a **HIGH finding** (DCR §2)[cite: 2].

### 2.3 What constitutes unacceptable disparity?
Fairness is evaluated using the 4/5ths rule on the lowest-cardinality geographic proxy (`region`, 4 Census regions, satisfying the $\le 50$ group cardinality cap `FD-05`, with group size minimum $N \ge 30$ `FD-03`)[cite: 2]:
* **$\text{DI} \ge 0.80$ (`OK`):** Acceptable disparity level (0 points)[cite: 2].
* **$0.60 \le \text{DI} < 0.80$ (`MODERATE`):** Elevated disparate impact requiring tracking (1 point)[cite: 2].
* **$\text{DI} < 0.60$ (`SEVERE`):** **Unacceptable disparity**, triggering an automatic CRITICAL finding, 3 risk score points, and blocking deployment (DCR §3 `FD-01`)[cite: 2].

### 2.4 What triggers further investigation?
Further technical investigation and diagnostic review are triggered by:
* **Investigate Performance Bands:** AUC falling in the $0.60\text{–}0.65$ investigate band (1 risk score point)[cite: 2].
* **HL Suspicious P-Values:** HL $p$-values of exactly $0.0$ or $1.0$ (flagged as numerical artifacts requiring implementation verification, `E-M03-06`)[cite: 2].
* **Explainability Anomalies:** $\ge 1$ counterintuitive feature direction under SHAP analysis, triggering a HIGH finding (`F-03.07`)[cite: 2].
* **Operating Precision:** High false-positive rates at the operating threshold, reviewed against business cost (`F-03.11`)[cite: 2].

### 2.5 What constitutes a failed validation?
A validation fails when the composite risk score or critical automated guards breach acceptable safety limits:
* **Risk Score Rating Bands (DCR §9):**
  * **0–2 points (`LOW`):** APPROVE[cite: 2]
  * **3–4 points (`MODERATE`):** APPROVE WITH CONDITIONS[cite: 2]
  * **5–7 points (`HIGH`):** **REJECT** (Hard prohibition on production use)[cite: 2]
  * **8–10 points (`CRITICAL`):** **REJECT**[cite: 2]
* **Failure Condition:** Any run resulting in a **HIGH** or **CRITICAL** rating (such as this run's score of **6/10**, driven by severe regional disparate impact and calibration failure) constitutes a failed validation, resulting in a mandatory **REJECT** verdict[cite: 2].

### 2.6 What requires escalation?
The following triggers mandate immediate escalation to the Model Risk Committee (MRC):
* **CRITICAL or HIGH Findings:** Any automated guard failure (`LC-G1` through `LC-G6`), such as leakage presence (`LC-G1`), severe disparate impact (`LC-G2`, $\text{DI} < 0.60$), or weak ranking sharpness (`LC-G4`, $\text{KS} < 0.30$)[cite: 2].
* **Remediation Delays:** Remediation tasks falling behind schedule (escalation mandated at 50% of timeline)[cite: 2].
* **Post-Validation Discovery:** Any newly discovered CRITICAL finding post-validation, requiring a full re-validation before any consideration of deployment[cite: 2].

---

## 3. Criterion catalogue — thresholds and bands

### 3.1 Performance criteria
| Criterion | Band / Threshold | Risk Score Points / Action | Source |
|---|---|---|---|
| **AUC-ROC** | $\ge 0.65$ | 0 points (Acceptable) | DCR §1[cite: 2] |
| **AUC-ROC** | $0.60\text{–}0.65$ | 1 point (Investigate) | DCR §1[cite: 2] |
| **AUC-ROC** | $0.55\text{–}0.60$ | 2 points | DCR §1[cite: 2] |
| **AUC-ROC** | $< 0.55$ | 3 points (Unacceptable floor) | DCR §1[cite: 2] |
| **KS Statistic** | $\ge 0.40$ | 0 points (Strong separation) | DCR §1[cite: 2] |
| **KS Statistic** | $0.30\text{–}0.40$ | 1 point (Moderate separation) | DCR §1[cite: 2] |
| **KS Statistic** | $< 0.30$ | HIGH finding (`F-03.06`) | DCR §1[cite: 2] |
| **Precision** | Reviewed against business cost | No fixed threshold (Informed review) | DCR §1[cite: 2] |

### 3.2 Calibration criteria
| Criterion | Band / Threshold | Action / Points | Source |
|---|---|---|---|
| **HL $p$-value** | $> 0.05$ | 0 points (`GOOD`) | DCR §2[cite: 2] |
| **HL $p$-value** | $\le 0.05$ | 2 points (`POOR` finding, `F-03.04`) | DCR §2[cite: 2] |
| **HL $p$-value** | Exactly $0.0$ or $1.0$ | `SUSPICIOUS` — Flag as artifact (`E-M03-06`) | DCR §2[cite: 2] |
| **Slope / Intercept** | Slope $\sim 1.0$, Intercept $\sim 0.0$ | Diagnostic context (Unscored) | DCR §2[cite: 2] |

### 3.3 Fairness criteria (`FD-01`, `FD-03`, `FD-05`)
| Rule / Metric | Threshold / Condition | Category / Action | Source |
|---|---|---|---|
| **Disparate Impact ($\text{DI}$)** | $\text{DI} \ge 0.80$ | `OK` (0 points) | DCR §3 `FD-01`[cite: 2] |
| **Disparate Impact ($\text{DI}$)** | $0.60 \le \text{DI} < 0.80$ | `MODERATE` (1 point) | DCR §3 `FD-01`[cite: 2] |
| **Disparate Impact ($\text{DI}$)** | $\text{DI} < 0.60$ | `SEVERE` (3 points, CRITICAL finding `F-03.03`) | DCR §3 `FD-01`[cite: 2] |
| **Group Size ($FD-03$)** | $N \ge 30$ minimum per group | Groups with $N < 30$ excluded from DI calculation | DCR §3 `FD-03`[cite: 2] |
| **Cardinality ($FD-05$)** | $\le 50$ total groups | Attributes above 50 groups invalid for 4/5ths test | DCR §3 `FD-05`[cite: 2] |

### 3.4 Stability criteria
| Criterion | Band / Threshold | Action / Points | Source |
|---|---|---|---|
| **PSI** | $< 0.15$ | 0 points (`STABLE`) | DCR §4[cite: 2] |
| **PSI** | $0.15\text{–}0.25$ | 1 point (`MODERATE`) | DCR §4[cite: 2] |
| **PSI** | $> 0.25$ | 2 points (`MAJOR`) | DCR §4[cite: 2] |
| **PSI** | `not_assessed` | 0 points (No penalty for data limitation) | DCR §4[cite: 2] |

### 3.5 Explainability criteria
| Criterion | Threshold / Condition | Action | Source |
|---|---|---|---|
| **SHAP Direction** | All top-10 features consistent | No finding | DCR §5[cite: 2] |
| **Counterintuitive** | $\ge 1$ feature opposing credit theory | HIGH finding (`F-03.07`) | DCR §5[cite: 2] |
| **Leakage in Top-20** | Any leakage feature present | CRITICAL finding | DCR §5[cite: 2] |

### 3.6 Governance & risk score components
| Component | Max Points | Source |
|---|---|---|
| **Performance** | 3 | DCR §1[cite: 2] |
| **Stability** | 2 | DCR §4[cite: 2] |
| **Calibration** | 2 | DCR §2[cite: 2] |
| **Fairness** | 3 | DCR §3 `FD-01`[cite: 2] |
| **Leakage** | 3 | DCR §3[cite: 2] |
| **Total Risk Score** | **10** | DCR §9[cite: 2] |

---

## 4. Results against criteria — this run

| Dimension | Criterion | Measured Value | Band / Status | Pass / Fail | Points | Finding |
|---|---|---|---|---|---|---|
| **Performance** | AUC-ROC | 0.631 | $0.60\text{–}0.65$ | FAIL | 1 | —[cite: 2] |
| **Performance** | KS Statistic | 0.2671 | $< 0.30$ | FAIL | 0 | `F-03.06` (HIGH)[cite: 2] |
| **Performance** | Precision | 0.085 | Business cost review | N/A | — | `F-03.11` (MEDIUM)[cite: 2] |
| **Calibration** | HL ($p$) — LightGBM | 0.729 | $> 0.05$ | PASS | 0 | —[cite: 2] |
| **Calibration** | HL ($p$) — XGBoost | 0.000 | $\le 0.05$ | FAIL (`POOR`) | 2 | `F-03.04` (HIGH)[cite: 2] |
| **Calibration** | HL ($p$) — LR & RF | 1.000 | Exactly 1.0 | SUSPICIOUS | — | `F-03.05` (MEDIUM)[cite: 2] |
| **Fairness** | Region DI | 0.5414 | SEVERE ($< 0.60$) | FAIL | 3 | `F-03.03` (CRITICAL)[cite: 2] |
| **Stability** | PSI | `not_assessed` | N/A | N/A | 0 | — (Data limitation)[cite: 2] |
| **Explainability** | SHAP Direction | 6 of 10 consistent | 4 counterintuitive | FAIL | 0 | `F-03.07` (HIGH)[cite: 2] |
| **Governance** | Compliance Score | 74.1 / 100 | Target $\ge 80$ | FAIL | — | See `MAP-VR-LC-001`[cite: 2] |

---

## 5. Verdict record — this run

### 5.1 Risk score components
* **Performance:** 1 / 3 (AUC 0.631, MODERATE band)[cite: 2]
* **Stability:** 0 / 2 (PSI `not_assessed`, no penalty)[cite: 2]
* **Calibration:** 2 / 2 (XGBoost poor, HL $p = 0.000$)[cite: 2]
* **Fairness:** 3 / 3 (DI 0.5414, SEVERE)[cite: 2]
* **Leakage:** 0 / 3 (All 9 leakage features removed)[cite: 2]
* **Total Score:** **6 / 10** (`HIGH` rating $\rightarrow$ **REJECT**)[cite: 2]

### 5.2 Primary driver of the verdict
The rejection is **fairness-driven, not performance-driven**[cite: 2]. AUC 0.631 contributes 1 point; fairness (3 points) and calibration (2 points) dominate the score[cite: 2]. The regional disparate impact finding (`F-03.03`, DI 0.5414) is the single largest contributor and must be remediated before the model can be reconsidered[cite: 2].

---

## 6. Automated threshold guards (`LC-G1` to `LC-G6`)

| Guard ID | Triggers When | Finding Generated | Source |
|---|---|---|---|
| **LC-G1** | Leakage features present in feature matrix | `F-03.01` (CRITICAL) | DCR §3[cite: 2] |
| **LC-G2** | Disparate Impact $\text{DI} < 0.60$ | `F-03.03` (CRITICAL) | DCR §3 `FD-01`[cite: 2] |
| **LC-G3** | HL calibration test $p \le 0.05$ | `F-03.04` (HIGH) | DCR §2[cite: 2] |
| **LC-G4** | KS statistic $< 0.30$ | `F-03.06` (HIGH) | DCR §1[cite: 2] |
| **LC-G5** | $\ge 1$ counterintuitive SHAP direction | `F-03.07` (HIGH) | DCR §5[cite: 2] |
| **LC-G6** | Split method is non-temporal / random | `F-03.02` (HIGH) | DCR §1[cite: 2] |

---

## 7. Change control, versioning, and authority

* **Version:** 2.2 (Minor revision: structural reorganization, DCR guard renaming to `LC-G` convention, REJECT definition added; no threshold values modified)[cite: 2].
* **Authority Chain:** 
  $$\text{Developer (Proposes)} \longrightarrow \text{Validator (Reviews)} \longrightarrow \text{MRC Chair (Approves with Rationale)} $$[cite: 2]
* **Sole Approver:** The Model Risk Committee (MRC) Chair is the sole authority for any threshold modifications[cite: 2]. Any unauthorized threshold change constitutes a major audit finding in subsequent validation cycles[cite: 2].
