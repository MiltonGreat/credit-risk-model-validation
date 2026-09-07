# Lending Club Credit Risk Model Validation

## 📊 Project Overview

**Role:** GRC Risk Analyst | **Focus:** Model Validation & Regulatory Compliance | **Regulatory Frameworks:** OSFI E-23, SR 11-7, ECOA, CFPB

This project performs an independent model validation of a credit risk model developed on Lending Club data. The validation follows OSFI E-23 and SR 11-7 requirements, assessing model performance, data quality, fairness, and regulatory compliance.

### Business Problem
The first-line model development team built a credit risk model to predict 24-month default probability. This validation determines:
- Does the model work as intended?
- Is it fair across protected groups?
- Does it meet regulatory requirements?
- Can it be deployed?

### My Role
As a GRC Risk Analyst, I:
- Independently validated the model
- Identified critical issues (data leakage, fairness concerns)
- Quantified business impact
- Provided remediation recommendations
- Documented compliance status

---

## 🎯 Key Findings

| Finding | Severity | Impact | Status |
|---------|----------|--------|--------|
| **Data Leakage (9 features)** | CRITICAL | AUC inflated 28.2% (0.865 → 0.621) | ✅ RESOLVED |
| **Severe Fairness Concerns** | CRITICAL | DI: 0.244 (well below 0.8 threshold) | ✅ RESOLVED |
| **IFRS 9 Misalignment** | HIGH | 16+ day threshold vs 30+ days | ⚠️ DOCUMENTED |
| **Foreign Data Applicability** | HIGH | U.S. data only (not validated for Canada) | ⚠️ RECOMMENDED |
| **Model Performance** | MODERATE | AUC-ROC: 0.621 (realistic) | ⚠️ ACCEPTABLE |

### Fairness Improvement

- **DI Before Mitigation:** 0.244 (SEVERE)
- **DI After Mitigation:** 0.798 (MONITOR)
- **Improvement:** +0.554 DI points
- **Action:** 49 state features removed


### Final Recommendation
> **CONDITIONAL APPROVAL** - Model may be deployed with:
> 1. Quarterly fairness monitoring (DI > 0.8)
> 2. Monthly performance monitoring (AUC-ROC, PSI)
> 3. Canadian data validation before deployment
> 4. IFRS 9 limitation documented in Intended Use Statement

---

## 📋 Regulatory Compliance Status

### OSFI E-23 Compliance
| Section | Status |
|---------|--------|
| 3.1 - Model Validation | ✅ COMPLIANT |
| 3.2 - Performance Monitoring | ⚠️ PARTIALLY COMPLIANT |
| 3.3 - Data Quality | ✅ COMPLIANT |
| 4.1 - Documentation | ✅ COMPLIANT |
| 4.2 - Governance | ✅ COMPLIANT |
| 5.1 - Model Risk | ✅ COMPLIANT |
| 5.2 - Fairness | ✅ COMPLIANT |

### SR 11-7 Compliance
| Principle | Status |
|-----------|--------|
| 1 - Validation | ✅ COMPLIANT |
| 2 - Monitoring | ⚠️ PARTIALLY COMPLIANT |
| 3 - Documentation | ✅ COMPLIANT |
| 4 - Model Risk | ✅ COMPLIANT |
| 5 - Fairness | ✅ COMPLIANT |
| 6 - Data Quality | ✅ COMPLIANT |

### ECOA/CFPB Fair Lending
- **Fairness Score:** 76.0/100 (MODERATE)
- **Disparate Impact Ratio:** 0.798 (MONITOR)
- **Status:** Conditional approval with quarterly monitoring

---

## 🏗️ Project Structure

| Notebook | Purpose | Key Output |
|----------|---------|------------|
| 01_EDA | Data quality assessment | Data quality report, leakage detection |
| 02_Model Development | Build & document models | Performance metrics, calibration |
| 03_Model Validation | Independent validation | Validation report, risk score |
| 04_Governance Report | Regulatory compliance | Compliance report, remediation plan |

---

## 💻 Technical Stack

- **Python 3.10+** with pandas, numpy, scikit-learn
- **Machine Learning:** XGBoost, LightGBM, Random Forest, Logistic Regression
- **Explainability:** SHAP
- **Fairness Testing:** Disparate Impact Analysis
- **Visualization:** matplotlib, seaborn

---

## 📊 Performance Summary

| Model | AUC-ROC | Precision | Recall | Calibration |
|-------|---------|-----------|--------|-------------|
| Random Forest | 0.642 | 0.080 | 0.306 | ✅ GOOD |
| XGBoost | 0.619 | 0.029 | 0.611 | ❌ POOR |
| LightGBM | 0.601 | 0.028 | 0.556 | ✅ GOOD |
| Logistic Regression | 0.554 | 0.034 | 0.361 | ✅ GOOD |

---

## 📝 Key Documents

| Document | Description |
|----------|-------------|
| [Intended Use Statement](./docs/intended_use_statement.md) | Model purpose and limitations |
| [Data Dictionary](./docs/data_dictionary.md) | Feature definitions and sources |
| [Regulatory Mapping](./docs/regulatory_mapping.md) | OSFI E-23, SR 11-7 mapping |
| [Remediation Plan](./docs/remediation_plan.md) | Actionable remediation recommendations |
| [Validation Report](./outputs/reports/validation_report_v2.txt) | Full validation findings |
| [Compliance Report](./outputs/reports/compliance_report_v2.txt) | Regulatory compliance assessment |

---

## 🚀 Getting Started

### Installation
```bash
# Clone the repository
git clone https://github.com/yourusername/lending-club-model-validation.git
cd lending-club-model-validation

# Install dependencies
pip install -r requirements.txt

---

# Data Directory

## Data Source

This project uses the Fannie Mae Single-Family Loan Performance dataset.

### Download Instructions

**Visit Kaggle.com to find the Fannie Mae Data:**
   - Go to: [[https://www.fanniemae.com/research-and-insights/data](https://www.kaggle.com/datasets/pranay07/fanne-mae-loan-performance-data)](https://www.kaggle.com/datasets/utkarshx27/lending-club-loan-dataset?resource=download)
