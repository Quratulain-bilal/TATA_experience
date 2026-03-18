# Predictive Model Plan: Geldium Customer Delinquency Risk

## Tata iQ Analytics Team - Task 2

---

## 1. Model Logic: How the Delinquency Risk Model Works

### Chosen Model: Gradient Boosting Classifier

Gradient Boosting was selected as the primary predictive model for identifying at-risk customers. It is an ensemble machine learning method that builds sequential decision trees, where each new tree corrects the errors of the previous ones, progressively improving predictions. This approach is particularly suited for Geldium's dataset because EDA revealed that all individual feature correlations with delinquency are very weak (|r| < 0.05), indicating the need for a model that can capture non-linear interactions between variables rather than relying on linear relationships.

### Top 5 Input Features

| Rank | Feature | Importance | Why It Matters |
|:----:|---------|:----------:|----------------|
| 1 | **Avg_Payment_Severity** | 0.1231 | Aggregated monthly payment behavior (On-time=0, Late=1, Missed=2) — captures overall payment discipline across 6 months |
| 2 | **Month_5** | 0.0908 | Recent payment status — more recent behavior is more predictive of near-term delinquency risk |
| 3 | **Month_2** | 0.0891 | Early payment behavior — establishes baseline payment pattern |
| 4 | **Month_6** | 0.0746 | Most recent payment status — strongest temporal signal of current risk |
| 5 | **Credit_Utilization** | 0.0629 | Percentage of available credit in use — higher utilization signals financial strain |

### Predictive Pipeline (End-to-End Workflow)

```
DATA INGESTION → PREPROCESSING → FEATURE ENGINEERING → MODEL TRAINING → PREDICTION → RISK SCORING
```

**Step-by-step:**

1. **Data Ingestion**: Load raw customer data (demographics, financials, payment history)
2. **Preprocessing**:
   - Standardize Employment_Status labels ("Employed"/"employed"/"EMP" → "Employed")
   - Cap Credit_Utilization at 1.0 (100%)
   - Impute missing values: Income (median), Loan_Balance (median + missing flag), Credit_Score (mean)
3. **Feature Engineering**:
   - Encode monthly payments: On-time=0, Late=1, Missed=2
   - Create `Avg_Payment_Severity` (mean of 6-month payment scores)
   - Create `Payment_Trend` (last 3 months avg minus first 3 months avg)
   - Create `Max_Consecutive_Missed` (longest streak of missed payments)
   - Encode categoricals: Employment, Credit Card Type, Location
4. **Class Balancing**: Apply SMOTE oversampling to training data (16% → 50% delinquency)
5. **Model Training**: Gradient Boosting with 200 trees, depth=3, learning_rate=0.05
6. **Prediction**: Generate probability score (0–1) for each customer
7. **Risk Categorization**: Low Risk (0–0.3), Medium Risk (0.3–0.6), High Risk (0.6–1.0)

### Decision Tree Rules (Companion Explainable Model)

For stakeholder transparency, a Decision Tree model provides interpretable rules:

```
IF Month_2 is On-time AND Month_5 is On-time AND Employment is not Retired:
    → Check Month_3 payment
    → IF Month_3 is On-time or Late → HIGH RISK flag
    → IF Month_3 is Missed → LOW RISK

IF Month_2 is Missed AND Account_Tenure > 9.5 years AND Debt_to_Income > 20%:
    → Check Credit Score
    → IF Credit_Score > 680 → LOW RISK
    → IF Credit_Score <= 680 → LOW RISK
```

---

## 2. Model Choice Justification

### Why Gradient Boosting?

Gradient Boosting is the most appropriate model for Geldium's delinquency prediction because it directly addresses the dataset's key challenge: **weak linear relationships between features and the target variable**. During EDA, no single feature showed a statistically significant difference (p < 0.05) between delinquent and non-delinquent customers, and all correlations were below |0.05|. This rules out simple linear models like logistic regression as standalone solutions. Gradient Boosting excels at discovering complex, non-linear interaction effects — for example, a customer with high credit utilization AND missed Month_5 payment AND low account tenure may be at elevated risk, even though none of these factors alone are predictive. Additionally, it provides built-in feature importance rankings and integrates seamlessly with SHAP (Shapley Additive Explanations) for per-customer explainability — a critical requirement for Geldium's Collections team, who need to understand *why* a customer was flagged, and for regulatory compliance in financial services. The Decision Tree serves as a companion "glass-box" model that can be directly shown to non-technical decision-makers.

### Model Comparison Summary

| Model | Strengths | Weaknesses | Best For |
|-------|-----------|------------|----------|
| **Logistic Regression** | Simple, interpretable, probability outputs, regulatory-friendly | Assumes linear relationships, poor with weak signals | Baseline model, regulatory reporting |
| **Decision Tree** | Transparent rules, easy to explain to stakeholders, handles mixed data | Prone to overfitting, unstable with small changes | Stakeholder communication, rule extraction |
| **Gradient Boosting** (Selected) | Captures non-linear interactions, ensemble reduces overfitting, SHAP explainability | More complex, slower to train, requires careful tuning | Primary production model |

### Cross-Validation Results (5-Fold Stratified)

| Model | Mean F1 Score | Mean AUC-ROC |
|-------|:------------:|:------------:|
| Logistic Regression | 0.8427 (±0.0362) | 0.9014 (±0.0267) |
| Decision Tree | 0.7190 (±0.0263) | 0.7437 (±0.0255) |
| **Gradient Boosting** | **0.8618 (±0.0299)** | **0.9288 (±0.0283)** |

Gradient Boosting achieves the highest cross-validation AUC-ROC (0.9288) and F1 (0.8618), confirming it is the most reliable model for this dataset.

---

## 3. Evaluation Strategy

### 3.1 Key Metrics and How to Interpret Them

| Metric | What It Measures | Target | Why It Matters for Geldium |
|--------|-----------------|--------|---------------------------|
| **Accuracy** | Overall correct predictions | > 0.80 | General model correctness |
| **Precision** | Of those flagged as delinquent, how many actually are | > 0.50 | Avoids wasting Collections team resources on false alarms |
| **Recall** | Of actual delinquent customers, how many were caught | > 0.70 | Missing a delinquent customer = financial loss (high recall is priority) |
| **F1 Score** | Harmonic mean of Precision & Recall | > 0.50 | Balanced measure when both false positives and false negatives are costly |
| **AUC-ROC** | Model's ability to rank-order risk (threshold-independent) | > 0.70 | Evaluates overall discriminative power regardless of threshold choice |
| **Confusion Matrix** | Breakdown of TP, FP, TN, FN | Visual | Diagnoses specific error types — where is the model failing? |

**Interpretation Guide:**
- If **Recall is low**: Model is missing delinquent customers → lower the decision threshold
- If **Precision is low**: Too many false alarms → raise the threshold or improve features
- If **AUC-ROC ≈ 0.50**: Model has no discriminative power → need better features or more data

### 3.2 Reliability Assessment

| Method | Purpose |
|--------|---------|
| **5-Fold Stratified Cross-Validation** | Ensures performance is stable across different data splits, not just one lucky test set |
| **Train/Test Split (80/20)** | Evaluates on unseen data to detect overfitting |
| **SMOTE on training only** | Prevents data leakage — synthetic samples never appear in test set |
| **Threshold Optimization** | Finds the optimal probability cutoff rather than defaulting to 0.5 |

### 3.3 Fairness & Bias Checks

| Check | Method | Acceptable Threshold |
|-------|--------|---------------------|
| **Disparate Impact** | Compare predicted delinquency rates across Employment Status groups | Ratio > 0.80 (4/5ths rule) |
| **Group Accuracy Parity** | Ensure model accuracy is consistent across Location and Age groups | Max accuracy gap < 15% |
| **Prediction Rate Parity** | Compare predicted vs actual delinquency rates per group | Gap < 5 percentage points |
| **Protected Group Analysis** | Check if Age, Location act as proxies for protected characteristics | No systematic disadvantage |

**Bias Mitigation Techniques:**
1. **Pre-processing**: SMOTE ensures balanced class representation in training
2. **In-processing**: Class weighting can adjust model sensitivity to minority groups
3. **Post-processing**: Threshold adjustment per demographic group if disparate impact is detected
4. **Monitoring**: Track fairness metrics monthly as new data arrives

### 3.4 Explainability Framework

| Tool | Purpose | Audience |
|------|---------|----------|
| **SHAP Values** | Per-customer feature contribution breakdown | Analytics team, Compliance |
| **Decision Tree Rules** | Human-readable if/then rules | Business stakeholders, Collections team |
| **Feature Importance** | Global ranking of what drives predictions | Leadership, Model governance |

Example SHAP explanation for a high-risk customer:
> *"Customer CUST0033 was flagged as High Risk (score: 0.98) primarily because of: high average payment severity (+0.25), missed Month_5 payment (+0.18), and high credit utilization (+0.12)."*

### 3.5 When to Retrain / Improve the Model

| Trigger | Action |
|---------|--------|
| Precision drops below 0.40 on new monthly data | Retrain with updated data |
| Delinquency rate shifts by > 5 percentage points | Recalibrate thresholds |
| New customer features become available (e.g., transaction data) | Add features and retrain |
| Disparate impact ratio falls below 0.80 | Investigate bias and apply mitigation |
| AUC-ROC drops below 0.60 on validation data | Consider alternative model architectures |

---

## 4. Critical Data Quality Observation

During EDA, statistical testing revealed that **no individual feature has a statistically significant difference** (p < 0.05) between delinquent and non-delinquent groups using Mann-Whitney U tests. This fundamental limitation means:

- All models are constrained by weak signal-to-noise ratio in the current dataset
- Ensemble methods (Gradient Boosting) perform best because they can combine weak signals
- **Recommendation to Geldium**: Enrich the dataset with additional behavioral signals such as:
  - Transaction frequency and amounts
  - Customer service contact history
  - Account age at first missed payment
  - External credit bureau data updates
  - Payment channel (auto-pay vs manual)

---

## 5. AI & GenAI Usage

GenAI (Claude Code) was used throughout this task to:

- **Model selection**: Analyzed dataset characteristics and recommended Gradient Boosting over simpler alternatives, citing weak linear correlations as the key factor
- **Code generation**: Generated the full modeling pipeline including feature engineering, SMOTE resampling, model training, threshold optimization, and evaluation
- **Performance analysis**: Identified that default 0.5 threshold was suboptimal and implemented automatic threshold tuning via F1 maximization
- **Explainability**: Set up SHAP analysis and decision tree rule extraction for model transparency
- **Fairness assessment**: Designed disparate impact analysis across employment, location, and age groups

**Prompts used:**
- "Generate a predictive modeling pipeline to forecast credit delinquency, from feature selection to model evaluation"
- "Compare logistic regression, decision trees, and gradient boosting for predicting delinquency — recommend one with justification"
- "Suggest evaluation metrics for a financial risk prediction model covering fairness, bias, and accuracy"
- "Generate SHAP-based explainability analysis for the gradient boosting model"

---

## 6. Files Produced

| File | Description |
|------|-------------|
| `Predictive_Model_Delinquency.ipynb` | Full Jupyter notebook with code, visualizations, and model outputs |
| `Delinquency_risk_scored.csv` | All 500 customers with risk scores and risk categories |
| `Predictive_Model_Plan.md` | This structured model plan (submission document) |
