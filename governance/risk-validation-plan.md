---
Artifact ID: LC-A06-RVP
Version: 1.2
Run ID: 20261119_141800
Document Title: Risk and Validation Plan (LC-RVP-001)
Project: Lending Club Consumer Credit Risk Model Validation
---

# Risk & Validation Plan — Lending Club Credit Default Model

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Fix, in advance, what the validation will test, how success will be judged, and who decides |
| **Scope** | Independent validation of the Lending Club model (NB02) — performance, stability, drift, fairness, explainability, stress testing, sensitivity, and governance |
| **Prescribed fairness test** | FD-05 4/5ths rule at region level (lowest-cardinality attribute) |
| **Prescribed discrimination floor** | $\text{AUC} \ge 0.65$, $\text{KS} \ge 0.30$ (DCR §1) |
| **Prescribed calibration floor** | HL $p > 0.05$ for $\ge 3$ of 4 models (DCR §2) |
| **Prescribed stability floor** | $\text{PSI} < 0.15$ on a time-based split (DCR §4) |
| **Authority** | MRC Chair is the only role with approval authority; validator's verdict is authoritative until MRC override |
| **Where results live** | `MAP-VR-LC-001` (Validation Report) and `LC-DCR-001` — not in this plan |

> **On the relationship between this plan and the Validation Report:** This plan specifies what the validation will test and what success looks like. The Validation Report (`MAP-VR-LC-001`) reports what the validation found. Measured run values live in the Validation Report, not in this plan, to preserve strict pre-registration integrity.

---

## 1. Purpose, scope, and core validation questions

This plan serves as the binding contract between the model developer and the Model Risk Committee (MRC). It defines what will be tested, what evidence will be produced, who holds sign-off authority, and what happens if the model is deemed unfit.

* **In scope:** Independent validation of the Lending Club model as trained (NB02), covering the seven core validation dimensions and governance compliance.
* **Out of scope:** Production monitoring (post-deployment), IFRS 9 provisioning use, Canadian portfolio applicability, direct protected-attribute fairness testing (no data), and macroeconomic stress testing (no macro data).

### The Seven Core Validation Questions
The technical validation activities in this plan are structured to answer seven foundational model risk questions:

1. **What is the performance?** — Does the model achieve sufficient discrimination and ranking sharpness for collections prioritization?
2. **What is the stability?** — Does the model maintain consistent population and score distributions across time?
3. **What is the drift?** — Are predicted probabilities well-calibrated, and how are feature/prediction shifts tracked?
4. **What is the fairness?** — Does the model produce disparate impact across protected-attribute proxies?
5. **What is the explainability?** — Are the model's drivers transparent and aligned with credit-risk theory?
6. **What is the stress testing?** — How does the model behave under adverse macroeconomic scenarios?
7. **What is the sensitivity analysis?** — How sensitive are model outcomes and fairness metrics to operating thresholds and proxy definitions?

---

## 2. Addressing the seven core validation dimensions

### 2.1 Performance (Question 1)
* **What is tested:** Discrimination power and ranking sharpness on the held-out validation set (NB03).
* **Prescribed method:** Independent recomputation of ROC-AUC, PR-AUC, KS statistic (derived from the ROC curve), Brier score, Log Loss, Precision, Recall, and F1 score directly from `predictions.pkl` (reconciled against NB02 within $1e-6$).
* **Prescribed criteria:** $\text{AUC-ROC} \ge 0.65$; $\text{KS} \ge 0.30$ (moderate separation) / $\ge 0.40$ (strong separation); precision/recall reviewed against operational collections cost at the operating point.

### 2.2 Stability (Question 2)
* **What is tested:** Population and score stability across temporal vintages (NB03).
* **Prescribed method:** Population Stability Index (PSI) computed across a time-based walk-forward split.
* **Prescribed criterion & scope exception:** $\text{PSI} < 0.15$ (DCR §4). **Documented Exception:** Because `issue_month` was removed to eliminate temporal leakage (LC-DS-001 §6), a random stratified 80/20 split is used. Consequently, PSI is explicitly recorded as `not_assessed` (rather than falsely reported as "STABLE"), carrying a formal model risk rating limitation (`F-03.02` / `LC-R-02`).

### 2.3 Drift & Calibration (Question 3)
* **What is tested:** Probability reliability (calibration drift) and ongoing post-deployment monitoring readiness (NB03).
* **Prescribed method:** Hosmer-Lemeshow (HL) goodness-of-fit test (10 percentile bins, $\text{df} = 8$) paired with logistic calibration slope and intercept analysis ($y$ regressed on $y_{\text{proba}}$), plus an explicit **degeneracy check** (flagging HL $p$-values equal to exactly $0.0$ or $1.0$ as numerical artifacts rather than valid calibration results).
* **Prescribed criterion:** HL $p > 0.05$ for $\ge 3$ of 4 candidate models (DCR §2), alongside post-deployment feature/score drift monitoring requirements.

### 2.4 Fairness (Question 4)
* **What is tested:** Disparate impact across demographic/geographic proxy groups under ECOA principles (NB03).
* **Prescribed method:** FD-05 4/5ths rule applied to the lowest-cardinality plausible proxy available (`region` — 4 Census regions, satisfying the $\le 50$ group cardinality cap), excluding groups with $N < 30$ (`FD-03`). Disparate Impact is computed as $\text{DI} = \min(\text{Approval Rate}) / \max(\text{Approval Rate})$ at threshold $0.5$.
* **Prescribed criteria:** $\text{DI} \ge 0.80$ (`OK`), $0.60\text{–}0.80$ (`MODERATE`), $< 0.60$ (`SEVERE` — deployment blocked per DCR §3 FD-01).

### 2.5 Explainability (Question 5)
* **What is tested:** Feature attribution transparency and alignment with economic/credit-risk theory (NB03).
* **Prescribed method:** TreeExplainer (SHAP) computed on a 100-row sample to extract top-10 feature contributions, followed by a formal monotonicity and directionality check against domain expectations.
* **Prescribed criterion:** Top-10 SHAP features must exhibit directional consistency with credit-risk theory (e.g., higher DTI or delinquency increasing predicted default risk); counterintuitive directions are logged as formal findings (`monotonicity_check.csv`).

### 2.6 Stress Testing (Question 6)
* **What is tested:** Model resilience under adverse macroeconomic regimes (e.g., unemployment spikes or interest rate shocks).
* **Prescribed status (Scope Exclusion):** **Excluded / Out of Scope** (`F-03.08`). The public Lending Club snapshot contains no macroeconomic variables or multi-cycle stress scenarios. Procyclical blindness is documented as an inherent limitation (`R-07`) requiring external macro overlay or out-of-time multi-cycle data prior to production reliance.

### 2.7 Sensitivity Analysis (Question 7)
* **What is tested:** Sensitivity of classification performance, precision/recall trade-offs, and disparate impact ratios to threshold selection and underwriting proxies (`grade`/`sub_grade`).
* **Prescribed method:** Comparative evaluation of performance metrics at both the default $0.5$ cutoff and optimal Youden threshold, alongside feature-ablation comparisons with and without proprietary platform grades (`LC-FDL-001` §4).
* **Prescribed status / Limitation:** Operating-point sensitivity of Disparate Impact (testing DI across thresholds other than $0.5$) is documented as a known validation-process limitation (`VL-5`) and queued as a future-version enhancement.

---

## 3. Summary of validation scope and exclusions

### 3.1 What is validated
| Dimension | Scope | Notebook |
|---|---|---|
| Data quality | Missingness, leakage, duplicates, quality score | NB01 |
| Model construction | Four model classes, frozen hyperparameters, split method | NB02 |
| Performance (Q1) | AUC, KS, precision, recall, PR-AUC on held-out set | NB03 |
| Stability (Q2) | PSI attempt (expected `not_assessed` due to random split) | NB03 |
| Drift / Calibration (Q3) | HL test + slope/intercept + degeneracy check | NB03 |
| Fairness (Q4) | FD-05 4/5ths rule at region level | NB03 |
| Explainability (Q5) | SHAP top-10 + monotonicity check | NB03 |
| Cross-population | German Credit direction-only generalization check | NB05 |

### 3.2 What is not validated
| Item | Reason | Documented as |
|---|---|---|
| Production scoring | No deployed service exists | Out of scope |
| IFRS 9 provisioning | Target is 16+ days, not 30+ days | `F-03.10` (scope restriction) |
| Canadian applicability | U.S. data only | `F-03.09` (limitation) |
| Direct race/gender fairness | No protected attributes in dataset | `LC-DS-001` §8 L3 |
| Macroeconomic stress (Q6) | No macro factors in dataset | `F-03.08` (limitation) |
| Time-based stability (Q2) | No usable date column after leakage removal | `F-03.02` (split exception) |

---

## 4. Risk assessment

### 4.1 Inherent model risks
| Risk ID | Inherent risk | Why it matters | How the validation detects it |
|---|---|---|---|
| **R-01** | Leakage contamination | Post-origination features artificially inflate performance | Pre-registered leakage list; reconciliation assertion; AUC drop measurement |
| **R-02** | Severe disparate impact | Region-level $\text{DI} < 0.60$ blocks deployment on fairness grounds | FD-05 4/5ths rule at region level |
| **R-03** | Poor calibration | HL $p \le 0.05$ produces misleading default probabilities | HL test + degeneracy check; slope/intercept |
| **R-04** | Methodological deviation | Random split instead of walk-forward prevents PSI assessment | Disclosed in `split_disclosure`; tracked as `LC-R-02` |
| **R-05** | Weak discrimination | $\text{KS} < 0.30$ limits ranking sharpness for collections | KS statistic from ROC curve |
| **R-06** | Counterintuitive SHAP | Features opposite to credit theory indicate spurious patterns | SHAP top-10 monotonicity check |
| **R-07** | No macro factors | Procyclical blindness (no stress testing) | Documented limitation (`F-03.08`) |
| **R-08** | U.S. data only | Jurisdictional applicability risk | Documented limitation (`F-03.09`) |
| **R-09** | IFRS 9 misalignment | 16+ vs. 30+ days delinquency threshold | Scope restriction (`F-03.10`) |
| **R-10** | Low operating precision | High false-positive rate in collections outreach | Precision/recall evaluation at operating threshold |

### 4.2 Validation process risks
| Risk ID | Validation risk | Impact on the validation | Mitigation |
|---|---|---|---|
| **VR-1** | Fairness test applied at wrong cardinality | DI degenerates if computed per-state one-hot | FD-05 cardinality constraint ($\le 50$ groups) enforced |
| **VR-2** | HL degeneracy undetected | Degenerate $p$-values ($0.0$ or $1.0$) misread as results | Degeneracy check enforced before scoring |
| **VR-3** | PSI reported as STABLE on random split | Misleading stability claim | PSI forced to `not_assessed` on non-temporal splits |
| **VR-4** | Verdict authored outside ownership chain | Unauditable or self-approved verdict | `authored_by == "NB03"` assertion in NB04/NB05 |
| **VR-5** | Reconciliation drift (NB02 vs. NB03) | Validator metrics diverge from developer metrics | $1e-6$ reconciliation assertion |

### 4.3 Residual risk acceptance conditions
| Risk ID | Residual risk acceptance condition | Tracking |
|---|---|---|
| **R-01** | All nine pre-registered leakage features removed; AUC drop disclosed | `LC-R-01` in `MAP-RP-001` |
| **R-02** | Region-level $\text{DI} \ge 0.80$ on held-out population at FD-05 cardinality | `LC-R-01` in `MAP-RP-001` |
| **R-03** | Poorly calibrated models recalibrated (Platt/isotonic) or replaced; HL degeneracy resolved | `LC-R-03` in `MAP-RP-001` |
| **R-04** | Either MD-001 §2 amended for snapshot data, or walk-forward run on dated data | `LC-R-02` in `MAP-RP-001` |
| **R-05** | $\text{KS} \ge 0.30$ achieved, or weaker KS accepted with MRC business rationale | `LC-R-04` in `MAP-RP-001` |
| **R-06** | Counterintuitive SHAP features investigated and re-engineered or justified | `LC-R-04` in `MAP-RP-001` |
| **R-07–R-09** | Macro, Canadian, and IFRS 9 exclusions accepted as formal scope restrictions | `LC-R-05` in `MAP-RP-001` |
| **R-10** | Operating threshold calibrated against collections business cost | Part of `LC-R-03` |

---

## 5. Validation governance & verdict-ownership chain

### 5.1 Verdict-ownership chain
```text
NB02 (Developer)  ──▶ writes performance_summary_v2.json WITHOUT verdict
       │
       ▼
NB03 (Validator)  ──▶ asserts no verdict, writes verdict + authored_by="NB03"
       │
       ├──▶ NB04 (Governance)       ──▶ asserts authored_by="NB03", inherits verdict
       └──▶ NB05 (Cross-validation) ──▶ asserts authored_by="NB03", reads context
