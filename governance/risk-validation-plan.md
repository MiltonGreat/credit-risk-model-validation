---
Artifact ID: LC-A06-RVP
Version: 1.2
Run ID: 20261119_141800
Document Title: Risk and Validation Plan (LC-RVP-001)
Project: Lending Club Consumer Credit Risk Model Validation
---

# Risk & Validation Plan — Lending Club Credit Default Model

## At a glance
* **Purpose:** Fix, in advance, what the validation will test, how success will be judged, and who decides[cite: 1].
* **Scope:** Independent validation of the Lending Club model (NB02) — fairness, performance, calibration, stability, governance[cite: 1].
* **Prescribed fairness test:** FD-05 4/5ths rule at region level (lowest-cardinality attribute)[cite: 1].
* **Prescribed discrimination floor:** AUC $\ge 0.65$, KS $\ge 0.30$ (DCR §1)[cite: 1].
* **Prescribed calibration floor:** HL $p > 0.05$ for $\ge 3$ of 4 models (DCR §2)[cite: 1].
* **Prescribed stability floor:** PSI $< 0.15$ on a time-based split (DCR §4)[cite: 1].
* **Authority:** MRC Chair is the only role with approval authority; validator's verdict is authoritative until MRC override[cite: 1].
* **Where results live:** MAP-VR-LC-001 (Validation Report) — not in this plan[cite: 1].

---

## 1. Purpose and scope of the plan
This plan defines how the Lending Club Credit Default Model will be validated: what is in scope, what risks are being tested, what evidence will be produced, who has authority to sign off, and what happens if the validation finds the model unfit[cite: 1].

* **In scope:** Independent validation of the Lending Club model as trained (NB02), fairness testing, calibration assessment, stability assessment, governance review[cite: 1].
* **Out of scope:** Production monitoring (post-deployment), IFRS 9 provisioning use, Canadian portfolio applicability, direct protected-attribute fairness testing (no data), macroeconomic stress testing (no macro data)[cite: 1].

---

## 2. Validation activities
The validation has five activities, ordered by priority[cite: 1]:

| # | Activity | What the validation will test | Prescribed method | Prescribed criterion |
|---|---|---|---|---|
| 1 | Fairness | Whether the model produces disparate impact across region groups[cite: 1] | FD-05 4/5ths rule at region level (lowest-cardinality attribute)[cite: 1] | $\text{DI} \ge 0.80$ (DCR §3 FD-01)[cite: 1] |
| 2 | Performance | Whether the model discriminates well enough for collections prioritization[cite: 1] | AUC, KS, precision, recall, PR-AUC on held-out set[cite: 1] | $\text{AUC} \ge 0.65$, $\text{KS} \ge 0.30$ (DCR §1)[cite: 1] |
| 3 | Calibration | Whether predicted probabilities are reliable for decision-making[cite: 1] | HL test with degeneracy check; slope/intercept[cite: 1] | HL $p > 0.05$ for $\ge 3$ of 4 models (DCR §2)[cite: 1] |
| 4 | Stability | Whether the model will perform consistently on new data[cite: 1] | PSI on a time-based split[cite: 1] | $\text{PSI} < 0.15$ (DCR §4)[cite: 1] |
| 5 | Governance | Whether the model is documented, reproducible, and compliant with regulatory expectations[cite: 1] | OSFI E-23, SR 11-7, ECOA mapping[cite: 1] | Compliance score $\ge 80$ (DCR §9)[cite: 1] |

---

## 3. Validation scope

### 3.1 What is validated
* **Data quality:** Missingness, leakage, duplicates, quality score (NB01)[cite: 1].
* **Model construction:** Four model classes, hyperparameters, split method (NB02)[cite: 1].
* **Performance:** AUC, KS, precision, recall, PR-AUC on held-out set (NB03)[cite: 1].
* **Fairness:** FD-05 4/5ths rule at region level (NB03)[cite: 1].
* **Calibration:** HL test + slope/intercept + degeneracy check (NB03)[cite: 1].
* **Stability:** PSI attempt (expected `not_assessed`) (NB03)[cite: 1].
* **Explainability:** SHAP top-10 + monotonicity check (NB03)[cite: 1].
* **Cross-population:** German Credit direction-only comparison (NB05)[cite: 1].

### 3.2 What is not validated
* **Production scoring:** No deployed service exists (Out of scope)[cite: 1].
* **IFRS 9 provisioning:** Target is 16+ days, not 30+ days (F-03.10 scope restriction)[cite: 1].
* **Canadian applicability:** U.S. data only (F-03.09 limitation)[cite: 1].
* **Direct race/gender fairness:** No protected attributes in dataset (LC-DS-001 §8 L3)[cite: 1].
* **Macroeconomic stress:** No macro factors in dataset (F-03.08 limitation)[cite: 1].
* **Time-based stability:** No usable date column after leakage removal (F-03.02 split exception)[cite: 1].

---

## 4. Risk assessment

### 4.1 Inherent risks
* **R-01:** Leakage contamination (post-origination features inflate performance) — tested via pre-registered leakage list and reconciliation assertion[cite: 1].
* **R-02:** Severe disparate impact (region-level DI below 0.60) — tested via FD-05 4/5ths rule[cite: 1].
* **R-03:** Poor calibration (HL $p \le 0.05$) — tested via HL test + degeneracy check and slope/intercept[cite: 1].
* **R-04:** Methodological deviation (random split, not walk-forward) — disclosed in `split_disclosure`[cite: 1].
* **R-05:** Weak discrimination (KS below 0.30) — tested via KS statistic from ROC curve[cite: 1].
* **R-06:** Counterintuitive SHAP (features opposite to credit theory) — tested via SHAP top-10 monotonicity check[cite: 1].
* **R-07 to R-10:** Macro factors, jurisdictional scope, IFRS 9 misalignment, and operating threshold precision — documented limitations and operational metrics[cite: 1].

### 4.2 Validation risks (Process risks)
* **VR-1:** Fairness test applied at the wrong cardinality — mitigated by enforcing FD-05 cardinality constraint[cite: 1].
* **VR-2:** HL degeneracy undetected — mitigated by enforcing degeneracy check ($p = 0.0$ or $1.0$)[cite: 1].
* **VR-3:** PSI reported as STABLE on a random split — mitigated by reporting `not_assessed` when split is non-temporal[cite: 1].
* **VR-4:** Verdict authored outside the ownership chain — mitigated by `authored_by` assertion[cite: 1].
* **VR-5:** Reconciliation drift between NB02 and NB03 — mitigated by $1e-6$ reconciliation assertion[cite: 1].

---

## 5. Validation governance & Verdict ownership chain

* **The Verdict-Ownership Chain:** 
  $$\text{NB02 (Developer)} \longrightarrow \text{NB03 (Validator writes verdict, authored\_by}=\text{"NB03"}) \longrightarrow \text{NB04/NB05 (Governance inherits)}$$[cite: 1]
* **Decision Authority Matrix:** Only the **MRC Chair** holds ultimate approval, rejection, or override authority (with documented rationale), while the Independent Validator authors the binding validation opinion[cite: 1].

---

## 6. Validation-process limitations (VL-1 to VL-7)
* Highlights constraints including region proxy usage (`VL-1`), absence of PSI on random split (`VL-2`), 100-row SHAP sample (`VL-3`), HL degeneracy (`VL-4`), single-threshold operating point (`VL-5`), lack of V1.0 baseline (`VL-6`), and analyst-pipeline execution structure noting technical vs. organizational independence (`VL-7`)[cite: 1].
