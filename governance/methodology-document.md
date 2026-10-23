
**The critical pipeline property.** Every stage writes a contract that the next stage asserts. NB02 raises if NB01's inventory is missing. NB03 raises if NB02's summary has a verdict (it should not). NB04 raises if NB03's summary was not authored by NB03. **The pipeline fails loudly at the first broken contract — no silent continuation.** This is the methodological commitment that makes the pipeline auditable: a reviewer can trust that a completed run means every contract was satisfied, because a broken contract halts the pipeline rather than producing partial artifacts.

---

## 4. Parameter specification (frozen values)

### Why the hyperparameters are frozen

The hyperparameters are frozen **before the validation is run**, not after. This is a methodological commitment: hyperparameter tuning after seeing validation results would make the validation results optimistic, because the model would have been selected to perform well on the validation set. Freezing the hyperparameters preserves the independence of the validation: a validator reading the MD knows that the hyperparameters were not chosen to match the validation outcome.

Any change to a hyperparameter is a **MAJOR version bump** (not a MINOR one) because it changes the model that is being validated, not just the run that produced it.

### Frozen hyperparameters (MD-001 §3.4)

| Parameter | LR | XGBoost | Random Forest | LightGBM |
|---|---|---|---|---|
| `n_estimators` | — | 100 | 100 | 100 |
| `max_depth` | — | 4 | 6 | 4 |
| `learning_rate` | — | 0.1 | — | default |
| `subsample` | — | 0.8 | — | default |
| `colsample_bytree` | — | 0.8 | — | default |
| `class_weight` / `scale_pos_weight` | balanced dict | 55.34 (ratio) | balanced dict | balanced dict |
| `C` (regularization) | 1.0 | — | — | — |
| `solver` | lbfgs | — | — | — |
| `random_state` | 42 | 42 | 42 | 42 |
| `n_jobs` | default | default | −1 | 1 |

### Run evidence

Cross-validation (StratifiedKFold, n_splits=5, shuffle=True, random_state=42, scoring `roc_auc`) produced: LR 0.662, XGB 0.625, RF 0.653, LGBM 0.621. The optimal Youden thresholds on the held-out set were: LR 0.629, XGB 0.243, RF 0.503, LGBM 0.194. **These are run evidence, not methodology** — a re-run with a different seed or a different dataset would produce different numbers.

**Seed.** `RANDOM_STATE = 42` (AS-001 §8). Changing it invalidates all reproducibility claims. If the seed must be changed, re-run the full pipeline and re-baseline the evidence — do not change the seed and cite the old numbers.

---

## 5. Training and evaluation methodology

### Prescribed split: walk-forward

The prescribed split (MD-001 §2) is **walk-forward**: train on loans issued before a cutoff date, validate on loans issued after. The purpose of walk-forward is to test the model's performance on **out-of-time** data, which is the regime a deployed model actually operates in.

### Run's split: random stratified 80/20 — documented exception

**This run used a random stratified 80/20 split**, not walk-forward. The exception reason: **no usable date column survives after leakage removal** (`issue_month` is removed per LC-DS-001 §6, because it is a temporal-leakage feature). Without a date column, walk-forward cannot be implemented on this dataset.

The exception is recorded in **F-03.02**, disclosed in `models/test_data.pkl` (the `split_disclosure` field) and in `performance_summary_v2.json` (the `split` field), and tracked as **LC-R-02** in MAP-RP-001. It is a **methodological deviation**, not a defect of the run — the run could not have followed the prescribed method on this data. A future version either implements walk-forward on a dataset with a usable date column, or formally amends MD-001 §2 to permit random splits on snapshot datasets.

### Prescribed evaluation metrics

Held-out evaluation reports AUC-ROC, PR-AUC, Brier, log loss, precision, recall, and F1 at both 0.5 and the optimal Youden threshold. The primary comparison metric is **held-out AUC-ROC** (DCR §1 threshold: ≥ 0.65). Secondary metrics are reported for context.

### Run evidence

**Model selected:** Random Forest, by held-out AUC (0.631). **Leakage impact:** developer-claimed AUC 0.865 → realistic 0.631, delta 0.234 (27.1%). This delta is the methodology's most important number — it is what happens when the nine leakage features are removed. It is recorded as **F-03.01** and discussed in **MAP-VR-LC-001 §3**.

**Class imbalance handling.** `class_weight="balanced"` for LR, RF, LGBM; `scale_pos_weight` for XGBoost. **No SMOTE** — SMOTE would synthesize minority-class examples, which is a different modelling assumption (the synthetic examples would not correspond to real loans). The choice preserves the data; the class weights preserve the loss function.

> **Why the delta is in the run-evidence block, not in the methodology.** The methodology prescribes *that* leakage must be removed and *that* the impact must be measured. The specific delta (0.234) is a run result — a re-run on a different dataset would produce a different delta, and the methodology's commitment (leakage removal is non-negotiable; the impact must be disclosed) would remain unchanged.

---

## 6. Fairness and calibration methodology

### Prescribed fairness methodology

**Requirement.** The methodology requires fairness testing on the lowest-cardinality attribute available. The 4/5ths rule (FD-05 of DCR-001) applies to attributes with ≤ 50 groups; applying it to a higher-cardinality attribute (e.g., per-state one-hot) is a **proxy-validity violation**, because the 4/5ths rule's comparison of "one group against the complement of all others" degenerates when the number of groups is large (many groups have zero defaults, so the minimum approval rate is 0, and the DI ratio is 0, which is not evidence of discrimination).

**Procedure.**

- **Attribute:** the lowest-cardinality attribute that is a plausible protected-attribute proxy. On this dataset, region (4 Census regions) is the lowest-cardinality geographic proxy.
- **Group sufficiency:** groups with N < 30 are excluded (FD-03). This prevents small-sample artefacts from dominating the DI.
- **Metric:** DI = min(approval rate) / max(approval rate). Approval rate = fraction of loans with `y_pred = 1` at threshold 0.5.
- **Thresholds:** OK ≥ 0.80, MODERATE 0.60–0.80, SEVERE < 0.60 (DCR §3 FD-01).

### Run's fairness result

Region-level DI = **0.5414 (SEVERE)**. The "Other" region (N = 6) was excluded on FD-03. Recorded in **F-03.03** and discussed in **MAP-VR-LC-001 §4**. This is the **primary driver of the REJECT verdict** — the verdict is fairness-driven, not performance-driven.

> **On the fairness result.** The DI of 0.5414 is a **run result** on the region proxy. It is indicative of a fairness issue that requires remediation; it is **not** a definitive fair-lending finding, because region is a proxy, not a direct protected attribute. The methodology's commitment is that fairness testing must be performed on the lowest-cardinality attribute available, and the result must be disclosed as a proxy finding. Whether the finding is severe enough to reject deployment is a DCR determination (and on this run, it was).

### Prescribed calibration methodology

**Requirement.** Calibration must be assessed, and the assessment must include a **degeneracy check**. The HL test can produce p-values of exactly 0.0 or 1.0 on a non-degenerate distribution — a p-value of 1.0 is a numerical artefact, not evidence of good calibration. A calibration finding based on a degenerate HL p-value is not a finding.

**Procedure.**

- **Test:** Hosmer-Lemeshow, 10 bins via percentile edges; HL statistic compared to chi-squared with df = 8.
- **Degeneracy check:** p exactly 0.0 or 1.0 is flagged as suspicious, not treated as a calibration result.
- **Slope and intercept:** from logistic regression of y on y_proba.

### Run's calibration result

3 of 4 models produce degenerate HL p-values (exactly 0.0 or 1.0). **Only LightGBM has a valid result** (p = 0.729, slope 1.078). XGBoost is flagged as poor calibration (HL p ≤ 0.05). The degenerate p-values are recorded as **F-03.05** and the XGBoost poor calibration as **F-03.04**. Both are discussed in **MAP-VR-LC-001 §4**.

> **On the calibration result.** The MD's commitment is that a calibration finding based on a p-value of exactly 1.0 is **not** a finding. The run's three degenerate p-values are therefore **flagged but not treated as calibration findings** in the risk score. The XGBoost p-value is a valid poor-calibration finding and is treated as such. The degenerate cases are recorded as a **diagnostic issue** — the HL implementation must be verified before they can be converted into findings.

---

## 7. Validation methodology

### Prescribed validation methodology

**Principle.** The validator must recompute all metrics independently from the model's predictions, **not** from the developer's reported metrics. A validator that copies the developer's numbers is not validating — it is attesting to the developer's arithmetic. The independent recompute is what catches developer implementation errors.

**Procedure (NB03).**

1. **Load NB02 artifacts** (models, predictions, test_data, summary) and **assert no verdict is present** (NB02 must not author one).
2. **Recompute metrics from predictions**, not from NB02's CSV. The independent recompute catches NB02 implementation errors.
3. **Recompute realistic AUC** from `y_val` and predictions; reconcile with NB02's reported realistic AUC to 1e-6. A difference greater than 1e-6 is a finding about NB02's implementation.
4. **Compute PSI — only if the split is time-based.** On a random split, PSI measures within-distribution noise, not population stability. A PSI of ~0 on a random split is a tautology, not stability evidence.
5. **SHAP + monotonicity.** `TreeExplainer` on XGBoost, 100-row sample. Check leakage features in top-20; check top-10 for direction consistency against credit-risk theory.
6. **KS statistic** from the ROC curve; thresholds 0.30 (MODERATE) and 0.40 (STRONG).
7. **Fairness at region level.** Same FD-05 test as in §6, but on NB03's own evaluation of predictions (not NB02's).
8. **Risk score.** Sum of five components from DCR §9; verdict bands from DCR §7.
9. **Findings register.** Conditional findings based on the conditions that fired; IDs F-03.01–F-03.11.
10. **Write verdict** to `performance_summary_v2.json` with `authored_by: "NB03"`.

### The verdict-ownership chain

The verdict is owned by NB03 and by no other notebook. The chain is:

- **NB02** asserts **no verdict** is present in the summary it writes.
- **NB03** asserts the summary has no verdict, **writes the verdict**, and stamps `authored_by: "NB03"`.
- **NB04 and NB05** assert `authored_by == "NB03"` before reading the verdict.

**No notebook can short-circuit the chain.** A notebook that writes a verdict outside this chain (e.g., NB04 re-deriving a risk score) is caught by the assertion in the next stage. The chain is the mechanism that makes the Lending Club validation's verdict credible: a reviewer can trace the verdict to a single notebook, and can verify that notebook's authorship via the `authored_by` field.

### Run evidence

- **Reconciliation:** NB03's realistic AUC (0.6307) reconciles with NB02's reported value to within 1e-6.
- **PSI:** `not_assessed` (random split). Not "STABLE."
- **KS:** 0.2671 (Weak) — recorded as F-03.06.
- **Counterintuitive SHAP features:** 4 (`loan_to_income`, `total_collection_amount_ever`, `annual_income`, `total_credit_limit`) — recorded as F-03.07.
- **Verdict:** 6/10 → HIGH → REJECT — recorded in `performance_summary_v2.json` with `authored_by: "NB03"` and in MAP-VR-LC-001.

> **On the run evidence.** Each of the run-evidence items is a **result**, not a methodology commitment. The methodology commits to *recomputing* PSI, *checking* for counterintuitive SHAP, and *writing* a verdict; the specific findings (not_assessed, 4 counterintuitive, 6/10 REJECT) are this run's results and are recorded here only so a reader of the MD can see how the run played out.

---

## 8. Known methodological limitations and open items

### On the relationship to LC-DS-001's limitations

The data limitations (L1–L8 in LC-DS-001 §8) apply to this methodology — a model cannot use data that does not exist. **They are not restated here.** A reader should consult LC-DS-001 §8 for the data limitations, and this section for the limitations that are specific to the *methodology* — i.e., limitations that arise from the modelling choices, not from the data.

### Methodology-specific limitations

| ID | Limitation | Impact | Tracking |
|---|---|---|---|
| **M1** | **Random split, not walk-forward** — prescribed method could not be executed because no usable date column survives leakage removal | Not time-validated; PSI not assessable; results optimistically biased | LC-R-02 in MAP-RP-001 |
| **M2** | **Four-model comparison does not include a neural network, SVM, or k-NN** — out of scope for 10,000 rows | The comparison is informative for the four classes tested, but not exhaustive | Documented; not closeable within 6-month horizon |
| **M3** | **SHAP is on 100 rows, not the full validation set** | Feature importance is indicative, not exhaustive | Documented; full-population SHAP is a future-version candidate |
| **M4** | **HL degeneracy for 3 of 4 models** — the HL implementation has not been verified, and the p-values of exactly 0.0 or 1.0 are numerical artefacts | Calibration is uncertain for 3 of 4 models; only LightGBM has a valid HL result | LC-R-03 in MAP-RP-001 |
| **M5** | **Region-as-proxy fairness** — the 4/5ths rule is applied to a region proxy, not a direct protected attribute | Fairness findings are indicative, not definitive | Documented; HMDA-equivalent data acquisition is a longer-horizon candidate |
| **M6** | **Single-threshold fairness** — approval rate at 0.5 only; the operating-point sensitivity of the DI is not tested | A different threshold might produce a different DI; the threshold choice is fixed by the methodology, not optimized | Documented; threshold sensitivity is a future-version candidate |

### Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **Split method** | Random stratified split used as a documented exception (F-03.02). | **LC-R-02**: implement walk-forward or formally amend MD-001 §2. |
| 2 | **Calibration implementation** | HL produces degenerate p-values for 3 of 4 models (F-03.05). | **LC-R-03**: verify HL implementation or switch to Spiegelhalter Z-test. |
| 3 | **Counterintuitive SHAP features** | 4 features have SHAP directions opposite to credit-risk theory (F-03.07). | **LC-R-04**: investigate or re-engineer; 2-week target in the Remediation Plan. |
| 4 | **Fairness proxy** | Region is a proxy for protected attributes (F-03.12). | HMDA-equivalent data acquisition is a **longer-horizon** item, not in the 6-month roadmap. |
| 5 | **SHAP sample size** | 100-row sample is indicative. | Full-population SHAP is a **future-version** candidate; not in the 6-month roadmap. |

> **Note on the open items.** Items 1–3 are routed to the Remediation Plan with named owners and due dates. Items 4–5 are documented but **not** routed to the 6-month roadmap, because they require data acquisition (item 4) or a methodology change that is not in the current scope (item 5). A future MAP revision could promote them to the roadmap; the current revision leaves them as documented but untracked.

---

*End of document.*
