# Credit Card Fraud Detection with Cost-Based Thresholds

Detecting fraudulent card transactions and choosing the alert threshold that minimizes total business cost, balancing missed fraud against blocked genuine customers.

## Data
Credit Card Fraud Detection (ULB, Kaggle): 284,807 European card transactions, 492 frauds (0.17%). Features V1 to V28 are anonymized for privacy.

## Key Findings
- Fraud peaks between 1 and 7 AM: 1.71% at 2 AM vs about 0.1% in daytime.
- Transactions of 2 EUR or less are 3.3x more likely to be fraud, consistent with card testing.
- Median fraud amount is 9.25 vs 22 for genuine transactions.

## Approach
1. Engineered features: night flag, small transaction flag, log amount.
2. Stratified 70/30 split.
3. Logistic regression with balanced class weights vs XGBoost.
4. Compared models on PR-AUC, since ROC-AUC is misleading on rare events.
5. Chose the threshold by minimizing cost under stated assumptions.

## Model Comparison
| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Logistic regression | 0.971 | 0.699 |
| XGBoost | 0.972 | 0.823 |

Same ROC-AUC, very different PR-AUC. At a 0.5 threshold, logistic regression blocked 1,999 genuine customers; XGBoost blocked 7.

![PR curve](img.png)

## Threshold Strategy
Assumptions: 150 EUR per missed fraud, 10 EUR per genuine customer wrongly blocked.

| Scenario | Fraud caught | Genuine blocked | Cost (EUR) |
|---|---|---|---|
| No model | 0% | 0 | 22,200 |
| Logistic, 0.5 threshold | 87% | 1,999 | 22,840 |
| XGBoost, 0.5 threshold | 74% | 7 | 5,920 |
| **XGBoost, 0.03 threshold** | **81%** | **39** | **4,590** |

**Recommendation:** XGBoost at a low threshold (0.02 to 0.05). It cuts fraud cost by 79% vs no model while blocking only 0.05% of genuine customers. Cost is stable across 0.01 to 0.10, so the choice is robust.

## Robustness Checks
**Cross-validation:** 5-fold stratified CV gives XGBoost a PR-AUC of 0.845 ± 0.030 (range 0.80 to 0.88), so the single-split result is not a lucky draw.

**Cost sensitivity:** the best threshold shifts with the cost of blocking a genuine customer, but savings stay large.

| Missed fraud : blocked customer | Best threshold | Fraud caught | Genuine blocked | Cost (EUR) |
|---|---|---|---|---|
| 30:1 | 0.03 | 120 | 39 | 4,395 |
| 15:1 | 0.03 | 120 | 39 | 4,590 |
| 6:1 | 0.07 | 118 | 21 | 5,025 |
| 3:1 | 0.13 | 116 | 14 | 5,500 |
| 2:1 | 0.13 | 116 | 14 | 5,850 |

At every ratio, cost stays 74 to 80% below no model (22,200 EUR).

## Limitations
- V14 drives 75% of XGBoost's decisions; reliance on one feature should be monitored for drift as fraud tactics change.
- Data covers two days of European transactions from 2013, so patterns may differ today.
