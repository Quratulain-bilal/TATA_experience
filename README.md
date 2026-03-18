# Tata iQ Data Analytics Virtual Experience — Geldium Delinquency Risk Project

## Overview

This project was completed as part of the **Tata iQ Data Analytics Virtual Experience Program**. The objective was to help **Geldium**, a financial services company, identify customers at risk of loan delinquency and design an AI-powered collections system to automate and optimize outreach efforts — responsibly and at scale.

The project spans **four tasks**, progressing from exploratory analysis to a fully designed autonomous collections framework:

| Task | Title | Deliverable |
|:----:|-------|-------------|
| 1 | Exploratory Data Analysis (EDA) | Cleaned dataset + EDA notebook + summary report |
| 2 | Predictive Modeling | Gradient Boosting model + risk-scored customers + model plan |
| 3 | Business Report | Stakeholder-ready Word report with SMART recommendations |
| 4 | AI-Powered Collections System | Executive PowerPoint presentation with system framework |

---

## Dataset

- **500 customer records** with 19 variables including demographics, financials, credit behavior, and 6-month payment history
- **Target variable:** `Delinquent_Account` (Binary: 0 = No, 1 = Yes)
- **Delinquency rate:** 16% (80 out of 500 customers)
- **Key fields:** Age, Income, Credit_Score, Credit_Utilization, Missed_Payments, Loan_Balance, Debt_to_Income_Ratio, Employment_Status, Account_Tenure, Credit_Card_Type, Location, Month_1 to Month_6

---

## Task 1: Exploratory Data Analysis (EDA)

**Goal:** Assess data quality, handle missing values, and identify early risk indicators.

**Key findings:**
- **Missing data:** Income (7.8%), Loan_Balance (5.8%), Credit_Score (0.4%) — addressed via median/mean imputation
- **Critical signal:** Customers with missing Loan_Balance data have a **24.1% delinquency rate** vs. 15.5% for complete records
- **Data quality fixes:** Standardized Employment_Status labels, capped Credit_Utilization at 1.0
- **Counterintuitive patterns:** Higher income and credit scores among delinquent customers — suggests non-linear relationships
- **All linear correlations with delinquency are weak** (|r| < 0.05), motivating non-linear modeling

**Files:**

| File | Description |
|------|-------------|
| `EDA_Delinquency.ipynb` | Full Jupyter notebook with code, visualizations (7 panels), and analysis |
| `Delinquency_cleaned_dataset.csv` | Cleaned dataset — 500 rows, 23 columns, 0 missing values |
| `EDA_SummaryReport.md` | Detailed EDA summary report |
| `EDA_SummaryReport_Template.docx` | Original report template provided |
| `Delinquency_prediction_dataset.xlsx` | Original raw dataset |

---

## Task 2: Predictive Modeling

**Goal:** Build and evaluate predictive models to identify at-risk customers.

**Approach:**
- Trained **3 models**: Logistic Regression, Decision Tree, Gradient Boosting
- Applied **SMOTE** oversampling to handle class imbalance (16% → 50%)
- Optimized decision thresholds via F1 maximization
- **Selected model:** Gradient Boosting Classifier (200 trees, depth=3, learning_rate=0.05)

**Model performance (5-fold cross-validation):**

| Model | F1 Score | AUC-ROC |
|-------|:--------:|:-------:|
| Logistic Regression | 0.843 | 0.901 |
| Decision Tree | 0.719 | 0.744 |
| **Gradient Boosting** | **0.862** | **0.929** |

**Top 5 predictors:**
1. `Avg_Payment_Severity` (12.3%) — 6-month payment behavior pattern
2. `Month_5` (9.1%) — Recent payment status
3. `Month_2` (8.9%) — Early payment baseline
4. `Month_6` (7.5%) — Most recent payment
5. `Credit_Utilization` (6.3%) — Financial strain signal

**Explainability:** SHAP values for per-customer explanations + Decision Tree rules as a companion "glass-box" model.

**Files:**

| File | Description |
|------|-------------|
| `Predictive_Model_Delinquency.ipynb` | Full modeling notebook with training, evaluation, SHAP, and fairness analysis |
| `Predictive_Model_Plan.md` | Structured model plan with justification and evaluation strategy |
| `Delinquency_risk_scored.csv` | All 500 customers with risk scores and risk categories (Low/Medium/High) |
| `Delinquency_cleaned_dataset.csv` | Cleaned input dataset |

---

## Task 3: Business Report

**Goal:** Translate predictive findings into a stakeholder-ready report for Geldium's Head of Collections.

**Report sections:**
1. **Summary of Predictive Insights** — Top 3 risk factors and high-risk customer segments
2. **SMART Business Recommendation** — Proactive outreach targeting customers with >80% credit utilization + 2+ missed payments; goal to reduce delinquency by 15% within 6 months
3. **Ethical & Responsible AI Considerations** — Fairness risks (employment bias, geographic proxy discrimination) with mitigation strategies
4. **AI & GenAI Usage Disclosure**

**Files:**

| File | Description |
|------|-------------|
| `Geldium_Delinquency_Business_Report.docx` | Two-page stakeholder report (Word format) |
| `generate_report.py` | Python script used to generate the report programmatically |

---

## Task 4: AI-Powered Collections System Presentation

**Goal:** Design a framework for an autonomous, responsible AI-powered collections system.

**Presentation slides (7 slides):**

| Slide | Content |
|:-----:|---------|
| 1 | Title slide with key project stats |
| 2 | System overview — 4-stage loop: Data Pipeline → Decision Engine → Action Layer → Learning Loop |
| 3 | Workflow detail — Risk-tier actions (Low/Medium/High) + top model predictors + real-time adaptation example |
| 4 | Agentic AI — Autonomous vs. Human-in-the-Loop activities (6 examples each) |
| 5 | Responsible AI guardrails — Fairness, Explainability, Compliance (ECOA/GDPR/FCA/FCRA), Monitoring |
| 6 | Expected business impact — 15% delinquency reduction, 40% less manual effort, better CX, regulatory confidence |
| 7 | Summary + 4-phase rollout plan (90-day pilot → validation → scale → ongoing) |

**Files:**

| File | Description |
|------|-------------|
| `Geldium_AI_Collections_System.pptx` | Executive briefing PowerPoint (7 slides) |
| `generate_pptx.py` | Python script used to generate the presentation programmatically |

---

## Project Structure

```
TATA_experience/
│
├── README.md                              # This file
│
├── Task1_EDA/
│   ├── Delinquency_prediction_dataset.xlsx   # Original raw dataset
│   ├── EDA_Delinquency.ipynb                 # EDA Jupyter notebook
│   ├── Delinquency_cleaned_dataset.csv       # Cleaned dataset
│   ├── EDA_SummaryReport.md                  # EDA summary report
│   └── EDA_SummaryReport_Template.docx       # Report template
│
├── Task2_Predictive_Model/
│   ├── Delinquency_cleaned_dataset.csv       # Input dataset
│   ├── Predictive_Model_Delinquency.ipynb    # Modeling notebook
│   ├── Predictive_Model_Plan.md              # Model plan document
│   └── Delinquency_risk_scored.csv           # Risk-scored output
│
├── Task3_Business_Report/
│   ├── Geldium_Delinquency_Business_Report.docx  # Stakeholder report
│   └── generate_report.py                         # Report generation script
│
└── Task4_AI_Collections_Presentation/
    ├── Geldium_AI_Collections_System.pptx    # Executive presentation
    └── generate_pptx.py                      # Presentation generation script
```

---

## Tech Stack

- **Python 3.11** — Primary language
- **pandas, numpy** — Data manipulation
- **scikit-learn** — Machine learning (Logistic Regression, Decision Tree, Gradient Boosting)
- **imbalanced-learn** — SMOTE oversampling
- **SHAP** — Model explainability
- **matplotlib, seaborn** — Data visualization
- **python-docx** — Word document generation
- **python-pptx** — PowerPoint generation
- **Jupyter Notebook** — Interactive analysis environment
- **Claude Code (GenAI)** — AI-assisted analysis, code generation, and report drafting

---

## Key Takeaways

- **Payment behavior is the #1 predictor** of delinquency — not credit score or income alone
- **Data quality signals matter** — missing Loan Balance data is itself a strong risk indicator
- **Non-linear models outperform linear ones** when individual feature correlations are weak
- **Responsible AI is non-negotiable** in financial services — fairness audits, explainability, and human oversight must be built in from the start
- **Agentic AI can transform collections** by enabling autonomous, personalized outreach at scale while keeping humans in the loop for high-stakes decisions

---

## Author

**Quratulain Bilal**
Tata iQ Data Analytics Virtual Experience Program — March 2026

---


