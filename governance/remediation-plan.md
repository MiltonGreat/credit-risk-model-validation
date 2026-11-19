# Recommended Remediation Plan: Credit Risk Portfolio Model

* **Finding Reference:** `F-01` (Leakage-free out-of-time AUC = `0.631` vs. `AP-01` tolerance of $\ge 0.70$ and developer claim of `0.865`)
* **Remediation ID:** `R-01`
* **Target Timeline:** `60 Days`
* **Primary Owner:** Model Owner / Lead Quantitative Developer
* **Second-Line Oversight:** Model Risk Management (MRM) / Escalation `ES-01`
* **Status:** Open

---

## 1. Objective

Remediate Finding `F-01` by establishing strict application-time feature boundaries, eliminating post-origination and target leakage, evaluating multi-period out-of-time (OOT) stability, and establishing a clear governance decision gate (re-validation vs. model retirement) within 60 days.

---

## 2. Phased Action Plan

### Phase 1: Feature Scope Lock & Pipeline Isolation (Days 1–15)
* **Freeze Feature Dictionary:** Document and lock an authoritative application-time feature list and explicitly exclude post-origination and outcome columns (such as `balance`, `paid_total`, `paid_principal`, `paid_interest`, `paid_late_fees`, `loan_status`, `Risk`, and `target`).
* **Enforce Automated Guardrails:** Add pre-flight schema and column-name assertions prior to train/test splitting and model fitting so excluded or post-origination fields cannot enter training matrices.
* **Deliverable:** Signed-off Leakage-Free Feature Manifest and automated validation checks.

### Phase 2: Model Re-Estimation & Calibration (Days 16–35)
* **Re-State Performance:** Re-train candidate models (`Logistic Regression`, `XGBoost`, `Random Forest`, `LightGBM`) strictly on the leakage-free feature set and document the target definition unambiguously.
* **Evaluate Calibration:** Assess probability calibration (e.g., Isotonic Regression or Platt Scaling) separately from rank-ordering (`AUC`) so predicted probabilities align with observed default frequencies rather than relying solely on uncalibrated class-weighted scores.
* **Deliverable:** Leakage-free re-training results and calibration diagnostic summary.

### Phase 3: Multi-Period Out-of-Time (OOT) & Sensitivity Validation (Days 36–50)
* **Second OOT Period:** Validate performance across a second independent out-of-time vintage window (beyond the initial 2016–2018 period) to test temporal stability and reduce single-period uncertainty.
* **Sensitivity & Alternative Checks:** Re-run sensitivity checks (including with and without `grade`/`sub_grade` and cross-dataset checks such as German Credit under Run ID `20261118_170144`) to separate genuine risk signal from grading or target-definition artifacts.
* **Deliverable:** Independent Second-Line OOT & Sensitivity Validation Report.

### Phase 4: Governance Review & Final Disposition (Days 51–60)
* **Committee Review:** Submit the re-stated metrics, multi-period OOT results, and calibration findings to Model Risk Management / the Model Risk Committee.
* **Decision Gate:** Issue formal sign-off if exit criteria are met, or execute documented model retirement / fallback procedures under escalation `ES-01`.
* **Deliverable:** Final Validation Report and formal governance sign-off or retirement record.

---

## 3. Success Criteria & Exit Conditions

| Criterion | Requirement |
| :--- | :--- |
| **Primary Benchmark (`AP-01`)** | Leakage-free out-of-time `AUC` $\ge 0.70$ across the out-of-time evaluation(s), verified in the validation report. |
| **Calibration & Stability** | Documented calibration check and multi-period OOT assessment with no unresolved leakage or target-definition ambiguity. |
| **Retirement / Sunset Fallback** | If leakage-free `AUC` remains below `0.70` or remediation milestones are not met within 60 days, execute a documented decision to retire the model and retain/fallback to approved baseline controls under `ES-01`. |

---

## 4. Verification Checklist

* [ ] Confirm all training scripts enforce the frozen leakage-free feature list and log exact Run IDs.
* [ ] Verify that re-stated metrics and second-period OOT results are reproducible from archived outputs and independent checks.
* [ ] Record final sign-off or retirement decision in the validation report and update `R-01` / `F-01` status.
