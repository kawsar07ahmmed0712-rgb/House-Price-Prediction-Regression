# Advanced Feature Engineering Route for Ames House Price Prediction

**Goal:** build a disciplined feature engineering + modeling route to target **RMSLE / log-RMSE ≈ 0.10–0.11** and strong validation **R² ≈ 0.92–0.95+**.

**Primary metric:** RMSE on `log1p(SalePrice)`  
**Secondary metric:** R² on original or log target, used only as supporting metric.

This report is based on the uploaded univariate, bivariate, outlier, categorical, and multivariate analysis outputs.

---

## 1. Source Reports Used

- `univariate_full_manual_report.csv`
- `univariate_full_master_report.csv`
- `numeric_manual_report.csv`
- `categorical_manual_report.csv`
- `time_manual_report.csv`
- `bivariate_feature_best_signal_report.csv`
- `bivariate_priority_features.csv`
- `bivariate_final_action_summary.csv`
- `outlier_manual_action_report.csv`
- `known_grlivarea_outlier_review.csv`
- `EDA_House_Price_clean_structured(1).ipynb`

---

## 2. Target and Modeling Reality

### 2.1 Target metric

Kaggle House Prices is best optimized using log-scale error:

```text
y_train_model = log1p(SalePrice)
prediction = expm1(model_prediction)
metric = RMSE(log1p(y_true), log1p(y_pred))
```

### 2.2 Target ladder

| Level | CV RMSLE / log-RMSE | Meaning |
|---|---:|---|
| Baseline | 0.140–0.160 | beginner / simple preprocessing |
| Good | 0.120–0.130 | clean preprocessing + decent model |
| Strong | 0.110–0.120 | strong FE + tuned boosting/linear |
| Advanced | 0.105–0.110 | disciplined CV + blending |
| Stretch | ≈ 0.100 | difficult, requires ensemble and careful leakage control |

### 2.3 Final target

```text
Minimum acceptable: RMSLE < 0.130
Good target:       RMSLE < 0.120
Strong target:     RMSLE < 0.115
Stretch target:    RMSLE ≈ 0.100–0.110
```

---

## 3. Global Modeling Strategy

Final route:

```text
quality + size + location + garage + basement + bath + age + selected interactions
```

Feature engineering must be **CV-driven**, not only EDA-driven.

Every major feature block should be tested by:

```text
same CV split
same metric
same preprocessing state
before/after CV comparison
feature count tracking
train-vs-CV gap check
```

---

## 4. Data Preparation Rules

### 4.1 Strict train/test safety

| Rule | Decision |
|---|---|
| Target transform | `log1p(SalePrice)` |
| Train/test preprocessing | fit on train only, apply to test |
| Rare grouping | use train frequencies only |
| Target encoding | only out-of-fold, never full-train direct mean |
| Test unknown categories | map to `Rare` or handle with `handle_unknown=ignore` |
| Missing-as-absence | preserve as signal where applicable |
| Scaling | needed for linear models, not needed for tree models |
| Outliers | remove only confirmed harmful train outliers |

### 4.2 Direct drop

| Column | Decision |
|---|---|
| `Id` | drop from modeling, keep only for submission |

---

## 5. Outlier Strategy

### 5.1 Known high-risk outliers

The highest-priority manual outlier action is for the classic `GrLivArea` high-size but low-price rows.

Recommended action:

```text
Remove from training only:
- Id 524
- Id 1299
```

These rows have extremely high `GrLivArea`, high `OverallQual`, but unexpectedly low `SalePrice`. They can distort regression models.

### 5.2 What not to do

Do **not** blindly remove/cap all upper-tail values.

Many upper-tail area/garage/lot values support higher prices and represent valid expensive houses.

### 5.3 Outlier route

| Feature group | Action |
|---|---|
| `GrLivArea` extreme low-price rows | remove only known problematic train rows |
| `LotArea`, `TotalBsmtSF`, `1stFlrSF`, `GarageArea`, `MasVnrArea` | keep upper tail, review only if model unstable |
| Sparse zero-heavy features | use indicators, not broad removal |
| Linear models | use robust scaling/log transform |
| Tree models | keep raw upper-tail values unless CV worsens |

---

## 6. Missingness and Absence Handling

### 6.1 Fill categorical absence with `None`

Use `None` for features where missing means absence:

```text
Alley
MasVnrType
BsmtQual
BsmtCond
BsmtExposure
BsmtFinType1
BsmtFinType2
FireplaceQu
GarageType
GarageFinish
GarageQual
GarageCond
PoolQC
Fence
MiscFeature
```

### 6.2 Fill numeric absence with `0`

Use `0` where missing/zero means no component:

```text
MasVnrArea
GarageArea
GarageCars
TotalBsmtSF
BsmtFinSF1
BsmtFinSF2
BsmtUnfSF
BsmtFullBath
BsmtHalfBath
PoolArea
MiscVal
```

### 6.3 Important presence indicators

High-priority indicators:

```text
HasGarage
HasBasement
HasFireplace
HasPool
HasMasVnr
HasGarageYrBlt
```

Medium/review indicators:

```text
HasWoodDeck
HasOpenPorch
HasScreenPorch
Has3SsnPorch
HasLowQualFinSF
HasMiscFeature
HasFence
HasAlley
LotFrontageMissing
```

---

## 7. Feature Engineering Priority Map

### 7.1 P0 — must create first

These are the first engineered features for both tree and linear routes:

| New feature | Formula / logic | Priority |
|---|---|---|
| `TotalSF` | `TotalBsmtSF + 1stFlrSF + 2ndFlrSF` | very high |
| `LivingPlusBasementSF` | `GrLivArea + TotalBsmtSF` | high/test |
| `TotalBath` | `FullBath + 0.5*HalfBath + BsmtFullBath + 0.5*BsmtHalfBath` | high |
| `HouseAge` | `YrSold - YearBuilt` | high |
| `RemodAge` | `YrSold - YearRemodAdd` | high |
| `GarageAge` | `YrSold - GarageYrBlt`; no garage handled separately | high |
| `IsNewHouse` | `YearBuilt == YrSold` or low age threshold | medium/high |
| `IsRemodeled` | `YearRemodAdd > YearBuilt` | medium |
| `HasGarage` | garage component exists | high |
| `HasBasement` | basement component exists | high |
| `HasFireplace` | `Fireplaces > 0` or `FireplaceQu != None` | medium/high |
| `HasPool` | `PoolArea > 0` or `PoolQC != None` | sparse review |
| `HasMasVnr` | `MasVnrArea > 0` or `MasVnrType != None` | medium |

### 7.2 P1 — strongest interaction features

Create after the clean baseline:

| New feature | Formula / logic | Decision |
|---|---|---|
| `OverallQual_x_TotalSF` | `OverallQual * TotalSF` | must test, highest priority |
| `OverallQual_x_GrLivArea` | `OverallQual * GrLivArea` | high |
| `OverallQual_x_TotalBath` | `OverallQual * TotalBath` | high |
| `KitchenQual_Ord_x_GrLivArea` | `KitchenQual ordinal * GrLivArea` | test |
| `ExterQual_Ord_x_GrLivArea` | `ExterQual ordinal * GrLivArea` | test |
| `BsmtQual_Ord_x_TotalBsmtSF` | `BsmtQual ordinal * TotalBsmtSF` | test |
| `GarageFinish_Ord_x_GarageArea` | `GarageFinish ordinal * GarageArea` | test |
| `GarageQual_Ord_x_GarageArea` | `GarageQual ordinal * GarageArea` | test |

### 7.3 P2 — component score features

| New feature | Components | Decision |
|---|---|---|
| `ExteriorQualityScore` | `ExterQual + ExterCond` | high |
| `KitchenQualityScore` | `KitchenQual` | high but may duplicate original |
| `BasementQualityScore` | `BsmtQual + BsmtCond + BsmtExposure` | high |
| `BasementFinishScore` | `BsmtFinType1 + BsmtFinType2` | medium |
| `GarageQualityScore` | `GarageQual + GarageCond + GarageFinish` | high |
| `LuxuryQualityScore` | fireplace/pool/garage/kitchen selected quality | review only |

### 7.4 P3 — location and building interaction review

These are not automatic. Test only after strong baseline.

| Candidate | Decision |
|---|---|
| `NeighborhoodPriceTier` | review; OOF/CV-safe only |
| `Neighborhood_x_TotalSFBin` | review; may overfit |
| `Neighborhood_x_GrLivAreaBin` | review; may overfit |
| `MSSubClass_x_TotalSFBin` | strong candidate, must CV-test |
| `MSSubClass_x_1stFlrSFBin` | strong candidate, must CV-test |
| `HouseStyle_x_TotalSFBin` | review |
| `Foundation_x_TotalBsmtSFBin` | review |

### 7.5 P4 — lower-priority ratio features

Do not prioritize initially.

| Feature | Decision |
|---|---|
| `BasementRatio` | low priority; test only |
| `FirstFloorRatio` | low priority |
| `SecondFloorRatio` | low priority |
| `GarageAreaPerCar` | low priority |
| `LivingAreaLotRatio` | low priority |
| `BathPerRoom` | low priority |
| `AreaPerRoom` | optional review |

Reason: ratio features often lose important absolute-size information.

---

## 8. Ordinal Encoding Map

Use manual ordinal maps. Missing/absence gets `0`; unknown/unseen gets `-1` when needed.

| Feature | Suggested order |
|---|---|
| `ExterQual`, `ExterCond`, `KitchenQual`, `HeatingQC`, `GarageQual`, `GarageCond`, `BsmtQual`, `BsmtCond`, `FireplaceQu`, `PoolQC` | `None=0, Po=1, Fa=2, TA=3, Gd=4, Ex=5` |
| `BsmtExposure` | `None=0, No=1, Mn=2, Av=3, Gd=4` |
| `BsmtFinType1`, `BsmtFinType2` | `None=0, Unf=1, LwQ=2, Rec=3, BLQ=4, ALQ=5, GLQ=6` |
| `GarageFinish` | `None=0, Unf=1, RFn=2, Fin=3` |
| `Functional` | `Sal=1, Sev=2, Maj2=3, Maj1=4, Mod=5, Min2=6, Min1=7, Typ=8` |
| `PavedDrive` | `N=0, P=1, Y=2` |
| `Fence` | `None=0, MnWw=1, GdWo=2, MnPrv=3, GdPrv=4` |
| `LotShape` | `Reg=0, IR1=1, IR2=2, IR3=3` or reverse if CV supports |
| `LandSlope` | `Gtl=0, Mod=1, Sev=2` |
| `OverallQual`, `OverallCond` | keep as ordered numeric scores |

For non-monotonic ordinal features, keep ordinal encoding first, but optionally compare OHE during model validation.

---

## 9. Nominal Encoding Map

### 9.1 Rare grouping + one-hot

Apply rare grouping before OHE:

```text
MSSubClass
Neighborhood
HouseStyle
RoofStyle
Exterior1st
Exterior2nd
```

Default rare rule:

```text
category count < 15 OR category frequency < 1%
```

For `Neighborhood`, use slightly careful grouping because it is high-signal. Do not over-group premium neighborhoods.

### 9.2 Normal one-hot

```text
MSZoning
LandContour
LotConfig
Condition1
Condition2
BldgType
Foundation
MasVnrType
Heating
CentralAir
Electrical
GarageType
MiscFeature
SaleType
SaleCondition
```

### 9.3 Binary / low-cardinality

```text
Street
Alley
Utilities
CentralAir
PavedDrive
```

Use binary encoding or OHE. Keep `Street` and `Utilities` as low-priority review.

### 9.4 Target encoding

Only optional and only OOF/CV-safe:

```text
Neighborhood
MSSubClass
Exterior1st
Exterior2nd
Foundation
```

Never compute full-train category target means and map them directly before CV.

---

## 10. Numeric Transformation Map

### 10.1 Tree-based models

Tree-based models do not need scaling and usually do not need log transforms. Use:

```text
raw numeric
imputed numeric
presence indicators
selected engineered interactions
```

### 10.2 Linear models

Linear models need more careful preprocessing:

```text
impute
log1p skewed features
scale numeric features
one-hot nominal features
regularization
```

Strong log/skew candidates:

```text
LotArea
GrLivArea
TotalSF
TotalBsmtSF
1stFlrSF
2ndFlrSF
MasVnrArea
BsmtFinSF1
BsmtFinSF2
WoodDeckSF
OpenPorchSF
EnclosedPorch
ScreenPorch
3SsnPorch
PoolArea
MiscVal
```

For zero-heavy features, use both:

```text
HasFeature indicator + log1p(raw value)
```

---

## 11. Model-Specific Dataset Design

### 11.1 Tree-based route

Applicable models:

```text
DecisionTreeRegressor
RandomForestRegressor
ExtraTreesRegressor
GradientBoostingRegressor
HistGradientBoostingRegressor
XGBoost
LightGBM
CatBoost
```

Tree route:

| Step | Treatment |
|---|---|
| Numeric | raw + median/zero imputation |
| Scaling | no |
| Skew transforms | usually no; optional for linear comparison only |
| Nominal categorical | OHE or native categorical for CatBoost |
| Ordinal categorical | manual ordinal encoding |
| Missing absence | `None`/0 + indicators |
| Interactions | selected P0/P1/P2 features |
| Redundancy | keep correlated features initially |
| Feature selection | model importance/permutation/CV |
| Sparse rare features | keep first, prune only after CV |

Tree-specific recommendation:

```text
Use tree dataset as the main dataset.
Keep most original variables.
Add strong FE.
Do not aggressively remove correlated features.
```

### 11.2 Linear route

Applicable models:

```text
Ridge
Lasso
ElasticNet
BayesianRidge
KernelRidge
SVR optional
```

Linear route:

| Step | Treatment |
|---|---|
| Numeric | impute + log1p skewed + scale |
| Nominal categorical | rare-group + one-hot |
| Ordinal categorical | ordinal encode or OHE if non-monotonic |
| Interactions | selected only; too many interactions cause variance |
| Redundancy | reduce with regularization; optional pruning |
| Scaling | required |
| Outliers | more sensitive; remove confirmed outliers |
| Target | log1p |
| Feature count | monitor carefully |

Linear-specific recommendation:

```text
Use Ridge/ElasticNet as strong baseline and ensemble member.
Do not rely only on tree models.
```

### 11.3 CatBoost route

CatBoost can use categorical features directly:

| Step | Treatment |
|---|---|
| Nominal categorical | pass as categorical if using CatBoost |
| Ordinal categorical | can pass as ordered numeric or categorical; test both |
| Missing | fill categorical with `None`, numeric with 0/median |
| Scaling | no |
| Interactions | selected FE only |
| Rare grouping | optional but still useful for stability |

---

## 12. Validation and Experiment Protocol

### 12.1 CV setup

Use one fixed CV plan for all experiments:

```text
5-fold KFold for experiments
10-fold or repeated CV for final confirmation
shuffle=True
fixed random_state
metric = RMSE(log1p target)
```

### 12.2 Keep/drop rule

Keep a new feature if:

```text
CV improves by >= 0.001–0.003
OR fold stability improves
OR high-price-tail error improves
```

Drop/reject if:

```text
CV worsens consistently
train-CV gap increases sharply
feature is unstable across folds
feature adds too many sparse columns without gain
```

### 12.3 Ablation order

Recommended experimental order:

| Round | Add / Change | Target |
|---|---|---|
| 0 | baseline preprocessing only | <0.135 |
| 1 | outlier removal + clean dtype + missing handling | <0.130 |
| 2 | P0 core features | <0.125 |
| 3 | P1 quality-size interactions | <0.120 |
| 4 | component scores | <0.118 |
| 5 | tuned Ridge/ElasticNet + XGB/LGBM/CatBoost | <0.115 |
| 6 | location/building interactions | <0.112 |
| 7 | blend/stack | 0.100–0.110 |

---

## 13. Full Column Treatment Master Table

This table gives the planned behavior for each original column.

| Column        | Component                             | Behavior                              | Main decision                        | Tree-based treatment                                                                 | Linear treatment                                                                    | Notes                                                                                               |
|:--------------|:--------------------------------------|:--------------------------------------|:-------------------------------------|:-------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------|
| Id            | id/target/other                       | row identifier, no predictive meaning | drop from modeling                   | drop                                                                                 | drop                                                                                | never use as feature                                                                                |
| MSSubClass    | building_structure                    | nominal categorical feature           | keep + safe unknown handling         | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.50 (strong); treat as categorical; rare/unseen safe handling                               |
| MSZoning      | location_zoning                       | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.33 (moderate)                                                                              |
| LotFrontage   | area_size                             | continuous numeric feature            | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale; add indicator                                                        | signal 0.41 (moderate)                                                                              |
| LotArea       | area_size                             | regular numeric                       | keep; transform if needed            | impute safely; keep raw numeric; no scaling required                                 | impute; test log1p/skew transform; scale                                            | signal 0.46 (moderate)                                                                              |
| Street        | utilities_access                      | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.04 (very weak); near-constant; low-priority review                                         |
| Alley         | outdoor_porch                         | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.15 (weak); missing means no alley; fill None                                               |
| LotShape      | id/target/other                       | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.28 (weak)                                                                                  |
| LandContour   | location_zoning                       | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.16 (very weak)                                                                             |
| Utilities     | utilities_access                      | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.01 (very weak); near-constant; low-priority drop review                                    |
| LotConfig     | location_zoning                       | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.14 (very weak)                                                                             |
| LandSlope     | location_zoning                       | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.05 (very weak)                                                                             |
| Neighborhood  | location_zoning                       | nominal categorical feature           | keep + rare grouping                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.74 (strong); core location signal; rare-group; OOF target encoding only as experiment      |
| Condition1    | location_zoning                       | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.18 (weak)                                                                                  |
| Condition2    | location_zoning                       | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.10 (very weak); rare/dominated; low-priority review                                        |
| BldgType      | building_structure                    | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.19 (weak)                                                                                  |
| HouseStyle    | building_structure                    | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.29 (weak)                                                                                  |
| OverallQual   | quality_condition, building_structure | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.83 (strong); core quality driver; use in quality × size interactions                       |
| OverallCond   | quality_condition, building_structure | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.35 (moderate)                                                                              |
| YearBuilt     | time_age                              | year-like                             | prefer engineered feature            | prefer engineered age feature; raw year optional for tree validation                 | use age/remodel age; scale; raw year only if CV improves                            | prefer HouseAge                                                                                     |
| YearRemodAdd  | time_age                              | year-like                             | prefer engineered feature            | prefer engineered age feature; raw year optional for tree validation                 | use age/remodel age; scale; raw year only if CV improves                            | prefer RemodAge and IsRemodeled                                                                     |
| RoofStyle     | building_structure                    | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.24 (weak)                                                                                  |
| RoofMatl      | building_structure                    | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.18 (weak); rare/dominated; low-priority review                                             |
| Exterior1st   | exterior_material                     | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.39 (moderate)                                                                              |
| Exterior2nd   | exterior_material                     | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.39 (moderate)                                                                              |
| MasVnrType    | exterior_material                     | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.43 (moderate)                                                                              |
| MasVnrArea    | area_size, exterior_material          | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.42 (moderate)                                                                              |
| ExterQual     | quality_condition, exterior_material  | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.69 (strong)                                                                                |
| ExterCond     | quality_condition, exterior_material  | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.15 (very weak)                                                                             |
| Foundation    | building_structure                    | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.51 (strong)                                                                                |
| BsmtQual      | quality_condition, basement           | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.67 (strong)                                                                                |
| BsmtCond      | quality_condition, basement           | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.39 (moderate)                                                                              |
| BsmtExposure  | basement                              | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.37 (moderate)                                                                              |
| BsmtFinType1  | basement                              | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.44 (moderate)                                                                              |
| BsmtFinSF1    | basement                              | regular numeric                       | keep + transform/check               | impute safely; keep raw numeric; no scaling required                                 | impute; test log1p/skew transform; scale                                            | signal 0.30 (weak)                                                                                  |
| BsmtFinType2  | basement                              | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.37 (moderate)                                                                              |
| BsmtFinSF2    | basement                              | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.07 (very weak)                                                                             |
| BsmtUnfSF     | basement                              | continuous numeric feature            | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.19 (very weak)                                                                             |
| TotalBsmtSF   | area_size, basement                   | regular numeric                       | keep; transform if needed            | impute safely; keep raw numeric; no scaling required                                 | impute; test log1p/skew transform; scale                                            | signal 0.60 (strong); basement size; use in TotalSF and basement interaction                        |
| Heating       | utilities_access                      | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.12 (very weak)                                                                             |
| HeatingQC     | quality_condition                     | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.44 (moderate)                                                                              |
| CentralAir    | utilities_access                      | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.25 (weak)                                                                                  |
| Electrical    | utilities_access                      | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.24 (weak)                                                                                  |
| 1stFlrSF      | area_size                             | regular numeric                       | keep; transform if needed            | impute safely; keep raw numeric; no scaling required                                 | impute; test log1p/skew transform; scale                                            | signal 0.58 (moderate)                                                                              |
| 2ndFlrSF      | area_size                             | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.29 (weak)                                                                                  |
| LowQualFinSF  | area_size                             | rare/sparse numeric feature           | keep indicator; raw feature optional | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.20 (weak)                                                                                  |
| GrLivArea     | area_size                             | regular numeric                       | keep; transform if needed            | impute safely; keep raw numeric; no scaling required                                 | impute; test log1p/skew transform; scale                                            | signal 0.73 (strong); core area driver; remove two known extreme low-price outliers before training |
| BsmtFullBath  | basement, bath_room_count             | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.23 (weak)                                                                                  |
| BsmtHalfBath  | basement, bath_room_count             | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.02 (very weak)                                                                             |
| FullBath      | bath_room_count                       | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.64 (strong)                                                                                |
| HalfBath      | bath_room_count                       | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.34 (weak)                                                                                  |
| BedroomAbvGr  | bath_room_count                       | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.24 (weak)                                                                                  |
| KitchenAbvGr  | bath_room_count                       | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.16 (very weak)                                                                             |
| KitchenQual   | quality_condition                     | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.68 (strong)                                                                                |
| TotRmsAbvGrd  | bath_room_count                       | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.55 (strong)                                                                                |
| Functional    | quality_condition, utilities_access   | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.13 (very weak)                                                                             |
| Fireplaces    | luxury_sparse                         | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.52 (moderate)                                                                              |
| FireplaceQu   | quality_condition, luxury_sparse      | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.34 (moderate)                                                                              |
| GarageType    | garage                                | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.50 (strong)                                                                                |
| GarageYrBlt   | garage, time_age                      | year-like with missing                | replace with engineered features     | prefer engineered age feature; raw year optional for tree validation                 | use age/remodel age; scale; raw year only if CV improves                            | prefer GarageAge + HasGarageYrBlt                                                                   |
| GarageFinish  | garage                                | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.52 (strong)                                                                                |
| GarageCars    | garage                                | discrete count feature                | keep                                 | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.70 (strong); garage capacity; keep even with GarageArea correlation                        |
| GarageArea    | area_size, garage                     | continuous numeric feature            | keep; transform if needed            | impute safely; keep raw numeric; no scaling required                                 | impute; scale                                                                       | signal 0.65 (strong); garage size; keep and test GarageScore                                        |
| GarageQual    | quality_condition, garage             | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.41 (strong)                                                                                |
| GarageCond    | quality_condition, garage             | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.41 (strong)                                                                                |
| PavedDrive    | outdoor_porch, utilities_access       | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.23 (weak)                                                                                  |
| WoodDeckSF    | outdoor_porch                         | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.35 (weak)                                                                                  |
| OpenPorchSF   | outdoor_porch                         | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.48 (moderate)                                                                              |
| EnclosedPorch | outdoor_porch                         | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.22 (moderate)                                                                              |
| 3SsnPorch     | outdoor_porch                         | rare/sparse numeric feature           | keep indicator; raw feature optional | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.16 (weak)                                                                                  |
| ScreenPorch   | outdoor_porch                         | zero-heavy numeric feature            | keep + binary indicator              | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.10 (very weak)                                                                             |
| PoolArea      | luxury_sparse                         | rare/sparse numeric feature           | keep indicator; raw feature optional | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.44 (strong); rare luxury numeric; keep indicator; raw optional                             |
| PoolQC        | quality_condition, luxury_sparse      | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.67 (strong); rare luxury feature; keep as sparse signal and HasPool                        |
| Fence         | outdoor_porch                         | ordinal categorical feature           | keep                                 | fill absence/unknown if needed; ordinal encode; keep original order                  | ordinal encode; optionally compare OHE if non-monotonic; scale not needed after OHE | signal 0.23 (weak); missing means no fence; fill None                                               |
| MiscFeature   | luxury_sparse                         | nominal categorical feature           | keep                                 | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.11 (weak); rare feature; fill None; raw category optional                                  |
| MiscVal       | luxury_sparse                         | rare/sparse numeric feature           | keep indicator; raw feature optional | impute safely; keep raw numeric; add presence/missing indicator; no scaling required | impute; test log1p/skew transform; scale; add indicator                             | signal 0.10 (very weak); rare numeric; indicator optional                                           |
| MoSold        | time_age                              | sale month                            | keep                                 | prefer engineered age feature; raw year optional for tree validation                 | use age/remodel age; scale; raw year only if CV improves                            |                                                                                                     |
| YrSold        | time_age                              | sale year                             | keep                                 | prefer engineered age feature; raw year optional for tree validation                 | use age/remodel age; scale; raw year only if CV improves                            |                                                                                                     |
| SaleType      | sale_transaction                      | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.37 (moderate); availability/leakage review; Kaggle OK, deployment review                   |
| SaleCondition | sale_transaction                      | nominal categorical feature           | keep but review                      | fill missing as designed; rare-group; one-hot or CatBoost native categorical         | rare-group + one-hot; drop_first optional for pure linear, Ridge can keep all       | signal 0.37 (moderate); availability/leakage review; Kaggle OK, deployment review                   |

---

## 14. High-Priority Feature List from Bivariate + Multivariate Analysis

Top signal features and engineered drivers:

```text
OverallQual
Neighborhood
GrLivArea
GarageCars
ExterQual
KitchenQual
BsmtQual
HouseAge
GarageArea
FullBath
TotalBsmtSF
GarageAge
RemodAge
1stFlrSF
TotRmsAbvGrd
Fireplaces
GarageFinish
Foundation
GarageType
MSSubClass
OpenPorchSF
LotArea
PoolArea
HeatingQC
BsmtFinType1
MasVnrType
MasVnrArea
```

Important note:

```text
PoolQC / PoolArea are sparse, but they may explain luxury/high-price houses.
Do not drop them only because they are rare.
```

---

## 15. Low-Priority Drop Review Features

These should not be dropped immediately. They should go through CV/model-importance review.

```text
Utilities
Street
Condition2
RoofMatl
LowQualFinSF
KitchenAbvGr
EnclosedPorch
3SsnPorch
ScreenPorch
MiscFeature
MiscVal
Alley
Fence
PoolQC
PoolArea
```

Decision:

```text
Keep in first full tree model.
Use indicators for sparse features.
Try removal only after model importance + CV.
```

---

## 16. Final Feature Engineering Build Order

### Build 1 — clean baseline

```text
drop Id
log1p target
remove two known GrLivArea outliers
correct MSSubClass as categorical
fill missing values
ordinal encode
one-hot nominal
```

### Build 2 — core FE

```text
TotalSF
TotalBath
HouseAge
RemodAge
GarageAge
IsNewHouse
IsRemodeled
HasGarage
HasBasement
HasFireplace
HasPool
HasMasVnr
ExteriorMatched
```

### Build 3 — high-priority interactions

```text
OverallQual_x_TotalSF
OverallQual_x_GrLivArea
OverallQual_x_TotalBath
KitchenQual_Ord_x_GrLivArea
ExterQual_Ord_x_GrLivArea
BsmtQual_Ord_x_TotalBsmtSF
GarageFinish_Ord_x_GarageArea
```

### Build 4 — component scores

```text
BasementQualityScore
BasementFinishScore
GarageQualityScore
ExteriorQualityScore
KitchenQualityScore
```

### Build 5 — review-sensitive advanced features

```text
NeighborhoodPriceTier
Neighborhood_x_TotalSFBin
MSSubClass_x_TotalSFBin
HouseStyle_x_TotalSFBin
Foundation_x_TotalBsmtSFBin
```

Only keep if CV improves.

### Build 6 — final model ensemble

```text
Ridge / ElasticNet
XGBoost
LightGBM
CatBoost
GradientBoosting
weighted blend
optional stacking with OOF predictions
```

---

## 17. Final Decision Tree / Tree-Based Plan

Use this dataset first as the main route.

### Tree dataset features

```text
Original numeric raw features
Ordinal encoded quality features
Rare-grouped one-hot nominal features
Core engineered features
Selected interactions
Presence indicators
Sparse luxury indicators
```

### Tree action rules

| Case | Decision |
|---|---|
| Correlated features | keep initially |
| Scaling | not needed |
| Log transform | usually not needed |
| Missing absence | use `None`/0 + indicator |
| Rare categories | group rare categories |
| Sparse features | keep first, prune by CV |
| Feature selection | permutation importance + CV |
| Outlier sensitivity | remove only two confirmed rows |

Tree models to train:

```text
ExtraTreesRegressor
RandomForestRegressor
GradientBoostingRegressor
HistGradientBoostingRegressor
XGBoost
LightGBM
CatBoost
```

Expected role:

```text
Tree models capture nonlinear effects and feature interactions.
They may not need every manually created interaction, but selected interactions can still help.
```

---

## 18. Final Linear Model Plan

Use this as a separate preprocessing route.

### Linear dataset features

```text
log1p/skew-transformed numeric
scaled numeric
rare-grouped one-hot categorical
ordinal encoded ordered features
selected interaction features
presence indicators
```

### Linear action rules

| Case | Decision |
|---|---|
| Numeric skew | log1p / Yeo-Johnson test |
| Scaling | required |
| Outliers | more sensitive; remove confirmed outliers |
| Correlated features | Ridge/ElasticNet handles; Lasso may select |
| Nominal categorical | one-hot |
| Ordinal categorical | ordinal encode, optionally OHE non-monotonic |
| Interaction terms | add selected, not all |
| High-cardinality interactions | avoid unless CV improves |

Linear models to train:

```text
Ridge
Lasso
ElasticNet
BayesianRidge
KernelRidge
SVR optional
```

Expected role:

```text
Regularized linear models are strong on this dataset and often improve ensemble stability.
```

---

## 19. Final Ensemble Route

0.10 target likely requires ensemble.

Recommended blending order:

1. Build stable Ridge/ElasticNet baseline.
2. Build tuned XGBoost.
3. Build tuned LightGBM.
4. Build CatBoost if available.
5. Compare CV and residual patterns.
6. Use weighted blend.
7. If needed, use OOF stacking with Ridge meta-model.

Do not trust a blend unless OOF/CV improves.

---

## 20. Final Checklist Before Modeling

### Data and target

- [ ] Drop `Id`
- [ ] `log1p(SalePrice)`
- [ ] Remove `Id 524` and `Id 1299` from train only
- [ ] Verify train/test columns aligned

### Missing and type

- [ ] Fill absence categorical with `None`
- [ ] Fill absence numeric with `0`
- [ ] Impute `LotFrontage`
- [ ] Treat `MSSubClass` as categorical
- [ ] Create safe unknown handling

### Feature engineering

- [ ] Create P0 features
- [ ] Create P1 interactions
- [ ] Create P2 component scores
- [ ] Rare group nominal features
- [ ] Add presence indicators
- [ ] Keep low-priority features until CV review

### Model routes

- [ ] Tree route dataset
- [ ] Linear route dataset
- [ ] Same CV split
- [ ] Log-RMSE metric
- [ ] Feature ablation report
- [ ] Ensemble only after individual models stable

---

## 21. Final Conclusion

To reach **RMSLE ≈ 0.10–0.11**, the best route is not to add every possible feature. The best route is:

```text
clean preprocessing
+ confirmed outlier removal
+ strong component features
+ selected quality-size interactions
+ safe categorical encoding
+ separate tree and linear pipelines
+ strict CV ablation
+ tuned boosting + regularized linear models
+ final blending/stacking
```

The strongest feature engineering direction is:

```text
quality + size + location + garage + basement + bath + age + selected interactions
```

Only `Id` should be dropped immediately. Other weak, sparse, or redundant features should be kept first and removed only after cross-validation confirms that removal improves the model.
