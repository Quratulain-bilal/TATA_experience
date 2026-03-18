# Exploratory Data Analysis (EDA) Summary Report

---

## 1. Introduction

The purpose of this report is to conduct an Exploratory Data Analysis (EDA) on Geldium's customer delinquency dataset to support Tata iQ's analytics team in understanding the current state of the data before predictive modeling begins. The goals are to: (1) assess data quality and completeness, (2) address missing and inconsistent data, and (3) identify early risk indicators and patterns that influence delinquency, which will shape how the company refines its delinquency risk model and improves intervention strategies.

---

## 2. Dataset Overview

This section summarizes the dataset, including the number of records, key variables, and data types. It also highlights any anomalies, duplicates, or inconsistencies observed during the initial review.

**Key dataset attributes:**

- **Number of records:** 500 customers
- **Number of features:** 19 columns
- **Target variable:** Delinquent_Account (Binary: 0=No, 1=Yes)
- **Delinquency rate:** 16.00% (80 delinquent out of 500)

**Key variables:**

| Variable | Type | Description |
|----------|------|-------------|
| Customer_ID | Categorical | Unique identifier for each customer |
| Age | Numerical | Customer's age (range: 18–74) |
| Income | Numerical | Annual income in USD (mean: $108,380; 39 missing) |
| Credit_Score | Numerical | Credit score 300–850 (mean: 578; 2 missing) |
| Credit_Utilization | Numerical | % of available credit in use (stored as decimal 0–1) |
| Missed_Payments | Numerical | Total missed payments in past 12 months (0–6) |
| Delinquent_Account | Binary | Target variable (0=No, 1=Yes) |
| Loan_Balance | Numerical | Outstanding loan balance in USD (29 missing) |
| Debt_to_Income_Ratio | Numerical | Ratio of total debt to income (0.10–0.55) |
| Employment_Status | Categorical | Employment status (6 raw categories, 4 after cleaning) |
| Account_Tenure | Numerical | Years as customer (0–19) |
| Credit_Card_Type | Categorical | Card type: Gold, Student, Business, Standard, Platinum |
| Location | Categorical | City: Los Angeles, Phoenix, Chicago, Houston, New York |
| Month_1 to Month_6 | Categorical | Monthly payment status: On-time, Late, or Missed |

**Anomalies and inconsistencies found:**

- **Employment_Status inconsistency:** Three labels represent the same category — "Employed", "employed", and "EMP" (combined 240 records). This must be standardized before modeling.
- **Credit_Utilization outliers:** 4 records exceed 1.0 (100%), with a maximum of 1.0258 — likely data entry errors or temporary over-limit charges.
- **No duplicate rows** were found; all Customer_IDs are unique.
- **Class imbalance:** Only 16% of customers are delinquent, which will require handling during modeling (e.g., SMOTE or class weighting).

---

## 3. Missing Data Analysis

Identifying and addressing missing data is critical to ensuring model accuracy. This section outlines missing values in the dataset, the approach taken to handle them, and justifications for the chosen method.

**Key missing data findings:**

| Column | Missing Count | Missing % | Strategy | Justification |
|--------|:------------:|:---------:|----------|---------------|
| Income | 39 | 7.8% | Median Imputation | Missingness is correlated with delinquency (MAR pattern: 12.8% delinquency rate in missing group vs 16.3% in non-missing). Median ($107,658) is more robust than mean for potentially biased subsets. |
| Loan_Balance | 29 | 5.8% | Median Imputation + Missing Indicator Flag | Missingness is **NOT random** — customers with missing loan balance have 24.1% delinquency rate vs 15.5% for non-missing. Median ($45,776) used for imputation, and a binary `Loan_Balance_Missing` flag was created to preserve the missingness signal for modeling. |
| Credit_Score | 2 | 0.4% | Mean Imputation | Only 2 records missing — negligible impact. Mean (578) preserves central tendency with minimal distribution distortion. |

**Additional data quality fixes applied:**

- **Employment_Status standardized:** "Employed", "employed", "EMP" all mapped to "Employed" (final categories: Employed, Self-employed, Unemployed, Retired)
- **Credit_Utilization capped** at 1.0 for 4 records exceeding 100%
- **Cleaned dataset:** 500 rows, 23 columns (including 4 engineered features), 0 missing values

---

## 4. Key Findings and Risk Indicators

This section identifies trends and patterns that may indicate risk factors for delinquency. Feature relationships and statistical correlations are explored to uncover insights relevant to predictive modeling.

### High-Risk Indicators

| Risk Indicator | Delinquency Rate | Why It Matters |
|----------------|:----------------:|----------------|
| **Unemployed customers** | 19.4% | Lack of stable income directly impacts repayment capacity |
| **Business credit card holders** | 21.3% | Highest among all card types — possibly reflects business cash flow volatility |
| **Los Angeles location** | 19.6% | Highest among all cities — may reflect regional cost-of-living pressures |
| **Missing Loan Balance data** | 24.1% | The missingness itself is a strong risk signal (vs 15.5% for non-missing) |
| **Retired customers** | 11.5% | Lowest delinquency rate — stable pension/retirement income provides reliability |
| **Platinum card holders** | 11.8% | Lowest among card types — these tend to be premium customers with stronger financials |

### Correlation Analysis

All individual feature correlations with Delinquent_Account are **very weak** (|r| < 0.05):

| Feature | Correlation | Direction |
|---------|:----------:|-----------|
| Income | +0.0454 | Weak positive |
| Account_Tenure | -0.0398 | Weak negative |
| Credit_Score | +0.0348 | Weak positive |
| Debt_to_Income_Ratio | +0.0344 | Weak positive |
| Credit_Utilization | +0.0342 | Weak positive |
| Missed_Payments | -0.0265 | Weak negative |
| Age | +0.0225 | Weak positive |
| Loan_Balance | -0.0043 | Negligible |

### Unexpected Anomalies Requiring Further Investigation

1. **Higher income among delinquent customers:** Delinquent customers average $113,902 vs $107,307 for non-delinquent. This is counterintuitive — possibly because higher earners qualify for larger loans and take on proportionally more debt.

2. **Higher credit scores among delinquent customers:** Delinquent group averages 591 vs 575. This contradicts conventional wisdom and may indicate that credit score alone is not protective against delinquency.

3. **Missed_Payments has a negative correlation with delinquency (-0.027):** More missed payments is slightly associated with *less* delinquency. This likely reflects a data definition nuance or non-linear effect that needs clarification with domain experts.

4. **Monthly payment history shows no temporal trend:** The distribution of "On-time", "Late", and "Missed" payments is nearly uniform across all 6 months, with no clear improvement or deterioration pattern.

### Mean Comparison: Delinquent vs Non-Delinquent

| Feature | Non-Delinquent Mean | Delinquent Mean | Difference |
|---------|:-------------------:|:---------------:|:----------:|
| Age | 46.1 | 47.1 | +1.0 |
| Income | $107,307 | $113,902 | +$6,595 |
| Credit_Score | 575 | 591 | +16 |
| Credit_Utilization | 0.49 | 0.51 | +0.02 |
| Missed_Payments | 2.99 | 2.85 | -0.14 |
| Loan_Balance | $48,709 | $48,358 | -$351 |
| Debt_to_Income_Ratio | 0.30 | 0.31 | +0.01 |
| Account_Tenure | 9.84 | 9.20 | -0.64 |

---

## 5. AI & GenAI Usage

Generative AI tools (Claude Code) were used throughout this EDA to accelerate analysis, automate data cleaning, and surface insights. Below are the key AI-assisted activities and the prompts used.

**AI-assisted activities:**

- **Dataset summarization:** AI analyzed the dataset structure, data types, distributions, and immediately flagged the Employment_Status inconsistency and Credit_Utilization anomalies.
- **Missing data strategy:** AI evaluated missingness patterns (MCAR vs MAR/MNAR) by comparing delinquency rates between missing and non-missing groups, then recommended appropriate imputation strategies.
- **Pattern detection:** AI computed correlations, performed group comparisons, and identified counterintuitive findings (e.g., higher income/credit scores in delinquent group).
- **Feature engineering:** AI created aggregated payment history features (Total_Missed_6M, Total_Late_6M, Total_Problematic_6M) and a Loan_Balance_Missing indicator.
- **Visualization:** AI generated distribution histograms, box plots, correlation heatmaps, delinquency rate charts, and pair plots for feature interactions.

**Example AI prompts used:**

- "Summarize key patterns, outliers, and missing values in this dataset. Highlight any fields that might present problems for modeling delinquency."
- "Identify the top 3 variables most likely to predict delinquency based on this dataset. Provide brief reasoning."
- "Suggest an imputation strategy for missing values in this dataset based on industry best practices."
- "Propose best-practice methods to handle missing credit utilization data for predictive modeling."
- "Analyze whether missingness in Loan_Balance is random or correlated with the target variable."

---

## 6. Conclusion & Next Steps

### Key Findings Summary

The Geldium delinquency dataset contains 500 customer records with a 16% delinquency rate. Three columns had missing values (Income 7.8%, Loan_Balance 5.8%, Credit_Score 0.4%), which were addressed through median/mean imputation. A critical finding is that Loan_Balance missingness is **not random** and correlates with higher delinquency (24.1% vs 15.5%), making the missing indicator itself a valuable predictor. The Employment_Status field required standardization due to inconsistent encoding. All individual linear correlations with delinquency are very weak (|r| < 0.05), and several counterintuitive patterns (higher income and credit scores among delinquent customers) suggest that the relationship between features and delinquency is non-linear and complex.

### Recommended Next Steps

1. **Use non-linear models** (Random Forest, Gradient Boosting, XGBoost) rather than logistic regression, given the weak linear correlations.
2. **Engineer features from payment history:** Aggregate monthly payment patterns into counts of missed/late payments and payment trajectory features.
3. **Include the Loan_Balance_Missing flag** as a model feature — it is one of the strongest risk signals identified.
4. **Address class imbalance** (16% delinquency) using SMOTE, class weighting, or stratified sampling.
5. **Investigate counterintuitive findings** with domain experts — particularly why higher-income and higher-credit-score customers show slightly elevated delinquency rates.
6. **Consider interaction terms** between Employment_Status, Credit_Card_Type, and Location, as these categorical features show meaningful delinquency rate variation.
7. **Validate the Missed_Payments variable definition** with the data team to clarify the negative correlation with delinquency.

### Files Produced

| File | Description |
|------|-------------|
| `EDA_Delinquency.ipynb` | Full Jupyter notebook with code, outputs, and 7 visualization panels |
| `Delinquency_cleaned_dataset.csv` | Cleaned dataset (500 rows, 23 columns, 0 missing values) |
| `EDA_SummaryReport.md` | This summary report |
