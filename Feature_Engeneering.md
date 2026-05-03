# Full Feature Engineering Matrix — Best Version

## Purpose

This document is a reusable, production-oriented Feature Engineering (FE) decision system for machine learning projects.

The goal is not to apply every possible feature engineering step.  
The goal is to decide:

> EDA finding → Candidate feature engineering → Model-family impact → Leakage-safe validation → Final decision

---

# 0. Legend

## 0.1 Decision Values

| Value | Meaning |
|---|---|
| Required | Must do for correctness, safety, or valid modeling |
| Recommended | Usually useful and should be part of default pipeline |
| Conditional | Use only when the condition/trigger is present |
| CV-test | Try as candidate and keep only if validation improves |
| Avoid | Usually not useful or can harm the model |
| Native-supported | Use only if the model/library supports it directly |
| Model-dependent | Depends on algorithm/library implementation |
| Task-dependent | Depends on regression/classification/forecasting/ranking |
| Domain-dependent | Depends on real-world meaning of the feature |
| Leakage-risk | Can leak target/future information if done incorrectly |
| Train-only-fit | Fit/learn from training data only |

---

## 0.2 Model Family Abbreviations

| Abbreviation | Meaning |
|---|---|
| Linear | Linear Regression, Logistic Regression, Ridge, Lasso, ElasticNet |
| Tree | Decision Tree, Random Forest, ExtraTrees |
| Boosting | XGBoost, LightGBM, CatBoost, Gradient Boosting |
| KNN | K-Nearest Neighbors |
| SVM | Support Vector Machine |
| NN | Neural Network / MLP |
| NB | Naive Bayes, mostly for text/count features |

---

## 0.3 Priority Levels

| Priority | Meaning |
|---|---|
| P0 | Must handle before modeling |
| P1 | Strongly recommended |
| P2 | Candidate experiment |
| P3 | Advanced / optional / expert-level |

---

# 1. Global Feature Engineering Principles

| Rule | Why It Matters | Priority |
|---|---|---|
| Fit all preprocessing only on train | Prevents train-test leakage | P0 |
| Apply same pipeline to validation/test/inference | Ensures consistency | P0 |
| Never learn preprocessing stats from test data | Prevents optimistic validation | P0 |
| Use Pipeline / ColumnTransformer where possible | Reduces mismatch and leakage | P0 |
| Keep raw feature lineage | Makes debugging easier | P1 |
| Save preprocessing artifacts | Required for production inference | P0 |
| Separate EDA insight from FE decision | Avoids overengineering | P1 |
| Prefer simple baseline first | Needed to measure FE value | P0 |
| Add FE step only if it improves validation or solves a real risk | Prevents feature bloat | P1 |
| Use domain knowledge carefully | Domain features can help but can also leak | P1 |

---

# 2. Pipeline Fit / Transform Safety Rules

| Transformation | Fit Rule | Leakage Risk | Notes |
|---|---|---|---|
| Missing imputation | Fit imputation values on train only | Low-Medium | Median/mode/constant must come from train |
| Scaling | Fit scaler on train only | Medium | Mean/std/min/max from train only |
| Encoding | Fit categories/vocabulary on train only | Medium | Unknown categories must be handled |
| Rare grouping | Learn rare categories from train only | Medium | Do not inspect test frequency |
| Capping / winsorization | Learn bounds from train only | Medium | Percentiles from train only |
| Binning | Learn bin edges from train only | Medium | Quantile bins must be train-fitted |
| PCA / SVD | Fit components on train only | Medium | Apply learned projection to validation/test |
| Feature selection | Select using train/CV only | High | Never select based on test result |
| Target encoding | Out-of-fold encoding only | Very High | Never direct-fit on full train without CV split |
| Text vocabulary | Fit vocabulary on train only | Medium | Avoid test vocabulary leakage |
| Time lag / rolling feature | Use past data only | Very High | Future rows must not be used |
| Target transformation | Fit/transform y inside CV correctly | Medium | Inverse transform if evaluating original scale |

---

# 3. Leakage Safety Checklist

| Leakage Type | Example | Prevention |
|---|---|---|
| Target leakage | feature created after target event | remove or validate availability time |
| Temporal leakage | using future values for past prediction | strict time split and past-only rolling |
| Aggregation leakage | group stats computed using validation/test rows | compute group stats inside train folds only |
| Target encoding leakage | category encoded using its own target | use out-of-fold target encoding |
| Test-set leakage | preprocessing learned from test | fit only on train |
| Duplicate leakage | same entity appears in train and validation | use group split if needed |
| ID leakage | ID encodes target/order/source | drop or review |
| Post-event leakage | payment_status used to predict purchase | remove if unavailable at prediction time |
| Label-derived leakage | feature directly calculated from y | remove |
| Human annotation leakage | annotation done after target known | review feature timing |

---

# 4. Target Engineering

## 4.1 Target Engineering Matrix

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Drop rows with missing target | target missing | Required | Required | Required | Required | Required | Required | train/validation split after cleaning | Low | required for supervised learning | P0 |
| Target leakage check | any supervised task | Required | Required | Required | Required | Required | Required | before modeling | Very High | manual + feature timing review | P0 |
| Target type validation | target exists but unclear | Required | Required | Required | Required | Required | Required | before modeling | Low | confirm regression/classification/forecasting | P0 |
| Regression target outlier review | extreme y values | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | EDA before modeling | Medium | compare with/without treatment | P1 |
| Target clipping | extreme y likely data error | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train only | Medium | CV + domain confirmation | P2 |
| `log1p(y)` transform | positive skewed target / RMSLE | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | inside CV | Medium | use inverse transform for metric | P1 |
| Box-Cox / Yeo-Johnson target transform | non-normal regression target | CV-test | Usually avoid | CV-test | CV-test | CV-test | CV-test | inside CV | Medium | compare validation metric | P2 |
| Inverse transform prediction | transformed y used | Required | Required | Required | Required | Required | Required | after prediction | Medium | evaluate correctly | P0 |
| RMSLE/log-metric alignment | metric is RMSLE/log-error | Required | Required | Required | Required | Required | Required | metric-aware | Medium | evaluate in correct scale | P0 |
| Label encoding for classification target | classification target non-numeric | Required | Required | Required | Required | Required | Required | fit on train labels | Low | preserve label map | P0 |
| Class imbalance handling | imbalanced classes | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | train only | Medium | stratified/group/time-aware CV | P1 |
| Class weights | imbalance exists | Recommended | Conditional | Recommended | Avoid | Recommended | Recommended | train only | Low | compare CV metric | P1 |
| Oversampling / SMOTE | minority class too small | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | inside CV folds only | High | never before split | P2 |
| Undersampling | majority class too dominant | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | inside CV folds only | Medium | compare precision/recall/F1 | P2 |
| Threshold tuning | classification probability output | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | validation only | Medium | tune on validation, final test once | P1 |
| Probability calibration | probability quality matters | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | calibration split/CV | Medium | Brier/logloss/calibration curve | P2 |
| Stratified split | classification with enough samples | Required | Required | Required | Required | Required | Required | split stage | Low | preserve target ratio | P0 |
| Group split | entity leakage possible | Required | Required | Required | Required | Required | Required | split stage | High | use GroupKFold/GroupShuffleSplit | P0 |
| Time-based split | forecasting/time-dependent data | Required | Required | Required | Required | Required | Required | split stage | Very High | chronological validation | P0 |
| Multilabel target handling | multiple labels per row | Task-dependent | Task-dependent | Task-dependent | Task-dependent | Task-dependent | Task-dependent | before modeling | Medium | use multilabel metrics | P1 |

---

# 5. Numeric Feature Engineering

## 5.1 Numeric Cleaning and Type Handling

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Drop ID column | ID-like unique ratio | Required | Required | Required | Required | Required | Required | rule-based | Medium | confirm not meaningful signal | P0 |
| Drop leakage feature | target/post-event info | Required | Required | Required | Required | Required | Required | manual/timing review | Very High | remove before modeling | P0 |
| Drop constant feature | one unique value | Required | Required | Required | Required | Required | Required | train only | Low | no signal | P0 |
| Drop near-constant feature | dominant value very high | Conditional | Conditional | Conditional | Recommended | Recommended | Recommended | train only | Low | compare CV if uncertain | P1 |
| Fix numeric dtype | numeric stored as object | Required | Required | Required | Required | Required | Required | deterministic | Low | schema validation | P0 |
| Unit normalization | mixed units suspected | Required | Required | Required | Required | Required | Required | domain rule | Medium | domain validation | P0 |
| Invalid numeric value cleaning | impossible values | Required | Required | Required | Required | Required | Required | domain/range rule | Medium | validate range | P0 |
| Hidden sentinel handling | -999, 9999, unknown-coded values | Required | Required | Required | Required | Required | Required | train schema + domain | Medium | convert to missing/flag | P0 |
| Value range validation | known min/max constraints | Required | Required | Required | Required | Required | Required | domain rule | Medium | error/review table | P0 |
| Physical constraint validation | age < 0, percent > 100 | Required | Required | Required | Required | Required | Required | domain rule | Medium | clean or review | P0 |
| Numeric-code to categorical | numeric has category meaning | Required | Required | Required | Required | Required | Required | semantic rule | Medium | compare encoded version | P1 |
| Year-like to age/duration | year value represents event date | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | reference date rule | Medium | compare CV | P1 |
| Month-like to categorical/season | month has cyclic/seasonal meaning | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P2 |
| Count feature handling | integer counts, skew/zero-heavy | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | train only if transformed | Low | compare raw/log/flag | P1 |
| Ordinal numeric keep | numeric ranking/order | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | Low | preserve order | P1 |

---

## 5.2 Numeric Missingness

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Missing value imputation | missing numeric exists | Required | Conditional | Conditional | Required | Required | Required | fit on train | Medium | compare imputation strategies | P0 |
| Native missing handling | model supports missing | Avoid | Avoid | Native-supported | Avoid | Avoid | Avoid | model-level | Low | compare native vs imputed | P1 |
| Median imputation | skew/outliers/missing numeric | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | median from train | Medium | safe default | P0 |
| Mean imputation | symmetric numeric distribution | Conditional | Usually avoid | Usually avoid | Conditional | Conditional | Conditional | mean from train | Medium | compare CV | P2 |
| Constant imputation | missing has special meaning | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | constant chosen from train/domain | Medium | add missing flag if meaningful | P1 |
| Zero imputation for absence | zero means absence | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | domain-confirmed | Medium | compare zero vs missing category | P1 |
| Missing indicator flag | missingness may be predictive | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | learned from train columns | Medium | keep only if improves CV | P1 |
| Co-missing pattern feature | features missing together | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train pattern only | Medium | compare CV | P2 |

---

## 5.3 Numeric Special Values and Flags

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Zero-heavy flag | high zero percentage | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | threshold from train | Medium | test zero flag vs raw | P1 |
| Presence flag | absence/presence meaning | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | domain/EDA | Medium | compare CV | P1 |
| Negative value review | negative values exist | Required | Required | Required | Required | Required | Required | domain rule | Medium | decide valid vs invalid | P0 |
| Invalid negative cleaning | negative impossible | Required | Required | Required | Required | Required | Required | domain rule | Medium | clean/flag/drop | P0 |
| Special value flag | sentinel/zero/negative has meaning | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | train/domain | Medium | compare CV | P1 |
| Non-zero transformed feature | zero-heavy + skewed non-zero values | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train only | Medium | compare raw + flag + nonzero transform | P2 |

---

## 5.4 Numeric Outliers

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Outlier manual review | outliers detected | Required | Required | Required | Required | Required | Required | EDA/domain | Medium | determine error vs valid extreme | P0 |
| Blind outlier removal | outliers exist but not confirmed error | Avoid | Avoid | Avoid | Avoid | Avoid | Avoid | none | High | do not remove blindly | P0 |
| Remove error-only outliers | confirmed data error | Required | Required | Required | Required | Required | Required | domain-confirmed | Medium | document rule | P0 |
| Outlier capping / winsorization | extreme tail affects model | CV-test | Usually avoid | Conditional | Recommended | Recommended | CV-test | bounds from train | Medium | compare CV | P1 |
| Robust outlier handling | outliers valid but harmful | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | train only | Medium | robust scaler/cap/transform test | P1 |
| Robust z-score feature | non-normal distribution | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | median/MAD from train | Medium | compare CV | P2 |
| Outlier flag | extreme value may carry signal | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | bounds from train | Medium | compare CV | P2 |

---

## 5.5 Numeric Transformations

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Skewness handling | skewed numeric feature | Recommended | CV-test | CV-test | Recommended | Recommended | Recommended | train only | Low | compare raw vs transformed | P1 |
| `log1p` transform | non-negative skewed feature | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P1 |
| Signed log transform | positive and negative skewed values | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P2 |
| Sqrt transform | count-like non-negative skew | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P2 |
| Yeo-Johnson transform | skewed with negative values | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | fit on train | Medium | compare CV | P2 |
| Box-Cox transform | strictly positive numeric | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | fit on train | Medium | compare CV | P2 |
| Quantile transform | severe skew/outliers | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | fit on train | Medium | compare CV + interpretability | P2 |
| Rank transform | extreme outliers / monotonic signal | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | fit on train | Medium | compare CV | P2 |
| Binning | nonlinear or monotonic relationship | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | bin edges from train | Medium | compare CV | P2 |
| Monotonic binning | interpretable monotonic trend | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train only | Medium | compare CV + monotonicity | P2 |

---

## 5.6 Scaling and Normalization

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| StandardScaler | scale-sensitive model | Recommended | Avoid | Avoid | Required | Required | Recommended | fit on train | Medium | default for scale-sensitive models | P0 |
| RobustScaler | outliers + scale-sensitive model | CV-test | Avoid | Avoid | Recommended | Recommended | Recommended | fit on train | Medium | compare with StandardScaler | P1 |
| MinMaxScaler | bounded distance/NN model | Conditional | Avoid | Avoid | Conditional | Conditional | Conditional | fit on train | Medium | compare CV | P2 |
| Normalizer | vector direction matters | Conditional | Avoid | Avoid | Conditional | Conditional | Conditional | fit/transform train pipeline | Medium | useful for sparse/text-like vectors | P3 |
| Group-wise normalization | entity/group baseline differs | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train folds only | High | avoid group leakage | P2 |

---

## 5.7 Numeric Feature Creation

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Polynomial features | linear model underfits nonlinear relation | CV-test | Avoid | Avoid | Avoid | CV-test | Avoid | train pipeline | Low | compare CV + regularization | P2 |
| Spline features | smooth nonlinear relation | CV-test | Avoid | Avoid | Avoid | CV-test | Avoid | knots from train | Medium | compare CV | P2 |
| Interaction features | domain/EDA suggests interaction | CV-test | CV-test | CV-test | Usually avoid | CV-test | CV-test | train pipeline | Medium | compare CV | P2 |
| Ratio features | meaningful numerator/denominator | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic + safe denominator | Medium | compare CV | P1 |
| Difference features | meaningful subtraction | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P1 |
| Total / aggregate features | components form meaningful total | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | deterministic | Medium | compare CV + domain check | P1 |
| Domain flags | known business rule | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | deterministic | Medium | compare CV | P1 |
| Divide-by-zero safe ratio | ratio denominator can be zero | Required if ratio used | Required if ratio used | Required if ratio used | Required if ratio used | Required if ratio used | Required if ratio used | deterministic | Medium | add epsilon/flag | P0 |

---

## 5.8 Numeric Redundancy and Selection

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Multicollinearity check | linear/logistic model | Required | Avoid | Avoid | Conditional | Conditional | Conditional | train only | Low | correlation/VIF review | P1 |
| VIF check | interpretable linear model | Recommended | Avoid | Avoid | Avoid | Conditional | Avoid | train only | Low | remove/regularize | P2 |
| High correlation drop | redundant features | Conditional | Usually avoid | Usually avoid | Recommended | Recommended | Conditional | train only | Medium | compare CV | P1 |
| PCA | high-dimensional correlated numeric | CV-test | Avoid | Avoid | CV-test | CV-test | CV-test | fit on train | Medium | compare CV + explainability | P2 |
| Feature selection | many noisy features | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | inside CV | High | compare selected vs full | P1 |
| Permutation importance review | model trained | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | validation only | Low | inspect stable importance | P2 |

---

# 6. Categorical Feature Engineering

## 6.1 Categorical Cleaning

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Categorical dtype fix | object/category mismatch | Required | Required | Required | Required | Required | Required | schema rule | Low | validate dtype | P0 |
| Trim whitespace | inconsistent category values | Required | Required | Required | Required | Required | Required | deterministic | Low | category count before/after | P0 |
| Case normalization | same label different case | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | category count before/after | P1 |
| Category spelling normalization | typo variants | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | train/domain map | Medium | manual review | P1 |
| Synonym mapping | semantic equivalents | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | domain map | Medium | compare CV | P2 |
| Multi-label categorical split | multiple labels in one cell | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Medium | encode safely | P1 |
| Category schema validation | production categories known | Required | Required | Required | Required | Required | Required | train schema | Medium | unknown category handling | P0 |

---

## 6.2 Categorical Missingness

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Categorical missing handling | missing categorical exists | Required | Required | Required | Required | Required | Required | train/domain | Medium | compare strategy | P0 |
| Fill absence with `"None"` | missing means absence | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | domain-confirmed | Medium | compare CV | P1 |
| Fill unknown with `"Unknown"` | missing means unknown | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | Medium | compare CV | P1 |
| Mode imputation | missing likely random | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | mode from train | Medium | compare with Unknown | P2 |
| Missing-as-category | missing may carry signal | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | train only | Medium | compare CV | P1 |
| Missing indicator for category | missing signal suspected | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train only | Medium | compare CV | P2 |

---

## 6.3 Categorical Encoding

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Ordinal categorical mapping | natural order exists | Required | Required | Required | Required | Required | Required | manual/domain map | Medium | verify ordering | P0 |
| Nominal one-hot encoding | low/medium nominal category | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | fit categories on train | Medium | handle unknown | P0 |
| Binary encoding | binary category | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | fit mapping on train | Medium | preserve mapping | P0 |
| Rare category grouping | rare labels exist | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | train frequency only | Medium | compare CV | P1 |
| High-cardinality handling | many categories | Required | Required | Required | Required | Required | Required | train only | Medium-High | compare encoding methods | P0 |
| Frequency encoding | medium/high cardinality | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train frequency only | Medium | compare CV | P2 |
| Count encoding | medium/high cardinality | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train count only | Medium | compare CV | P2 |
| Target encoding | high-cardinality with target signal | CV-test | CV-test | CV-test | Avoid | CV-test | CV-test | out-of-fold only | Very High | strict OOF CV | P1 |
| Leave-one-out target encoding | high-card categorical | CV-test | CV-test | CV-test | Avoid | CV-test | CV-test | OOF/regularized | Very High | compare CV | P2 |
| WOE encoding | binary classification / credit-style | CV-test | CV-test | CV-test | Avoid | CV-test | CV-test | OOF or train folds | High | compare CV + monotonicity | P2 |
| Hashing encoding | very high cardinality / streaming | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare hash size | P2 |
| Embedding encoding | large data + NN/deep model | Usually avoid | Usually avoid | Conditional | Avoid | Avoid | CV-test | train only | Medium | compare CV | P3 |
| Native categorical handling | model supports native cat | Avoid | Avoid | Native-supported | Avoid | Avoid | Avoid | model-specific | Medium | compare native vs encoded | P1 |
| `handle_unknown="ignore"` | one-hot pipeline | Required | Required if one-hot | Required if one-hot | Required | Required | Required | encoder config | Medium | production safety | P0 |
| Infrequent category handling | rare/unseen categories likely | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | train only | Medium | compare CV | P1 |
| Sparse dummy control | many one-hot columns | Recommended | Conditional | Conditional | Recommended | Recommended | Conditional | encoder config | Low | memory/performance check | P1 |

---

## 6.4 Categorical Risk and Selection

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Drop categorical ID-like feature | unique ratio high | Required | Required | Required | Required | Required | Required | train/EDA | Medium | review before drop | P0 |
| Categorical leakage review | category target relation too strong | Required | Required | Required | Required | Required | Required | manual/timing | Very High | remove if leakage | P0 |
| Drop dominant weak category | near-constant and weak signal | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | train only | Low | compare CV | P2 |
| Categorical feature selection | many encoded columns | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | inside CV | High | compare CV | P1 |
| Categorical interaction feature | category combination useful | CV-test | CV-test | CV-test | Usually avoid | CV-test | CV-test | train only | Medium | compare CV | P2 |
| Category combination feature | two categories jointly meaningful | CV-test | CV-test | CV-test | Usually avoid | CV-test | CV-test | train only | Medium | compare CV | P2 |
| Hierarchical category encoding | categories have hierarchy | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | train/domain | Medium | compare hierarchy levels | P2 |
| Category drift monitoring | production category distribution changes | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | post-training monitor | Low | PSI/JS drift | P1 |

---

# 7. Datetime / Time-Series Feature Engineering

## 7.1 Datetime Cleaning and Parsing

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Parse datetime | datetime-like column exists | Required | Required | Required | Required | Required | Required | deterministic | Low | parse success rate | P0 |
| Datetime missing handling | missing datetime exists | Required | Required | Required | Required | Required | Required | train/domain | Medium | missing vs target review | P0 |
| Invalid datetime cleaning | invalid dates | Required | Required | Required | Required | Required | Required | deterministic/domain | Medium | invalid date table | P0 |
| Timezone normalization | mixed timezone data | Required | Required | Required | Required | Required | Required | deterministic | Medium | standardize timezone | P0 |
| Reference date validation | age/duration feature used | Required | Required | Required | Required | Required | Required | prediction-time rule | High | confirm availability | P0 |
| Datetime availability check | feature timestamp may occur after prediction | Required | Required | Required | Required | Required | Required | feature timing | Very High | remove if unavailable | P0 |

---

## 7.2 Datetime Component Features

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Year extraction | yearly trend exists | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Medium | compare CV | P2 |
| Month extraction | monthly/seasonal pattern | Recommended | Recommended | Recommended | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P1 |
| Day extraction | day-of-month pattern | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P2 |
| Day-of-week extraction | weekday pattern | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P1 |
| Hour extraction | hourly pattern | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P1 |
| Weekend flag | weekday/weekend behavior differs | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P2 |
| Quarter feature | quarterly seasonality | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P2 |
| Season feature | real-world seasonal behavior | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | deterministic | Low | compare CV | P2 |
| Holiday flag | holiday effects likely | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | external calendar | Medium | compare CV | P2 |
| Business day flag | business calendar matters | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | calendar rule | Medium | compare CV | P2 |
| Month start/end flag | financial/recurring cycle | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P2 |
| Pay cycle flag | payroll/finance behavior | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | domain rule | Medium | compare CV | P2 |

---

## 7.3 Duration, Recency, Lag and Rolling Features

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Age / duration feature | event date relative to reference | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | prediction-time reference | High | confirm no future info | P1 |
| Elapsed time feature | time since event matters | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | prediction-time rule | High | compare CV | P1 |
| Recency feature | days since last event | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only | Very High | time CV | P1 |
| Frequency feature | event count per period | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only | Very High | time CV | P1 |
| Time since previous event | sequence behavior | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only | Very High | time CV | P1 |
| Time until next event | future known only in some tasks | Usually avoid | Usually avoid | Usually avoid | Usually avoid | Usually avoid | Usually avoid | only if known at prediction | Very High | strict availability check | P0 |
| Lag features | forecasting/sequential data | Task-dependent | Task-dependent | Task-dependent | Task-dependent | Task-dependent | Task-dependent | past-only | Very High | time split validation | P1 |
| Group-wise lag | per user/product/store history | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only per group | Very High | Group/time CV | P1 |
| Rolling features | moving stats over time | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only window | Very High | time CV | P1 |
| Expanding window features | cumulative stats | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only | Very High | time CV | P2 |

---

## 7.4 Time Encoding and Split

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Cyclic encoding | hour/day/month cyclic pattern | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | deterministic | Low | compare with one-hot/raw | P1 |
| One-hot time category | low-cardinality time component | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | train categories | Medium | compare CV | P2 |
| Time binning | time intervals meaningful | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | bins from train/domain | Medium | compare CV | P2 |
| Time-safe train-test split | time-dependent data | Required | Required | Required | Required | Required | Required | split stage | Very High | chronological split | P0 |
| Temporal leakage check | any datetime/rolling feature | Required | Required | Required | Required | Required | Required | feature availability | Very High | manual + validation | P0 |

---

# 8. Text Feature Engineering

## 8.1 Text Cleaning and Normalization

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | NB | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Text missing handling | missing text exists | Required | Required | Required | Required | Required | Required | Required | deterministic | Medium | missing token vs empty | P0 |
| Empty string normalization | blank strings exist | Required | Required | Required | Required | Required | Required | Required | deterministic | Low | count before/after | P0 |
| Unicode normalization | mixed unicode forms | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | Low | normalize text | P1 |
| HTML tag removal | web/HTML text | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | inspect examples | P1 |
| URL normalization/removal | URLs in text | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Medium | compare keep/mask/remove | P2 |
| Email/phone masking | PII or leakage risk | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | High | mask PII | P1 |
| PII removal/masking | sensitive personal info | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | High | privacy + leakage review | P1 |
| Basic text cleaning | noisy text | Recommended | Conditional | Conditional | Recommended | Recommended | Recommended | Recommended | deterministic | Low | compare CV | P1 |
| Lowercasing | case not meaningful | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV if case matters | P2 |
| Whitespace normalization | inconsistent spacing | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | Low | required cleanup | P0 |
| Stopword handling | classical NLP | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | CV-test | train pipeline | Low | compare CV | P2 |
| Lemmatization/stemming | classical NLP | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | CV-test | deterministic/tool | Low | compare CV | P3 |
| Emoji handling | social/sentiment text | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | deterministic | Low | compare CV | P2 |
| Language-specific preprocessing | multilingual text | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | Conditional | language detection | Medium | separate/normalize language | P2 |

---

## 8.2 Text Statistical Features

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | NB | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Character count | length may carry signal | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Medium | review leakage | P2 |
| Word count | length may carry signal | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Medium | compare CV | P2 |
| Sentence count | long-form text | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P3 |
| Keyword flags | known important keywords | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic/domain | Medium | compare CV | P2 |
| Regex flags | known patterns | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic/domain | Medium | compare CV | P2 |
| Text leakage review | text may contain label info | Required | Required | Required | Required | Required | Required | Required | manual | Very High | inspect examples | P0 |
| Text ID-like drop | unique/free-ID text | Required | Required | Required | Required | Required | Required | Required | train/EDA | Medium | drop/review | P0 |

---

## 8.3 Text Vectorization

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | NB | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Vocabulary fit | TF-IDF/count vectorizer used | Required | Required | Required | Required | Required | Required | Required | fit on train | Medium | no test vocab fitting | P0 |
| Count vectorizer | classification/text count model | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | Recommended | fit on train | Medium | compare CV | P1 |
| TF-IDF | sparse text signal | Recommended | Conditional | Conditional | Conditional | Recommended | Conditional | Conditional | fit on train | Medium | compare CV | P1 |
| Char n-grams | typo/noisy short text | CV-test | Usually avoid | Usually avoid | CV-test | CV-test | CV-test | CV-test | fit on train | Medium | compare CV | P2 |
| Word n-grams | phrase signal | CV-test | Conditional | Conditional | CV-test | CV-test | CV-test | CV-test | fit on train | Medium | tune ngram range | P2 |
| `min_df` / `max_df` control | noisy/rare/common tokens | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | fit on train | Medium | tune via CV | P1 |
| `max_features` control | high-dimensional text | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | fit on train | Medium | compare sizes | P1 |
| TF-IDF + SVD | high-dimensional sparse text | CV-test | Conditional | Conditional | Recommended | Recommended | CV-test | Conditional | fit on train | Medium | compare dimension sizes | P2 |
| Embeddings | semantic text signal | CV-test | CV-test | CV-test | CV-test | CV-test | Recommended | Usually avoid | model/version fixed | Medium | compare with TF-IDF | P2 |
| Embedding pooling | transformer embeddings | Conditional | Conditional | Conditional | Conditional | Conditional | Recommended | Avoid | train/model config | Medium | compare mean/CLS/max | P3 |
| Embedding model versioning | embeddings used | Required | Required | Required | Required | Required | Required | Avoid | record model/version | Medium | reproducibility | P0 |
| Sparse text feature control | too many text dimensions | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | train pipeline | Medium | memory + CV | P1 |

---

# 9. Interaction / Aggregation / Domain Feature Engineering

| Feature Engineering Item | Condition / Trigger | Linear | Tree | Boosting | KNN | SVM | NN | Fit Rule | Leakage Risk | Validation Rule | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Numeric interaction | EDA/domain shows interaction | CV-test | Conditional | Conditional | Usually avoid | CV-test | CV-test | train pipeline | Medium | compare CV | P2 |
| Categorical interaction | category pair meaningful | CV-test | CV-test | CV-test | Usually avoid | CV-test | CV-test | train only | Medium | compare CV | P2 |
| Numeric x categorical segment feature | numeric effect differs by category | CV-test | CV-test | CV-test | Usually avoid | CV-test | CV-test | train only | Medium | compare CV | P2 |
| Ratio feature | meaningful business ratio | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Medium | compare CV | P1 |
| Difference feature | meaningful contrast | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | deterministic | Low | compare CV | P1 |
| Total / sum feature | components form total | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | deterministic | Medium | compare CV | P1 |
| Group aggregate feature | user/product/store aggregate | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | fold-safe only | Very High | group/time CV | P1 |
| Target-based aggregate | aggregate uses target | CV-test | CV-test | CV-test | Avoid | CV-test | CV-test | OOF only | Very High | strict CV | P1 |
| Historical aggregate | past behavior summary | CV-test | CV-test | CV-test | CV-test | CV-test | CV-test | past-only | Very High | time CV | P1 |
| Domain rule flag | known business rule | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | Domain-dependent | deterministic | Medium | compare CV | P1 |
| Constraint violation flag | inconsistent values | Recommended | Recommended | Recommended | Recommended | Recommended | Recommended | deterministic | Medium | detect data issue vs signal | P1 |

---

# 10. Model-wise Sensitivity Matrix

## 10.1 Main Model Sensitivities

| Question | Linear | Tree | Boosting | KNN | SVM | NN |
|---|---|---|---|---|---|---|
| Needs scaling? | Yes | No | No | Yes | Yes | Yes |
| Needs imputation? | Yes | Usually yes | Model-dependent | Yes | Yes | Yes |
| Can use native missing values? | No | Usually no | Model-dependent | No | No | No |
| Needs one-hot categorical? | Yes | Usually yes | Model-dependent | Yes | Yes | Usually yes/embedding |
| Can use native categorical? | No | Usually no | CatBoost/LightGBM/XGBoost-dependent | No | No | No |
| Sensitive to outliers? | High | Low | Low-Medium | High | High | Medium-High |
| Needs skew transform? | Often | Usually no | Usually no | Often | Often | Often |
| Sensitive to high dimension? | Medium | Medium | Medium | High | High | Medium-High |
| Sensitive to multicollinearity? | High | Low | Low | Medium | Medium | Medium |
| Benefits from feature selection? | Yes | Conditional | Conditional | Yes | Yes | Yes |
| Benefits from domain aggregates? | Yes | Yes | Yes | Conditional | Conditional | Yes |
| Benefits from interactions? | Yes | Trees learn many automatically | Boosting learns many automatically | Usually no | Sometimes | Sometimes |
| Interpretability impact of FE | High | Medium | Medium | Low | Low | Medium |

---

## 10.2 Model-wise Engineering Focus

| Model Family | Strongly Needs | Usually Avoids | Main Risk |
|---|---|---|---|
| Linear | imputation, encoding, scaling, skew handling, collinearity control, regularization | raw high-cardinality categories, unscaled features | poor fit on nonlinear patterns |
| Tree | clean types, leakage control, raw useful features, domain flags | unnecessary scaling, aggressive outlier removal | overfitting if noisy/leaky |
| Boosting | leakage control, categorical/missing handling, robust validation, strong domain features | blind feature explosion | overfitting leakage/noise |
| KNN | imputation, scaling, dimensionality reduction, noisy feature removal | high-dimensional sparse features | curse of dimensionality |
| SVM | scaling, encoding, feature selection, kernel-aware feature prep | high-dimensional noisy features without control | slow training / poor margins |
| NN | scaling, embeddings, dense clean inputs, stable numeric ranges | raw strings/categories | unstable training / overfitting |
| Naive Bayes | count/TF-IDF features, simple sparse representations | complex scaling-heavy features | independence assumption mismatch |

---

# 11. Task-wise Feature Engineering Rules

## 11.1 Regression

| FE Area | Recommended Action | Notes |
|---|---|---|
| Target skew | test log/Box-Cox/Yeo-Johnson | align with metric |
| Target outliers | review and possibly cap if data error | do not remove blindly |
| Numeric skew | test log/sqrt/Yeo-Johnson | especially for Linear/KNN/SVM |
| Scaling | required for Linear/KNN/SVM/NN | not needed for trees |
| Outlier handling | important for Linear/KNN/SVM | less important for trees |
| Metric alignment | transform y only if metric supports it | inverse transform predictions |

---

## 11.2 Classification

| FE Area | Recommended Action | Notes |
|---|---|---|
| Class imbalance | class weights/resampling/threshold tuning | do inside CV |
| Target encoding | OOF only | high leakage risk |
| Stratified split | use unless time/group split overrides | preserve class ratio |
| Calibration | test if probability quality matters | Platt/isotonic/CV calibration |
| Rare class | monitor recall/precision | do not optimize only accuracy |
| Categorical encoding | handle unknown categories | required for production |

---

## 11.3 Forecasting / Time-dependent Prediction

| FE Area | Recommended Action | Notes |
|---|---|---|
| Split | chronological split | no random split |
| Lag features | past-only | no future values |
| Rolling features | past-only shifted rolling | avoid current target leakage |
| Expanding features | past-only | useful for cumulative behavior |
| Calendar features | month, day-of-week, holiday | domain-specific |
| Target leakage | strict feature availability check | prediction-time validation |
| Drift | monitor temporal drift | compare periods |

---

## 11.4 Ranking / Recommendation

| FE Area | Recommended Action | Notes |
|---|---|---|
| User aggregates | historical user behavior only | past-only |
| Item aggregates | historical item behavior only | past-only |
| User-item interaction | count, recency, frequency | leakage-safe windows |
| Negative sampling | define carefully | affects metric |
| Temporal split | recommended | random split can leak |
| Embeddings | useful for users/items/text | version and validate |

---

# 12. Validation Strategy

## 12.1 Validation Rules by FE Type

| FE Type | Validation Requirement |
|---|---|
| Simple cleaning | required before baseline |
| Imputation | compare only if multiple plausible strategies |
| Scaling | required for scale-sensitive models |
| Encoding | compare encoding strategy for high-cardinality features |
| Target encoding | strict OOF CV |
| Outlier treatment | compare with raw version |
| Transformations | compare raw vs transformed |
| Interaction features | add incrementally |
| Text vectorization | tune vocabulary/ngram/max_features |
| Aggregates | fold-safe or time-safe computation |
| Feature selection | perform inside CV |
| Drift handling | validate on time/holdout split |

---

## 12.2 Recommended FE Experiment Plan

| Experiment Step | Purpose |
|---|---|
| 1. Minimal baseline | clean types, split, simple imputation, basic encoding |
| 2. Missing indicators | test missingness signal |
| 3. Numeric transforms | test skew/outlier handling |
| 4. Outlier strategy | test capping/robust scaling |
| 5. Categorical rare grouping | improve stability |
| 6. High-cardinality encoding | test frequency/target/hashing |
| 7. Datetime features | test temporal/calendar signal |
| 8. Text features | test TF-IDF/embeddings/text stats |
| 9. Interaction/domain features | test domain-specific signal |
| 10. Feature selection | reduce noise/dimension |
| 11. Model family comparison | ensure FE is not model-specific overfit |
| 12. Final ablation | remove unnecessary FE steps |

---

## 12.3 Ablation Table Template

| Experiment | Added FE | Model | CV Metric | Delta vs Baseline | Keep? | Notes |
|---|---|---|---|---|---|---|
| baseline | minimal cleaning | model_name | metric | 0.000 | yes | reference |
| exp_01 | missing indicators | model_name | metric | +0.012 | yes/no |  |
| exp_02 | log transforms | model_name | metric | -0.004 | no |  |
| exp_03 | rare grouping | model_name | metric | +0.008 | yes |  |

---

# 13. Final Feature Engineering Decision Table

| Feature | Type | EDA Finding | Candidate FE | Model Family | Fit Rule | Leakage Risk | Validation Result | Final Decision | Priority |
|---|---|---|---|---|---|---|---|---|---|
| age | numeric | skew/outliers | median impute + cap + scale | Linear/KNN/SVM | train only | medium | improved CV | keep transformed | P1 |
| city | categorical | high cardinality | rare grouping + OOF target encoding | Linear/Boosting | OOF train only | high | improved CV | keep OOF encoding | P1 |
| signup_date | datetime | temporal pattern | duration + month + day-of-week | all | prediction-time safe | high | improved CV | keep safe features | P1 |
| review_text | text | sparse signal | TF-IDF + SVD | Linear/KNN/SVM | train vocabulary only | medium | improved CV | keep SVD features | P2 |
| customer_id | ID | unique identifier | drop | all | deterministic | medium | not useful | drop | P0 |

---

# 14. Production / Inference Safety Checklist

| Check | Why It Matters | Priority |
|---|---|---|
| Same input schema in train and inference | prevents runtime errors | P0 |
| Data type validation | prevents silent bugs | P0 |
| Unknown category handling | production robustness | P0 |
| Missing value handling for new data | production robustness | P0 |
| Feature order preserved | model compatibility | P0 |
| Saved preprocessing pipeline | reproducibility | P0 |
| Encoder/scaler/imputer versioning | reproducibility | P1 |
| Text vectorizer vocabulary saved | reproducibility | P0 |
| Embedding model version saved | reproducibility | P1 |
| Range validation | catches invalid production data | P1 |
| Drift monitoring | detects model degradation | P1 |
| Target encoder leakage prevention | correctness | P0 |
| Time feature availability validation | prevents future leakage | P0 |
| Inference-time fallback values | handles unexpected inputs | P1 |
| Feature documentation | maintainability | P1 |

---

# 15. Feature Engineering Configuration Template

```yaml
feature_engineering_config:
  global:
    random_state: 42
    fit_on_train_only: true
    use_pipeline: true
    save_artifacts: true
    prevent_leakage: true

  missingness:
    numeric_default: median
    categorical_default: Unknown
    add_missing_indicator: cv_test
    high_missing_threshold: 0.40
    very_high_missing_threshold: 0.70

  numeric:
    drop_constant: true
    drop_id_like: true
    handle_sentinel_values: true
    outlier_method: iqr
    cap_outliers: cv_test
    skew_transform: cv_test
    scaling:
      linear: standard
      knn: standard_or_robust
      svm: standard
      tree: none
      boosting: none

  categorical:
    trim_whitespace: true
    normalize_case: conditional
    rare_grouping: true
    rare_threshold: 0.01
    unknown_category: Unknown
    one_hot_max_categories: 30
    high_cardinality_strategy:
      - rare_grouping
      - frequency_encoding_cv_test
      - target_encoding_oof_cv_test

  datetime:
    parse_datetime: true
    timezone_normalization: conditional
    feature_availability_check: true
    cyclic_encoding: cv_test
    lag_features: task_dependent
    rolling_features: task_dependent_past_only

  text:
    normalize_whitespace: true
    unicode_normalization: true
    pii_masking: conditional
    tfidf: cv_test
    embeddings: cv_test
    max_features: 50000
    svd: cv_test

  validation:
    baseline_first: true
    compare_with_ablation: true
    target_encoding_requires_oof: true
    feature_selection_inside_cv: true
    time_series_uses_time_split: true

  production:
    schema_validation: true
    unknown_category_handling: true
    save_pipeline: true
    monitor_drift: true
```

---

# 16. What to Add, Remove, and Re-label

## 16.1 Add

| Add Item | Reason |
|---|---|
| Condition / Trigger column | makes matrix decision-driven |
| Fit Rule column | prevents leakage |
| Leakage Risk column | critical for real ML |
| Validation Rule column | tells how to keep/drop FE |
| Production Safety section | required for inference |
| Experiment Plan | prevents random overengineering |
| Ablation Table | proves FE value |
| Task-wise Rules | regression/classification/forecasting differ |
| Advanced model families | Tree-based is too broad |
| Final FE Decision Table | converts planning into action |

---

## 16.2 Remove or Merge

| Current Type | Recommendation |
|---|---|
| Repeated leakage rows | move to global leakage section |
| Repeated invalid/sentinel cleaning | merge under invalid/special value handling |
| Zero-heavy flag + presence flag | combine under absence/presence signal where appropriate |
| Count encoding + frequency encoding | keep separate only if difference is clear |
| Year extraction + year-like conversion | organize under datetime-derived features |
| Keyword flags + regex flags | combine as rule-based text flags |
| Universal domain flags | make domain-dependent |
| Universal season feature | make domain-dependent |
| Universal target transform | make CV-test or metric-dependent |

---

## 16.3 Re-label from Yes to Conditional / CV-test

| Item | Better Label | Reason |
|---|---|---|
| Zero-heavy flag | CV-test | not always useful |
| Presence flag | Conditional | useful only if absence has meaning |
| Total / aggregate features | Domain-dependent | depends on feature semantics |
| Season feature | Domain-dependent | not always meaningful |
| One-hot time category | Conditional | can increase dimension |
| Missing-as-category | Conditional | only if missing is meaningful |
| Fill absence with None | Conditional | only if missing means absence |
| Target transformation | CV-test / metric-dependent | can hurt metric |
| Feature selection | Recommended for high-dimensional models | not always needed |
| Domain flags | Domain-dependent | must be meaningful |
| Outlier capping | CV-test | can remove real signal |

---

# 17. Final Rule

A strong feature engineering system should not ask:

> What transformations can I apply?

It should ask:

> What problem did EDA reveal, what transformation can solve it, can it be fitted safely, and does validation prove it helps?

Final principle:

> Feature engineering is not a checklist. It is a leakage-safe, validation-driven decision system.