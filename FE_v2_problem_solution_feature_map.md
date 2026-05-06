# FE v2 Full Action Plan — House Price Prediction

**Project:** House Price Prediction Regression  
**Current public best:** ~0.12030  
**Target range:** 0.111–0.115 public score  
**Main decision:** Same FE দিয়ে আরও RandomForest / tuning / stacking করলে বড় gain আসার সম্ভাবনা কম। FE v2 দরকার।

---

## 0. Executive Summary

FE v1 খারাপ ছিল না। FE v1 যথেষ্ট ভালো ছিল বলেই Ridge/ElasticNet/ensemble প্রায় `0.120` public score পর্যন্ত গেছে। কিন্তু `0.111–0.115` range-এ যেতে হলে FE v1-এর limitation solve করতে হবে।

### FE v1-এর মূল সমস্যা

1. `Neighborhood` signal strongest হলেও price-aware numeric encoding ছিল না।
2. Location × quality interaction যথেষ্ট strong ছিল না।
3. Location × size interaction যথেষ্ট strong ছিল না।
4. Low-price এবং high-price tail houses-এ model বেশি ভুল করছে।
5. Sparse categorical features অনেক noise দিচ্ছে।
6. Age/remodel effect আছে, কিন্তু quality/location context ছাড়া under-modeled।
7. Local CV public score estimate ঠিকমতো করছে না।
8. Outlier handling একটাই version; controlled variants test করা হয়নি।
9. Linear model best হলেও linear-specific pruning/feature stability এখনো enough নয়।

### FE v2-এর মূল লক্ষ্য

```text
Current FE pipeline
+ leakage-safe OOF target encoding
+ Neighborhood price tier
+ location-quality interaction
+ location-size interaction
+ refined age/remodel interaction
+ luxury / low-quality tail features
+ sparse-feature cleanup
+ stronger validation discipline
```

---

## 1. Evidence-Based Diagnosis

### 1.1 Model evidence

Current submissions show that model-side tuning alone score wall ভাঙছে না।

| Model / Submission Type | Result Meaning |
|---|---|
| Ridge baseline | public best family so far |
| Tuned linear | public score improve করেনি |
| CatBoost / tree model | weaker |
| Master ensemble | very small improvement |
| Ridge + forest blend | tiny improvement only |

Conclusion:

```text
Modeling-side improvement প্রায় exhausted for current FE.
Next gain must come from better feature representation.
```

### 1.2 EDA evidence

EDA showed these groups as highly important:

```text
OverallQual
Neighborhood
GrLivArea
GarageCars / GarageArea
TotalBsmtSF / 1stFlrSF
KitchenQual
ExterQual
BsmtQual
HouseAge / RemodAge / GarageAge
```

Important interaction patterns were visible:

```text
OverallQual × GrLivArea
OverallQual × TotalSF
OverallQual × GarageArea
OverallQual × TotalBath
GrLivArea × TotalBsmtSF
GarageCars × GarageArea
HouseAge × OverallQual
Neighborhood × GrLivArea
KitchenQual × GrLivArea
BsmtQual × TotalBsmtSF
GarageType × GarageArea
GarageFinish × GarageArea
```

So FE v2 should not randomly create features. It should target these high-signal interactions.

---

## 2. FE v1 Problems and Required Fixes

## Problem 1 — Neighborhood signal was too shallow

### Current issue

`Neighborhood` was treated mostly as categorical/OHE/rare grouping. But neighborhood has strong price-level information.

Problem:

```text
Neighborhood exists, but market value of neighborhood is not represented as a smooth numeric signal.
```

### Why this hurts

Linear models cannot fully understand that:

```text
Same OverallQual + same GrLivArea
but different Neighborhood
can mean very different SalePrice.
```

### FE v2 fix

Create leakage-safe OOF target encoding:

```text
Neighborhood_TE
Neighborhood_Median_TE
Neighborhood_Count
Neighborhood_Rank_TE
NeighborhoodPriceTier
```

### Implementation rule

Train data:

```text
Use OOF target encoding only.
Do not calculate target mean using the same row's target.
```

Test data:

```text
Use full-train smoothed target encoding.
Unknown category -> global mean fallback.
```

---

## Problem 2 — Location × quality interaction missing/weak

### Current issue

`OverallQual` is very strong. `Neighborhood` is also strong. But their interaction was not represented enough.

### FE v2 fix

Create:

```text
OverallQual_x_Neighborhood_TE
ExterQual_Ord_x_Neighborhood_TE
KitchenQual_Ord_x_Neighborhood_TE
BsmtQual_Ord_x_Neighborhood_TE
GarageFinish_Ord_x_Neighborhood_TE
```

### Priority

Very high.

### Expected effect

This should help model understand:

```text
High quality in premium area -> stronger price premium
High quality in low-price area -> limited premium
```

---

## Problem 3 — Location × size interaction missing/weak

### Current issue

`GrLivArea`, `TotalSF`, and `Neighborhood` are all strong. But market-adjusted size signal was weak.

### FE v2 fix

Create:

```text
Neighborhood_TE_x_GrLivArea
Neighborhood_TE_x_TotalSF
Neighborhood_TE_x_1stFlrSF
Neighborhood_TE_x_TotalBsmtSF
Neighborhood_TE_x_GarageArea
Neighborhood_TE_x_TotalBath
```

### Priority

Very high.

### Expected effect

This helps distinguish:

```text
Large house in cheap neighborhood
vs
Large house in premium neighborhood
```

---

## Problem 4 — Low-price and high-price tail errors

### Current issue

Model performs well in middle price range but weaker at low and high price tails.

### FE v2 fix

Create tail-aware features:

```text
LuxuryScore
LuxuryAreaScore
LowQualityRiskScore
PremiumNeighborhoodLargeHouse
CheapNeighborhoodLargeHouse
OldLowQualityHouse
NewHighQualityHouse
```

### Candidate definitions

```text
LuxuryScore =
    OverallQual
  + ExterQual_Ord
  + KitchenQual_Ord
  + BsmtQual_Ord
  + GarageFinish_Ord
  + FireplaceQu_Ord

LuxuryAreaScore =
    LuxuryScore × log1p(TotalSF)

LowQualityRiskScore =
    low OverallQual
  + old HouseAge
  + low ExterQual
  + low KitchenQual
  + no garage
  + no basement

PremiumNeighborhoodLargeHouse =
    1 if NeighborhoodPriceTier is premium and TotalSF is high

CheapNeighborhoodLargeHouse =
    1 if NeighborhoodPriceTier is low and TotalSF is high

OldLowQualityHouse =
    1 if HouseAge high and OverallQual low

NewHighQualityHouse =
    1 if HouseAge low and OverallQual high
```

---

## Problem 5 — Age/remodel effect under-modeled

### Current issue

Already created:

```text
HouseAge
RemodAge
GarageAge
IsRemodeled
IsNewHouse
```

But age effect depends on quality/location.

### FE v2 fix

Create:

```text
OverallQual_x_HouseAge
OverallQual_x_RemodAge
Neighborhood_TE_x_HouseAge
Neighborhood_TE_x_RemodAge
QualityAgeIndex
QualityRemodIndex
AgePenaltyByQuality
```

### Candidate formulas

```text
QualityAgeIndex = OverallQual / (HouseAge + 1)

QualityRemodIndex = OverallQual / (RemodAge + 1)

AgePenaltyByQuality = HouseAge / (OverallQual + 1)
```

### Priority

High.

---

## Problem 6 — Garage and basement features need stronger score representation

### Current issue

Garage and basement features are useful, but their area × quality combined effect can be stronger.

### FE v2 fix

Create:

```text
GarageScore = GarageCars × GarageArea

GarageQualityAreaScore =
    GarageArea × GarageQual_Ord

GarageFinishAreaScore =
    GarageArea × GarageFinish_Ord

BasementQualityAreaScore =
    TotalBsmtSF × BsmtQual_Ord

BasementExposureAreaScore =
    TotalBsmtSF × BsmtExposure_Ord

FinishedBasementRatio =
    (BsmtFinSF1 + BsmtFinSF2) / (TotalBsmtSF + 1)
```

### Priority

Medium-high.

---

## Problem 7 — Sparse categorical noise

### Current issue

Some features have too many missing/rare categories and may hurt linear generalization.

Examples:

```text
PoolQC
MiscFeature
Alley
Fence
Condition2
RoofMatl
Utilities
Street
```

These features can be useful as absence/presence flags, but detailed categories may be noisy.

### FE v2 fix

Do not blindly keep all sparse dummies. Convert or simplify.

Recommended actions:

| Feature | FE v2 Action |
|---|---|
| `PoolQC` | keep `HasPool`; drop detailed PoolQC dummies unless signal strong |
| `PoolArea` | keep `HasPool`, maybe keep numeric PoolArea only for tree; review for linear |
| `MiscFeature` | keep `HasMiscFeature`; drop detailed rare dummies |
| `MiscVal` | review/drop for linear unless transformed |
| `Alley` | keep `HasAlley`; detailed categories optional |
| `Fence` | keep `HasFence`; detailed categories optional |
| `Utilities` | drop or keep only if train/test variation exists |
| `Street` | drop or keep only if variation exists |
| `Condition2` | rare-group strongly or drop detailed dummies |
| `RoofMatl` | rare-group strongly or drop detailed dummies |
| `LowQualFinSF` | keep binary `HasLowQualFinSF`; numeric review |
| `KitchenAbvGr` | review/drop for linear if noisy |
| `3SsnPorch` | combine into `TotalPorchSF`; optional binary |
| `ScreenPorch` | combine into `TotalPorchSF`; optional binary |
| `EnclosedPorch` | combine into `TotalPorchSF`; optional binary |

---

## Problem 8 — Redundant numeric features

### Current issue

Several numeric features are highly correlated. Regularized linear models can handle this, but too many redundant interaction features may increase public overfit.

Known high-correlation groups:

```text
GarageCars ↔ GarageArea
GrLivArea ↔ TotRmsAbvGrd
TotalBsmtSF ↔ 1stFlrSF
```

### FE v2 fix

Do not automatically drop strong base features. Instead:

For tree dataset:

```text
Keep most base features.
Tree models can handle redundancy better.
```

For linear dataset:

```text
Keep strong base features:
- GrLivArea
- TotalSF
- TotalBsmtSF
- 1stFlrSF
- GarageCars
- GarageArea

But review weak redundant features:
- TotRmsAbvGrd if GrLivArea + BedroomAbvGr + FullBath already present
- GarageQual/GarageCond sparse dummies if GarageScore exists
- many rare one-hot columns
```

---

## Problem 9 — Target encoding not implemented though it was a candidate

### Current issue

Categorical encoding hints already indicated target encoding as a candidate, but FE v1 did not include leakage-safe target encoding.

### FE v2 fix

Add OOF target encoding for high-cardinality / high-signal categorical features.

Recommended target-encoding features:

```text
Neighborhood
MSSubClass
Exterior1st
Exterior2nd
Foundation
HouseStyle
SaleCondition
GarageType
```

Recommended combined target encoding:

```text
Neighborhood_OverallQual
Neighborhood_MSSubClass
Neighborhood_HouseStyle
MSSubClass_OverallQual
Exterior1st_OverallQual
Foundation_OverallQual
```

---

## 3. Exact Feature Creation Plan

## 3.1 Target encoding features

### Must create

```text
Neighborhood_TE
Neighborhood_Median_TE
Neighborhood_Count
NeighborhoodPriceTier
NeighborhoodPriceTier_Ord
```

### Strong candidates

```text
MSSubClass_TE
Exterior1st_TE
Exterior2nd_TE
Foundation_TE
GarageType_TE
HouseStyle_TE
SaleCondition_TE
```

### Combined target encoding candidates

```text
Neighborhood_OverallQual_TE
Neighborhood_MSSubClass_TE
Neighborhood_HouseStyle_TE
MSSubClass_OverallQual_TE
Exterior1st_OverallQual_TE
Foundation_OverallQual_TE
```

### Smoothing formula

Use smoothed mean:

```text
encoded_value =
    (category_count × category_mean + smoothing × global_mean)
    / (category_count + smoothing)
```

Recommended smoothing:

```text
smoothing = 10 to 30
```

---

## 3.2 Location interaction features

```text
OverallQual_x_Neighborhood_TE
ExterQual_Ord_x_Neighborhood_TE
KitchenQual_Ord_x_Neighborhood_TE
BsmtQual_Ord_x_Neighborhood_TE
GarageFinish_Ord_x_Neighborhood_TE

Neighborhood_TE_x_GrLivArea
Neighborhood_TE_x_TotalSF
Neighborhood_TE_x_1stFlrSF
Neighborhood_TE_x_TotalBsmtSF
Neighborhood_TE_x_GarageArea
Neighborhood_TE_x_TotalBath
```

---

## 3.3 Quality-size interaction features

```text
OverallQual_x_TotalSF
OverallQual_x_GrLivArea
OverallQual_x_TotalBath
OverallQual_x_GarageScore
OverallQual_x_BasementQualityAreaScore
KitchenQual_Ord_x_GrLivArea
ExterQual_Ord_x_TotalSF
BsmtQual_Ord_x_TotalBsmtSF
```

Some of these may already exist in v1. In v2, keep them and add missing ones.

---

## 3.4 Age/remodel features

```text
HouseAge
RemodAge
GarageAge

IsRemodeled
IsNewHouse
IsVeryOldHouse
IsOldButRemodeled

OverallQual_x_HouseAge
OverallQual_x_RemodAge
Neighborhood_TE_x_HouseAge
Neighborhood_TE_x_RemodAge

QualityAgeIndex
QualityRemodIndex
AgePenaltyByQuality
```

---

## 3.5 Garage features

```text
HasGarage
GarageScore
GarageAreaPerCar
GarageQualityAreaScore
GarageFinishAreaScore
GarageAge
GarageAgeMissingFlag
```

Important:

```text
GarageYrBlt missing should not remain as missing.
Use GarageAge plus HasGarage/GarageAgeMissingFlag.
```

---

## 3.6 Basement features

```text
HasBasement
BasementScore
BasementQualityAreaScore
BasementExposureAreaScore
FinishedBasementRatio
UnfinishedBasementRatio
TotalBsmtSF_x_BsmtQual_Ord
TotalBsmtSF_x_BsmtExposure_Ord
```

---

## 3.7 Bath/room features

```text
TotalBath
BathPerRoom
BathPerBedroom
BedroomPerRoom
RoomSizeAvg = GrLivArea / (TotRmsAbvGrd + 1)
```

Review carefully because ratio features may be noisy.

---

## 3.8 Porch/outdoor features

```text
TotalPorchSF
HasPorch
HasWoodDeck
OutdoorSF = WoodDeckSF + OpenPorchSF + EnclosedPorch + 3SsnPorch + ScreenPorch
OutdoorSF_x_Neighborhood_TE
OutdoorSF_x_OverallQual
```

---

## 3.9 Tail/luxury/risk features

```text
LuxuryScore
LuxuryAreaScore
PremiumNeighborhoodFlag
LowPriceNeighborhoodFlag
PremiumNeighborhoodLargeHouse
CheapNeighborhoodLargeHouse
OldLowQualityHouse
NewHighQualityHouse
NoGaragePenaltyFlag
NoBasementPenaltyFlag
```

---

## 4. Feature Removal / Review Plan

## 4.1 Drop or simplify for both tree and linear

| Feature | Action |
|---|---|
| `Utilities` | drop if near-constant |
| `Street` | drop if near-constant |
| raw `Id` | never use as feature |
| duplicate generated columns | drop |
| columns with all-zero train or test | drop |
| columns with one unique value | drop |

---

## 4.2 Convert to binary/presence, then drop detailed sparse dummies

| Original Feature | Keep | Drop/Review |
|---|---|---|
| `PoolQC` | `HasPool` | detailed PoolQC dummies |
| `PoolArea` | optional numeric for tree | review/drop for linear |
| `MiscFeature` | `HasMiscFeature` | detailed rare dummies |
| `MiscVal` | optional log1p / binary | review/drop for linear |
| `Alley` | `HasAlley` | detailed category dummies optional |
| `Fence` | `HasFence` | detailed category dummies optional |
| `Condition2` | rare-grouped / binary unusual flag | detailed sparse dummies |
| `RoofMatl` | rare-grouped / premium roof flag | detailed sparse dummies |

---

## 4.3 Linear dataset-specific review/drop list

These are not automatic drops. Validate with CV.

```text
PoolArea
MiscVal
LowQualFinSF
KitchenAbvGr
3SsnPorch
ScreenPorch
EnclosedPorch
Condition2 rare dummies
RoofMatl rare dummies
Utilities dummy
Street dummy
very rare Neighborhood dummies after TE exists
very rare Exterior dummies after TE exists
```

Recommended rule:

```text
If feature is sparse, weak, and not useful in repeated CV:
drop from linear dataset.
```

---

## 4.4 Keep important base features

Do not drop these blindly:

```text
OverallQual
Neighborhood OHE or grouped version
GrLivArea
TotalSF
TotalBsmtSF
1stFlrSF
GarageCars
GarageArea
TotalBath
HouseAge
RemodAge
ExterQual_Ord
KitchenQual_Ord
BsmtQual_Ord
GarageFinish_Ord
```

---

## 5. FE v2 Dataset Variants

## FE v2 A — Location Encoding Core

Includes:

```text
Current FE
+ Neighborhood_TE
+ NeighborhoodPriceTier
+ Neighborhood × quality
+ Neighborhood × size
```

Use this first.

---

## FE v2 B — Location + Age + Component Scores

Includes:

```text
FE v2 A
+ quality-age features
+ garage score
+ basement score
+ luxury/risk features
```

Use if v2 A improves robust CV.

---

## FE v2 C — Linear Cleanup Version

Includes:

```text
FE v2 B
+ sparse dummy pruning
+ low-variance removal
+ near-constant removal
```

Use only after v2 B.

---

## FE v2 D — Outlier Strategy Variant

Compare:

```text
D1: current outlier removal
D2: no outlier removal
D3: only extreme GrLivArea outliers removed
D4: residual-based suspicious row review
```

Do not delete many rows blindly.

---

## 6. Validation Plan

Current local CV was too optimistic. FE v2 must use stronger validation.

### Required

```text
10-fold CV
seeds: 42, 2024, 777
metric: RMSE on log1p(SalePrice)
```

### Model shortlist for FE validation

```text
Ridge
ElasticNet
KernelRidge
SVR
GradientBoosting
XGBoost
CatBoost / LightGBM if stable
```

### Success rule

FE v2 is accepted only if:

```text
Repeated CV improves by at least 0.003–0.005
Prediction distribution remains sane
Low/high price quantile error improves
Public-safe Ridge/Kernel blend improves over 0.12030
```

---

## 7. Modeling Strategy After FE v2

### Linear/kernel models become priority

```text
Ridge
ElasticNet
KernelRidge polynomial
SVR RBF
BayesianRidge
HuberRegressor
```

### Boosting models as diversity

```text
XGBoost
GradientBoosting
CatBoost
LightGBM
```

### Low priority

```text
RandomForest
ExtraTrees
```

Forest helped slightly but did not break the 0.12 wall.

### Blend strategy

Use log-space weighted average:

```text
blend_log =
    w1 * Ridge_log
  + w2 * ElasticNet_log
  + w3 * KernelRidge_log
  + w4 * SVR_log
  + w5 * Boosting_log
```

Do not use aggressive stacking first.

---

## 8. Final Prioritized To-Do List

## Must Do

```text
1. Add OOF Neighborhood target encoding
2. Add Neighborhood price tier
3. Add Neighborhood × OverallQual
4. Add Neighborhood × GrLivArea / TotalSF
5. Add quality × size features
6. Add quality × age/remodel features
7. Add luxury/risk tail features
8. Simplify sparse categorical features
9. Run repeated CV
```

## Should Do

```text
1. Add combined target encoding:
   Neighborhood_OverallQual_TE
   Neighborhood_MSSubClass_TE

2. Add garage/basement quality-area score

3. Create linear-specific pruned dataset

4. Compare outlier variants
```

## Do Not Do Now

```text
1. More RandomForest tuning
2. More blind stacking
3. Public LB chase without CV improvement
4. Direct target encoding leakage
5. Massive feature creation without validation
```

---

## 9. Expected Impact

| Change | Expected Impact |
|---|---|
| OOF Neighborhood target encoding | High |
| Neighborhood × quality/size | High |
| Neighborhood price tier | Medium-high |
| Quality-age interactions | Medium |
| Luxury/risk tail features | Medium-high |
| Sparse dummy cleanup | Medium |
| More RandomForest tuning | Low |
| More blind stacking | Low / risky |

---

## 10. Final Decision

FE v1 was strong but not enough for `0.111–0.115`. The main weakness was that the model did not have enough price-aware location representation. `Neighborhood` was one of the strongest signals, but it was not converted into leakage-safe target-encoded numeric signal. The model also lacked enough location-quality and location-size interactions, which are necessary to reduce error in both premium and low-price houses.

FE v2 should focus on:

```text
OOF target encoding
NeighborhoodPriceTier
location-quality interaction
location-size interaction
age/remodel interaction
tail-specific luxury/risk features
sparse feature cleanup
robust repeated CV
```

This route does not guarantee `0.111–0.113`, but it is the correct route. Same FE দিয়ে আরও RandomForest/tuning করলে target range-এ যাওয়ার chance কম।

---

## 11. Implementation Order

Recommended notebook order:

```text
1. Create FE v2 setup and folders
2. Load FE v1 cleaned base data before final encoding if available
3. Add OOF target encoding helper
4. Create Neighborhood_TE and NeighborhoodPriceTier
5. Add interaction features
6. Add tail/luxury/risk features
7. Create tree_v2 and linear_v2 datasets
8. Run dataset validation
9. Export fe_result_v2
10. Run quick repeated CV model validation
```

Recommended folder outputs:

```text
fe_result_v2/
fe_report_v2/
model_result_v2/
model_report_v2/
```

Recommended final files:

```text
X_train_tree_v2.csv
X_test_tree_v2.csv
X_train_linear_v2.csv
X_test_linear_v2.csv
y_train_log_v2.csv
y_train_raw_v2.csv
train_ids_v2.csv
test_ids_v2.csv
v2_feature_inventory.csv
v2_target_encoding_report.csv
v2_validation_report.csv
```
