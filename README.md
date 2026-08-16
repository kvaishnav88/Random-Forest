## Overview
Random Forest is an **ensemble learning** method: rather than relying on one decision tree (which tends to overfit badly on its own), it builds many decision trees and combines their predictions. The "random" in the name comes from two deliberate sources of randomness injected during training, which together decorrelate the individual trees — this decorrelation is *why* averaging many trees reduces variance so effectively compared to a single deep tree.

## How It Works
1. **Bootstrap Aggregating (Bagging):** each tree is trained on a random bootstrap sample of the training data (sampled with replacement, same size as the original dataset) — meaning each tree sees a slightly different version of the data
2. **Random feature subsetting:** at each split in each tree, only a random subset of features (commonly √n_features for classification, n_features/3 for regression) is considered as candidates for the best split — this is the second, and arguably more important, source of decorrelation, since it prevents a single dominant feature from driving nearly identical splits across every tree
3. **Aggregation:** for classification, each tree "votes" for a class and the majority vote wins (or probabilities are averaged); for regression, predictions from all trees are averaged

## Methods & Techniques
- **Key hyperparameters and their tuning:**
  - **`n_estimators`** (number of trees): more trees generally improve performance up to a point of diminishing returns, and — unlike boosting — more trees in a Random Forest do *not* increase overfitting risk (each additional tree just further averages down variance); the main cost is training/prediction time
  - **`max_depth`:** limits how deep each tree can grow — shallower trees reduce overfitting but may underfit; unlimited depth is common in Random Forest specifically (unlike single decision trees) because the ensemble averaging already controls variance
  - **`max_features`:** the number of features considered at each split — the primary lever controlling tree decorrelation; lower values increase diversity between trees (more randomness) at some cost to individual tree accuracy
  - **`min_samples_split` / `min_samples_leaf`:** minimum samples required to split a node or exist in a leaf — higher values act as regularization, preventing trees from creating overly specific, noise-fitting splits
  - Standard tuning approach: **grid search or randomized search with cross-validation**, since Random Forest has relatively few critical hyperparameters and is fairly robust to imperfect tuning compared to boosting methods
- **Out-of-Bag (OOB) evaluation:** because each tree is trained on a bootstrap sample, roughly 37% of the data is left out ("out-of-bag") for each tree — these left-out samples can be used to get a free, built-in validation estimate (`oob_score=True`) without needing a separate held-out validation set
- **Feature importance:** Random Forest provides two standard ways to rank feature importance —
  - **Mean Decrease in Impurity (MDI):** the default, fast, but can be biased toward high-cardinality/continuous features
  - **Permutation importance:** shuffles each feature's values and measures the resulting drop in model performance — slower to compute but generally more reliable and less biased than MDI
  - **SHAP values** are increasingly preferred over both for a more rigorous, theoretically grounded feature attribution, especially when explaining individual predictions rather than just global importance
- **Handling class imbalance:** `class_weight='balanced'` (or `'balanced_subsample'`, which reweights per bootstrap sample rather than globally) to upweight the minority class during tree construction
- **Missing value handling:** Random Forest (via most implementations) handles missing values less gracefully than boosting libraries like XGBoost/LightGBM — typically requires imputation beforehand
- **Extremely Randomized Trees (Extra Trees):** a close relative that adds a third source of randomness (random split thresholds, not just random feature subsets) — usually faster to train, sometimes with a small further reduction in variance, worth benchmarking against standard Random Forest

## Evaluation Metrics
Same as Logistic Regression/SVM for classification (Accuracy, Precision, Recall, F1, ROC-AUC, confusion matrix) or Linear Regression for regression (RMSE, MAE, R²) — plus OOB score as a built-in sanity check, and feature importance rankings as a standard part of the analysis.

## When to Use It
An excellent default choice for tabular/structured data — handles nonlinear relationships and feature interactions automatically (no manual polynomial/interaction terms needed, unlike linear/logistic regression), is robust to outliers and doesn't require feature scaling, and gives interpretable feature importances "for free." A strong baseline to try before reaching for more complex gradient boosting methods (XGBoost/LightGBM), which usually edge out Random Forest in raw accuracy but require more careful tuning.

## Strengths & Limitations
| Strengths | Limitations |
|---|---|
| Handles nonlinear relationships and feature interactions automatically | Less interpretable than a single decision tree or linear model |
| Robust to outliers and doesn't require feature scaling | Can be slow to predict with very large numbers of trees |
| Built-in OOB validation and feature importance | Tends to be outperformed by gradient boosting (XGBoost/LightGBM) on many tabular benchmarks |
| Resistant to overfitting relative to a single decision tree | Struggles with very high-dimensional sparse data (e.g. raw text) compared to linear models or SVM |
| Handles both classification and regression | Large model size/memory footprint with many/deep trees |

---
---
