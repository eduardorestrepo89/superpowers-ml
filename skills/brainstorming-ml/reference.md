# ML/DS Brainstorming Reference

Lookup tables for the five-stage question sequence in `SKILL.md`. Pull specific rows into the conversation as they become relevant — don't dump whole tables on the user unprompted.

## Problem Types & Target (Y) Shapes

| Problem type | Y shape | Example | Typical metric family |
|---|---|---|---|
| Binary classification | 0/1 label | churn, fraud flag | precision/recall, F1, ROC-AUC, PR-AUC |
| Multiclass classification | one label from N | topic tagging | macro/micro F1, log loss |
| Multilabel classification | vector of labels | tag prediction | Hamming loss, subset accuracy |
| Regression | continuous value | price, demand | RMSE, MAE, MAPE, R² |
| Ordinal regression | ordered category | star rating | QWK, MAE on ranks |
| Clustering | group assignment, no ground truth | customer segmentation | silhouette, Davies-Bouldin, Calinski-Harabasz |
| Ranking | relative order | search results | NDCG, MAP, MRR |
| Forecasting | future value(s)/sequence | sales forecast | RMSE, MAPE, sMAPE (backtested) |
| Anomaly detection | outlier score/flag | fraud, defect detection | precision@k, AUC if labeled, else domain validation |

## Supervised vs Unsupervised vs Semi-supervised — decision questions

- Ground-truth labels for most of the data? → **supervised**
- No labels, looking for structure? → **unsupervised** (clustering, dim reduction, anomaly detection)
- A few labels, mostly unlabeled? → **semi-supervised** (pseudo-labeling, label propagation, self-training) or weak supervision
- Building representations without labels, then fine-tuning downstream? → **self-supervised**

## Data Requirements Checklist

- **Grain:** one row per entity, per event, per time step, or relational (multiple joined tables)?
- **Data types present:** numeric, categorical, text, image, audio, time series, graph
- **Volume:** rows/columns available vs. needed — classical models want roughly ≥10x samples per feature; deep learning wants much more
- **Label provenance:** human-annotated, proxy/heuristic, or logged outcome? How noisy?
- **Typical feature families by domain:**
  - Tabular/business: demographics, behavioral aggregates, recency/frequency/monetary, categorical encodings
  - Time series: lags, rolling stats, calendar/holiday flags, seasonality indicators
  - Text: TF-IDF/embeddings, length, sentiment, entity counts
  - Images: pretrained CNN/ViT embeddings, color histograms
  - Graph: degree, centrality, community features, node embeddings

## EDA Checklist

- **Structural:** shape, dtypes, missingness map, duplicate rows, cardinality of categoricals
- **Target:** distribution/class balance, outliers in target
- **Univariate:** histograms, boxplots per feature
- **Bivariate vs. target:** scatter/boxplot, group means
- **Variance:** near-zero-variance feature check (drop candidates)
- **Linearity:** scatterplots + residual plots vs. target; Ramsey RESET test; Rainbow test
- **Correlation / multicollinearity:** Pearson (linear), Spearman/Kendall (monotonic), correlation heatmap, Variance Inflation Factor (VIF)
- **Hypothesis tests:**
  - t-test / ANOVA — do group means differ (e.g., does a feature differ by class)?
  - Chi-square — association between categoricals, or categorical feature vs. categorical target
  - Kolmogorov–Smirnov — compare two distributions (e.g., train vs. test drift)
  - Shapiro-Wilk / D'Agostino — normality check (informs parametric test choice, scaling)
  - Mann-Whitney U / Kruskal-Wallis — non-parametric group comparisons
- **If time series, also run:**
  - Stationarity: Augmented Dickey-Fuller (ADF), KPSS
  - Seasonality: STL decomposition, ACF/PACF plots, periodogram
  - Trend: Mann-Kendall trend test
  - Autocorrelation: Ljung-Box test

## Feature Engineering

### Dimensionality reduction — when to use which

| Method | Use when |
|---|---|
| PCA | Linear, continuous, correlated numeric features; fast, interpretable variance explained |
| Truncated SVD | Like PCA but works on sparse data (e.g., TF-IDF matrices) |
| t-SNE / UMAP | Visualization/exploration of nonlinear structure — not for production model inputs (non-parametric, unstable across runs) |
| LDA | Supervised, classification only, maximizes class separability |
| Autoencoders | Nonlinear structure, high-dim data (images), needs more data |
| Feature hashing | High-cardinality categoricals, memory-constrained settings |

### Feature selection — when to use which

| Method class | Examples | Trade-off |
|---|---|---|
| Filter (fast, model-agnostic) | correlation threshold, mutual information, chi-square, ANOVA F-test | Cheap, ignores feature interactions |
| Wrapper (model-in-the-loop) | RFE, forward/backward stepwise selection | More accurate, expensive to run |
| Embedded (selection during training) | L1/Lasso, tree-based importance (RF, gradient boosting gain/SHAP) | Good balance, tied to one model family |

## Data Pipeline Design

- **Split strategy** — pick based on data structure to avoid leakage:
  - Random split: i.i.d. tabular data
  - Stratified split: imbalanced classification (preserve class ratios)
  - Group split: multiple rows per entity (e.g., per user) — split by entity, never by row
  - Time-based split: time series, or any temporal leakage risk — train on past, validate/test on future
- **Preprocessing order:** clean → impute → encode categoricals → scale/normalize → engineer features → select features. Fit all transforms on train only; apply to val/test.
- **Leakage checks:** any feature computed using future/target information? any preprocessing fit on the full dataset instead of train only?
- **Reproducibility:** fixed random seeds, versioned dataset snapshots, documented row counts per split.

## Model Selection & Evaluation

### Metric selection — notes

- Imbalanced classification → prefer PR-AUC/F1 over raw accuracy (accuracy is misleading when one class dominates)
- Regression with outliers/skew → prefer MAE or Huber loss over RMSE (RMSE over-weights large errors)
- Business cost is asymmetric (false negatives cost more than false positives, or vice versa) → pick a metric or threshold tuned to that cost, not a generic default
- Clustering has no ground truth → validate with silhouette/Davies-Bouldin *and* domain sanity checks (do the clusters mean something to a stakeholder?)
- Forecasting → always backtest with rolling-origin cross-validation, never a random split

### Candidate models by problem type

| Problem type | Models to consider | Key hyperparameters |
|---|---|---|
| Classification | Logistic Regression, Random Forest, Gradient Boosting (XGBoost/LightGBM/CatBoost), SVM, k-NN, small neural nets | tree depth/leaves, learning rate, n_estimators, regularization strength, C/gamma (SVM), k (k-NN) |
| Regression | Linear/Ridge/Lasso, Random Forest, Gradient Boosting, SVR, small neural nets | same families as above; alpha (Ridge/Lasso) |
| Clustering | k-Means, DBSCAN, Hierarchical (Agglomerative), Gaussian Mixture Models | k (k-Means/GMM), eps/min_samples (DBSCAN), linkage (Hierarchical) |
| Time series | ARIMA/SARIMA, Prophet, exponential smoothing, gradient boosting with lag features, LSTM/temporal models | p/d/q orders (ARIMA), seasonality period, lag window size |

### Hyperparameter tuning approaches

| Approach | Use when |
|---|---|
| Grid search | Small search space, few hyperparameters |
| Random search | Larger space, more efficient than grid for the same budget |
| Bayesian optimization (Optuna, Hyperopt, scikit-optimize) | Expensive-to-train models, want a smart/adaptive search |
| Early stopping | Iterative models (boosting, neural nets) — stop on validation loss plateau instead of a fixed iteration count |
