# Improved EDA Markdown — Visualization-First Planning

## Purpose

This is a complete EDA planning framework focused on visualization.  
The goal is not to show every possible plot. The goal is to show the right plot at the right time so that each visualization supports a decision.

---

# 0. Visualization Control System

## 0.1 Visualization Priority

| Priority | Meaning | Use Case |
|---|---|---|
| P0 | Always show | Core dataset, target, missingness, feature type, drift, final decision dashboards |
| P1 | Conditional show | Show when an issue, threshold, or modeling risk is detected |
| P2 | Drill-down show | Show for selected/high-risk/important features only |
| P3 | Expert/debug show | Advanced plots for deeper investigation |

---

## 0.2 Visualization Intent

| Intent | Purpose | Example Visualization |
|---|---|---|
| Overview | Quickly understand structure | dtype chart, feature type count, target card |
| Diagnose | Detect problems | missingness matrix, outlier chart, Q-Q plot |
| Compare | Compare groups/train-test/classes/time | grouped bar, overlay histogram, ECDF |
| Decide | Support preprocessing/modeling decision | imputation chart, encoding decision table |
| Validate | Check assumption or transformation | raw vs transformed histogram, Q-Q plot |
| Communicate | Summarize final insight | dashboard, scorecard, final action table |

---

## 0.3 Adaptive Visualization Rules

| Condition | Visualization Strategy |
|---|---|
| rows < 10,000 | Full scatter/distribution plots are acceptable |
| rows >= 10,000 and < 100,000 | Use sampling for scatter plots |
| rows >= 100,000 | Use hexbin, density plot, ECDF, binned plot, or sampled plot |
| columns > 50 | Show top-N risky/problematic features |
| numeric features > 30 | Use sorted horizontal summary charts |
| categorical cardinality <= 10 | Full count plot |
| categorical cardinality > 10 and <= 30 | Sorted horizontal bar chart |
| categorical cardinality > 30 | Top-N + Other chart |
| categorical cardinality > 100 | Pareto chart / frequency rank plot |
| skewness > 1 | Log histogram + ECDF + Q-Q plot |
| high missing columns | Missingness matrix + sorted missing chart + UpSet plot |
| train/test drift exists | Overlay distribution + drift score chart |
| many correlated numeric features | Clustered heatmap + high-correlation table |
| sparse numeric feature | Zero percentage chart + non-zero distribution |

---

## 0.4 Visualization Annotation Rules

| Plot Type | Required Annotation |
|---|---|
| Bar chart | count + percentage |
| Distribution plot | mean, median, skewness, sample size |
| Boxplot | outlier percentage + IQR bounds |
| Train/test chart | train size, test size, gap percentage |
| Category-target chart | sample size per category |
| Transformed plot | transformation method |
| Sampled plot | sample size + sampling ratio |
| Log-scale plot | clearly mention log scale |
| Top-N chart | mention N and whether `Other` bucket is used |
| Drift chart | mention drift metric and threshold |

---

## 0.5 Do Not Plot Rules

| Condition | Avoid |
|---|---|
| ID-like feature | histogram, KDE, feature-target interpretation |
| binary feature | KDE |
| high-cardinality categorical | full category bar chart |
| discrete count feature with few values | KDE |
| very sparse numeric | raw histogram only |
| too many numeric features | pairplot |
| imbalanced classification target | raw count chart without percentage |
| suspected leakage feature | over-interpreting target relationship |
| very large dataset | full scatter without sampling/density |
| text ID-like column | word cloud or top word chart |

---

# 1. Basic Checking

## 1.1 Basic Checking Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Problem Type | regression / classification / clustering / forecasting | problem type card | P0 | Overview |
| Target Column | target exists, target type, target availability | target card | P0 | Overview |
| ID Column | unique ID, duplicate ID, ID-like behavior | duplicate ID card / ID review table | P1 | Diagnose |
| Train/Test Shape | rows, columns, train-test shape difference | shape comparison bar chart | P0 | Compare |
| Column Overview | column names, total columns, column mismatch | column difference table | P0 | Diagnose |
| Data Types | numeric, categorical, datetime, text, bool, object | dtype count bar chart | P0 | Overview |
| Duplicate Rows | full row duplicates in train/test | duplicate summary card | P1 | Diagnose |
| Missing Overview | total missing values, missing columns, missing percentage | missing percentage bar chart / missingness matrix | P0 | Diagnose |
| Feature Type Count | numeric/categorical/datetime/text/id/target count | feature type count chart | P0 | Overview |
| Initial Excluded Features | ID, target, leakage-like, unusable features | excluded feature table | P1 | Decide |
| Data Dictionary Review | semantic meaning of features | data dictionary table | P2 | Diagnose |
| Dataset Health Score | overall basic health | dataset health dashboard | P0 | Communicate |

## 1.2 Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| many missing columns | missingness matrix + sorted missing bar |
| train/test shape difference | grouped shape bar chart |
| many dtypes | dtype count chart |
| duplicate rows/IDs exist | duplicate summary card |
| feature type detection done | feature type summary bar chart |
| column mismatch exists | column difference table |
| dataset is large | compact dashboard + top-N issue tables |

## 1.3 Basic Checking Output

| Output | Description |
|---|---|
| Dataset health dashboard | rows, columns, target, missingness, duplicates, feature type |
| Initial issue list | critical issues before deeper EDA |
| Initial excluded feature list | ID-like, leakage-like, unusable, invalid features |
| Next-step recommendation | target analysis / type correction / data cleaning |

---

# 2. Target Analysis

## 2.1 Target Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Target Basic | dtype, missing count, missing percentage | target summary card | P0 | Overview |
| Target Summary | min, max, mean, median, mode, std, variance, range | target summary table | P0 | Overview |
| Target Distribution | skewness, kurtosis, distribution shape | histogram + KDE / ECDF | P0 | Diagnose |
| Target Outlier | target outlier count, outlier percentage, extreme values | boxplot / boxen plot | P1 | Diagnose |
| Target Percentile | P01, P05, P25, P50, P75, P95, P99 | percentile range plot | P1 | Diagnose |
| Target Transformation | raw target vs log/other transformed target | raw vs transformed histogram | P1 | Validate |
| Target Q-Q Check | normality-style visual check | Q-Q / probability plot | P1 | Validate |
| Target Metric Alignment | target transform aligns with metric or not | metric decision table | P1 | Decide |
| Target Drift | target difference across train/test/time | overlay distribution / time trend | P1 | Compare |

## 2.2 Regression Target Visualization

| Check | Visualization | Priority |
|---|---|---|
| distribution shape | histogram + KDE + ECDF | P0 |
| skewness | raw vs log histogram | P1 |
| heavy tail | Q-Q plot + percentile plot | P1 |
| outliers | boxplot / boxen plot | P1 |
| extreme values | P99-to-max and P01-to-min plot | P1 |
| target over time | target trend line | P1 |
| target by important category | boxplot / violinplot | P2 |

## 2.3 Classification Target Visualization

| Check | Visualization | Priority |
|---|---|---|
| class count | class count plot | P0 |
| class percentage | normalized class percentage bar | P0 |
| imbalance ratio | imbalance summary card | P0 |
| rare class | log-scale class count plot | P1 |
| target by category | normalized stacked bar chart | P2 |
| target by numeric bins | class rate by bin plot | P2 |
| multilabel target | label frequency heatmap | P2 |

## 2.4 Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| target right-skewed | histogram + KDE + log histogram + ECDF |
| target heavy-tailed | Q-Q plot + boxplot + percentile plot |
| target outliers exist | boxplot + percentile table |
| regression problem | target distribution mandatory |
| classification problem | class count + imbalance chart mandatory |
| target missing exists | target missing card + affected row table |
| target has temporal order | target trend over time |
| target transform considered | raw vs transformed distribution + metric decision table |

## 2.5 Target Analysis Output

| Output | Description |
|---|---|
| Target health summary | type, missingness, balance/skew, outlier risk |
| Target transformation recommendation | none / log / sqrt / Yeo-Johnson / review |
| Metric alignment note | whether target transformation fits evaluation metric |
| Modeling risk note | imbalance, heavy tail, leakage risk, invalid target |

---

# 3. Numeric Analysis

## 3.1 Numeric Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Numeric Dtype | int, float, nullable numeric | dtype count chart | P0 | Overview |
| Primary Type | continuous, count, ordinal, year-like, month-like, numeric-code, binary | numeric type summary chart | P0 | Diagnose |
| Manual Type | manually corrected feature type | manual type table | P1 | Decide |
| Detection Confidence | confidence of automated type detection | confidence count chart | P1 | Diagnose |
| Semantic Review | whether numeric meaning is clear | semantic review table | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| many numeric types | horizontal type count chart |
| numeric-code/year/month candidates exist | review table |
| low confidence types exist | confidence table |
| binary numeric exists | binary count chart |
| count-like numeric exists | count distribution / count plot |

---

## 3.2 Numeric Uniqueness Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Unique Count | number of unique values | unique count bar chart | P1 | Diagnose |
| Unique Ratio | unique count / row count | unique ratio chart | P1 | Diagnose |
| Cardinality Level | low, medium, high cardinality | cardinality summary chart | P1 | Overview |
| Constant Feature | one unique value | constant feature table | P1 | Decide |
| Near-constant Feature | one value dominates heavily | dominant value percentage chart | P1 | Diagnose |
| Categorical-like Numeric | numeric but behaves like category | review table | P1 | Decide |
| ID-like Numeric | too many unique values with ID behavior | ID-like review table | P1 | Decide |
| Frequency Rank | repeated value structure | frequency rank plot | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| Unique_Ratio high | ID-like review table |
| Dominant_Value_% high | dominant value chart |
| low-cardinality numeric exists | count plot for selected features |
| constant/near-constant exists | drop/review table |
| too many numeric features | top-N unique ratio chart |

---

## 3.3 Numeric Missingness Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Missing Count | number of missing values | missing count bar chart | P0 | Diagnose |
| Missing Percentage | percentage of missing values | missing percentage bar chart | P0 | Diagnose |
| Missing Status | no, low, moderate, high, very high | missing status summary chart | P0 | Overview |
| Missing Pattern | MCAR-like, MAR-like, MNAR-like, absence-like | missingness heatmap / UpSet plot | P1 | Diagnose |
| Missing Indicator Candidate | whether missing flag may help | candidate table | P1 | Decide |
| Missing vs Target | whether missingness relates to target | missing flag vs target boxplot/bar plot | P1 | Diagnose |
| Train-Test Missing Difference | missing gap between train and test | grouped train-test missing chart | P1 | Compare |
| Co-missing Features | features missing together | missingness correlation heatmap | P1 | Diagnose |
| Imputation Hint | mean/median/constant/zero/drop/review | imputation decision chart | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| missing exists | missing percentage chart |
| many missing columns | missingness matrix + UpSet plot |
| train-test gap exists | grouped missing chart |
| missing indicator candidate exists | missing flag vs target plot |
| high missing exists | high-missing feature table |
| missingness clusters exist | nullity correlation heatmap |

---

## 3.4 Numeric Central Tendency Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Mean | average value | mean vs median chart | P1 | Diagnose |
| Median | middle value | mean vs median chart | P1 | Diagnose |
| Mode | most frequent value | mode table | P2 | Diagnose |
| Mode Percentage | dominance of mode | mode dominance chart | P1 | Diagnose |
| Mean-Median Gap | difference between mean and median | mean-median gap chart | P1 | Diagnose |
| Center Stability | stable, shifted, mode-dominated | center status chart | P1 | Overview |
| Imputation Candidate | mean/median/mode suitability | imputation candidate table | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| mean-median gap high | mean vs median bar chart |
| mode percentage high | mode dominance chart |
| skew suspected | histogram with mean/median vertical lines |
| imputation decision needed | mean/median/mode comparison table |

---

## 3.5 Numeric Spread Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Min / Max | lowest and highest value | min-max range chart | P1 | Diagnose |
| Range | max - min | range bar chart | P1 | Diagnose |
| Variance / Std | spread around center | std/variance chart | P1 | Diagnose |
| Q1 / Q3 | lower and upper quartile | boxplot | P1 | Diagnose |
| IQR | Q3 - Q1 | IQR bar chart | P1 | Diagnose |
| Spread Level | no, low, moderate, wide spread | spread summary chart | P1 | Overview |
| IQR Bounds | lower/upper outlier boundary | boxplot | P1 | Diagnose |
| Robust Spread | median absolute deviation | MAD chart | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| many features | IQR/std horizontal chart |
| selected continuous features | boxplot grid |
| wide spread | min-Q1-median-Q3-max range plot |
| scale comparison needed | log-scale range chart |
| outlier-heavy feature | boxen plot / percentile strip |

---

## 3.6 Numeric Percentile Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Percentiles | P01, P05, P25, P50, P75, P95, P99 | percentile range plot | P1 | Diagnose |
| Upper Tail | Max-to-P99 gap/ratio | max-to-P99 chart | P1 | Diagnose |
| Lower Tail | Min-to-P01 gap | lower-tail gap chart | P1 | Diagnose |
| Tail Label | normal, moderate, extreme tail | tail summary chart | P1 | Overview |
| Percentile Review | whether tail behavior needs review | percentile review table | P1 | Decide |
| Percentile Compression | value compression in lower/upper region | percentile strip plot | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| upper tail extreme | max-to-P99 bar chart |
| lower tail extreme | min-to-P01 chart |
| selected features | percentile line plot |
| all continuous features | box-and-whisker grid |
| percentile compression | percentile table + percentile strip |

---

## 3.7 Numeric Distribution Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Skewness | distribution asymmetry | skewness bar chart | P1 | Diagnose |
| Kurtosis | tail heaviness | kurtosis bar chart | P1 | Diagnose |
| Distribution Shape | symmetric, right-skewed, left-skewed | distribution shape chart | P1 | Overview |
| Tail Behavior | normal, heavy, extreme | tail behavior chart | P1 | Diagnose |
| Transformation Candidate | log/sqrt/Yeo-Johnson/no-transform | transformation table | P1 | Decide |
| Distribution Review | features needing manual review | review table | P1 | Decide |
| ECDF | cumulative distribution shape | ECDF plot | P1 | Diagnose |
| Train/Test Distribution | train-test distribution consistency | overlay histogram / ECDF | P1 | Compare |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| abs(skewness) <= 0.5 | histogram + optional KDE |
| skewness > 0.5 | histogram + KDE + ECDF + Q-Q plot |
| skewness > 1 | histogram + KDE + log histogram + ECDF + Q-Q plot |
| skewness < -0.5 | histogram + KDE + ECDF + Q-Q plot |
| high kurtosis | Q-Q plot + boxplot |
| many features | skewness/kurtosis bar chart |
| important feature | histogram + KDE + ECDF + boxplot + Q-Q panel |
| count-like feature | count plot instead of KDE |
| zero-heavy feature | zero/non-zero split plot |

---

## 3.8 Numeric Special Values Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Zero Count | number of zeros | zero count chart | P1 | Diagnose |
| Zero Percentage | percentage of zeros | zero percentage chart | P1 | Diagnose |
| Zero Status | no, low, moderate, zero-heavy, dominated | zero status chart | P1 | Overview |
| Negative Count | number of negative values | negative count chart | P1 | Diagnose |
| Negative Percentage | percentage of negative values | negative percentage chart | P1 | Diagnose |
| Sentinel Values | hidden values like -999, 9999, NA-as-string | sentinel table | P1 | Diagnose |
| Absence-like Zero | whether zero means absence | zero-absence table | P1 | Decide |
| Special Value Action | keep, flag, replace, review | action table | P1 | Decide |
| Non-zero Distribution | distribution after excluding zero | non-zero histogram / boxplot | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| zero-heavy exists | zero percentage chart |
| negative exists | negative value chart |
| sentinel suspected | special value table |
| zero means absence | zero vs non-zero count plot |
| sparse numeric features | sorted zero percentage bar chart |
| zero-heavy and target-related | zero flag vs target plot |

---

## 3.9 Numeric Outlier Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Lower Outlier Count | below lower IQR boundary | lower outlier chart | P1 | Diagnose |
| Upper Outlier Count | above upper IQR boundary | upper outlier chart | P1 | Diagnose |
| Total Outlier Count | total outliers | outlier count chart | P1 | Diagnose |
| Outlier Percentage | percentage of outliers | outlier percentage chart | P1 | Diagnose |
| Outlier Direction | upper, lower, both | direction chart | P1 | Overview |
| Outlier Severity | no, low, moderate, high | severity chart | P1 | Diagnose |
| Extreme Tail | max/P99, min/P01 behavior | percentile plot | P1 | Diagnose |
| Capping Candidate | whether capping should be tested | capping review table | P1 | Decide |
| Outlier Action | keep, review, cap-test, remove-error-only | action table | P1 | Decide |
| Outlier Influence | whether outliers drive target relation | scatter with outlier highlight | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| outliers exist | outlier percentage chart |
| high severity | boxplot grid |
| upper outlier dominant | boxplot + percentile plot |
| lower outlier dominant | boxplot + min/P01 table |
| zero-heavy outlier | zero/non-zero plot + non-zero boxplot |
| capping experiment | before/after capping comparison plot |
| outlier affects target relation | scatter with outlier highlight |

---

## 3.10 Numeric Stability Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Dominant Value | most frequent value | dominant value table | P1 | Diagnose |
| Dominant Count | count of dominant value | dominant count chart | P1 | Diagnose |
| Dominant Percentage | percentage of dominant value | dominant percentage chart | P1 | Diagnose |
| Constant Stability | fully constant feature | constant table | P1 | Decide |
| Near-constant Stability | nearly constant feature | near-constant table | P1 | Decide |
| Zero-heavy Stability | zero-dominated feature | zero-heavy chart | P1 | Diagnose |
| Stable Feature | acceptable variation | stable feature chart | P2 | Overview |
| Stability Review | features needing review | stability review table | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| dominant value high | dominant percentage chart |
| near-constant exists | near-constant table |
| zero-heavy exists | zero percentage chart |
| stable vs unstable needed | stability summary chart |
| many unstable features | sorted instability chart |

---

## 3.11 Numeric Manual Review Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Review Needed | whether feature needs manual review | review count chart | P1 | Overview |
| Review Reason | missing, type, outlier, special value, leakage, semantic issue | reason table | P1 | Diagnose |
| Data Quality Issue | invalid value or suspicious value | issue table | P1 | Diagnose |
| Semantic Issue | unclear feature meaning | semantic table | P2 | Diagnose |
| Modeling Risk | leakage, instability, sparse signal | risk table | P1 | Decide |
| Final Numeric Decision | keep, drop, transform, flag, engineer, review | decision chart | P1 | Decide |

---

## 3.12 Numeric Feature Engineering Hints Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Imputation Decision | mean/median/constant/zero/drop/review | imputation summary chart | P1 | Decide |
| Missing Indicator Decision | create or not create missing flag | missing indicator chart | P1 | Decide |
| Zero Flag Decision | create or not create zero/presence flag | zero flag chart | P1 | Decide |
| Transformation Decision | raw/log/sqrt/Yeo-Johnson/no-transform | transformation chart | P1 | Decide |
| Outlier Handling Decision | keep/cap-test/remove-error-only/review | outlier action chart | P1 | Decide |
| Scaling Decision | scaler needed for linear/KNN, not tree | model-wise scaling table | P1 | Decide |
| Model-wise Treatment | linear vs tree vs KNN plan | model treatment table | P1 | Decide |
| Final Numeric Action | final feature engineering action | final action chart | P0 | Communicate |

---

## 3.13 Final Numeric EDA Summary

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Final Numeric Groups | continuous, count, ordinal, year, month, numeric-code | feature group chart | P0 | Communicate |
| Final Imputation Features | features needing imputation | imputation group chart | P0 | Communicate |
| Final Missing Flags | missing indicator features | missing flag chart | P0 | Communicate |
| Final Zero Flags | zero/presence flag features | zero flag chart | P1 | Communicate |
| Final Transformation Candidates | features for transform experiment | transformation chart | P0 | Communicate |
| Final Outlier Review | outlier review features | outlier review table | P0 | Communicate |
| Final Numeric Plan | model-wise numeric preprocessing plan | final numeric dashboard | P0 | Communicate |

---

# 4. Categorical Analysis

## 4.1 Categorical Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Dtype | object/category/bool/string | dtype count chart | P0 | Overview |
| Unique Count | number of categories | unique count chart | P1 | Diagnose |
| Unique Ratio | unique count / row count | unique ratio chart | P1 | Diagnose |
| Sample Values | example categories | sample table | P2 | Diagnose |
| Basic Type | binary/low/medium/high/constant | type count chart | P0 | Overview |
| Possible Ordinal | natural order candidate | ordinal candidate table | P1 | Decide |
| Possible Absence-related | missing may mean absence | absence table | P1 | Decide |
| Manual Type | final semantic categorical type | manual type chart | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| unique categories <= 10 | full count plot |
| 10 < unique categories <= 30 | sorted horizontal bar |
| unique categories > 30 | top-N + Other bar |
| unique categories > 100 | Pareto chart / frequency rank |
| binary category | binary count + percentage card |
| possible ordinal exists | ordinal review table |

---

## 4.2 Categorical Missingness Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Missing Count | missing count | missing count chart | P0 | Diagnose |
| Missing Percentage | missing percentage | missing percentage chart | P0 | Diagnose |
| Train/Test Missing | count and percentage | train-test missing chart | P1 | Compare |
| Missing Type | absence/unknown/review | missing type chart | P1 | Decide |
| Imputation Hint | None/Unknown/mode/drop/review | imputation chart | P1 | Decide |
| Missing Review | unclear missing meaning | review table | P1 | Diagnose |
| Missing vs Target | target relation with missing flag | missing flag target plot | P1 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| missing exists | missing percentage chart |
| train-test gap exists | grouped missing chart |
| absence-related exists | absence table |
| high missing exists | high-missing chart |
| missingness may be signal | missing flag vs target plot |

---

## 4.3 Categorical Cardinality Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Category Count | number of unique categories | unique count chart | P1 | Diagnose |
| Cardinality Level | binary/low/medium/high | cardinality chart | P0 | Overview |
| Encoding Risk | one-hot risk / high-cardinality risk | encoding risk chart | P1 | Decide |
| Action Hint | one-hot/rare-group/target-encoding-review | action table | P1 | Decide |
| Pareto Concentration | whether few categories dominate | Pareto chart | P1 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| low cardinality | full count plot |
| medium cardinality | sorted horizontal count plot |
| high cardinality | top-N + Other plot |
| very high cardinality | frequency rank plot + encoding risk table |
| one-hot risk exists | encoding risk chart |

---

## 4.4 Dominant Category Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Dominant Category | most frequent category | dominant table | P1 | Diagnose |
| Dominant Percentage | dominance level | dominant percentage chart | P1 | Diagnose |
| Second Category | second most frequent category | top category table | P2 | Diagnose |
| Dominance Status | balanced/moderate/high/near-constant | dominance chart | P1 | Overview |
| Action Hint | keep/drop/review/rare-group | action table | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| one category dominates | dominant percentage chart |
| near-constant categorical | near-constant table |
| dominance with target difference | dominant category vs target plot |
| many dominant features | sorted dominance chart |

---

## 4.5 Rare Category Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Rare Category Count | categories below threshold | rare count chart | P1 | Diagnose |
| Rare Row Percentage | rows belonging to rare categories | rare row chart | P1 | Diagnose |
| Rare List | rare category names | rare table | P2 | Diagnose |
| Rare Impact | low/moderate/high | rare status chart | P1 | Overview |
| Rare Grouping Candidate | whether to group as Rare | candidate table | P1 | Decide |
| Rare Target Stability | unstable target estimate for rare labels | rare target stability table | P2 | Diagnose |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| many rare labels | top-N + rare bucket plot |
| rare row percentage high | rare row percentage chart |
| rare grouping needed | rare grouping table |
| rare target signal unstable | target statistic with sample count table |

---

## 4.6 Train-Test Category Consistency Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Train Categories | categories in train | train-test count chart | P1 | Compare |
| Test Categories | categories in test | train-test count chart | P1 | Compare |
| Common Categories | shared categories | consistency table | P1 | Diagnose |
| Train-only Categories | only in train | train-only table | P1 | Diagnose |
| Test-only Categories | unseen in train | test-only table | P1 | Diagnose |
| Unseen Risk | low/moderate/high | unseen risk chart | P1 | Decide |
| Encoding Safety | handle unknown / align columns | safety table | P1 | Decide |
| Category Drift | train-test frequency shift | normalized drift bar chart | P1 | Compare |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| test-only categories exist | unseen category table |
| train-test category frequency shift exists | normalized stacked bar / drift chart |
| high-cardinality feature | top-N train/test category comparison |
| encoding risk exists | encoding safety table |

---

## 4.7 Ordinal Categorical Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Manual Ordinal | confirmed ordered feature | ordinal feature table | P1 | Decide |
| Mapping | category-to-rank mapping | mapping table | P1 | Decide |
| Mapping Coverage | mapped vs unmapped categories | coverage chart | P1 | Diagnose |
| Unmapped Categories | categories missing in mapping | unmapped table | P1 | Diagnose |
| Ordinal Action | encode/review/fix mapping | action table | P1 | Decide |
| Ordinal Target Trend | target movement by ordinal level | ordered line/bar plot | P2 | Validate |

---

## 4.8 Nominal Categorical Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Nominal Feature | unordered categorical feature | nominal count chart | P0 | Overview |
| Nominal Subtype | binary/standard/dominant/rare-sensitive/high-cardinality | subtype chart | P1 | Diagnose |
| Encoding Hint | one-hot/rare-group/frequency/target-encoding-review | encoding table | P1 | Decide |
| Review Features | encoding risk features | review table | P1 | Decide |

---

## 4.9 Categorical Target Relationship Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Category Count | rows per category | count plot | P1 | Diagnose |
| Target Mean/Median | target by category | target bar chart with count labels | P1 | Diagnose |
| Target Spread | target variability by category | boxplot / violinplot | P2 | Diagnose |
| Eta Squared | categorical-target separation strength | eta squared chart | P2 | Validate |
| Target Strength | weak/moderate/strong | strength chart | P1 | Overview |
| Rare Category Risk | rare category target instability | rare vs target table | P1 | Diagnose |
| Target Action | keep/group/review/encoding candidate | action table | P1 | Decide |

### Dynamic Visualization Rule

| Condition | Visualization |
|---|---|
| regression target | category target mean/median bar + boxplot |
| classification target | normalized stacked class bar |
| many categories | top-N target plot + Other |
| rare categories exist | target table with sample size |
| unstable category signal | confidence interval / count annotation |

---

## 4.10 Categorical Encoding Hints Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Missing Handling | None/Unknown/mode/drop | decision chart | P1 | Decide |
| Rare Decision | group or keep rare labels | rare decision chart | P1 | Decide |
| Ordinal Encoding | mapping needed or not | ordinal chart | P1 | Decide |
| One-hot Encoding | one-hot needed or not | one-hot chart | P1 | Decide |
| Frequency Encoding | frequency/count encoding candidate | frequency encoding table | P1 | Decide |
| Target Encoding Candidate | CV-safe target encoding candidate | candidate table | P1 | Decide |
| Final Action | final categorical action | final action chart | P0 | Communicate |

---

## 4.11 Categorical Leakage / Drop Review

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Constant Feature | one category only | constant table | P1 | Decide |
| Near-constant Feature | one category dominates | dominant chart | P1 | Diagnose |
| ID-like Feature | category behaves like ID | ID-like table | P1 | Decide |
| Leakage Suspect | post-target or target-like information | leakage table | P1 | Diagnose |
| Drop Decision | keep/drop/review | decision chart | P1 | Decide |
| Suspicious Target Link | unusually strong category-target relation | target relation chart | P2 | Diagnose |

---

## 4.12 Final Categorical EDA Summary

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Final Ordinal Features | ordinal encoding list | group chart | P0 | Communicate |
| Final Nominal Features | one-hot/frequency/target encoding list | group chart | P0 | Communicate |
| Final Rare Group Features | rare grouping list | rare group chart | P0 | Communicate |
| Final Missing Groups | None/Unknown/mode groups | imputation group chart | P0 | Communicate |
| Final Review Features | features needing review | review table | P0 | Communicate |
| Final Categorical Plan | model-wise categorical preprocessing | categorical dashboard | P0 | Communicate |

---

# 5. Datetime / Time-Series Analysis

## 5.1 Datetime Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Datetime Feature Detection | date/time/year/month columns | datetime feature table | P0 | Overview |
| Parsed Datetime Validity | valid vs invalid parsed dates | parse success chart | P1 | Diagnose |
| Min / Max Date | earliest and latest date | date range card | P0 | Overview |
| Date Range | total time span | timeline plot | P0 | Overview |
| Time Granularity | year/month/day/hour/minute level | granularity table | P1 | Diagnose |
| Missing Datetime | missing date count/percentage | missing datetime chart | P1 | Diagnose |
| Timezone Check | timezone consistency if applicable | timezone table | P2 | Diagnose |
| Future Date Check | dates beyond valid range | future date table | P1 | Diagnose |

---

## 5.2 Datetime Component Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Year | extracted year | year count plot | P1 | Overview |
| Quarter | extracted quarter | quarter count plot | P1 | Overview |
| Month | extracted month | month count plot | P1 | Diagnose |
| Week | extracted week | week count plot | P2 | Diagnose |
| Day | day of month | day count plot | P2 | Diagnose |
| Day of Week | weekday/weekend pattern | weekday count plot | P1 | Diagnose |
| Hour | hourly pattern | hour count plot | P1 | Diagnose |
| Season | season feature | season count chart | P2 | Diagnose |
| Calendar Pattern | daily/monthly pattern | calendar heatmap | P2 | Diagnose |

---

## 5.3 Time-Series Trend Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Target Trend | target movement over time | line plot | P0 | Diagnose |
| Rolling Mean | smoothed target trend | rolling mean plot | P1 | Diagnose |
| Rolling Std | time-varying volatility | rolling std plot | P1 | Diagnose |
| Trend Direction | increasing/decreasing/stable | trend line | P1 | Overview |
| Time Group Target | target by year/month/season | grouped target plot | P1 | Compare |
| Seasonality | repeated periodic pattern | seasonal plot | P2 | Diagnose |
| Decomposition | trend/seasonal/residual components | seasonal decomposition plot | P3 | Diagnose |

---

## 5.4 Time-Series Quality Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Duplicate Timestamp | repeated timestamps | duplicate timestamp table | P1 | Diagnose |
| Missing Timestamp Gap | gaps in time index | gap plot | P1 | Diagnose |
| Irregular Frequency | inconsistent intervals | interval distribution plot | P1 | Diagnose |
| Future Date | date beyond valid range | future date table | P1 | Diagnose |
| Invalid Date | impossible date values | invalid date table | P1 | Diagnose |
| Data Density Over Time | row availability over time | time density plot | P1 | Diagnose |

---

## 5.5 Time-Series Modeling Risk Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Temporal Leakage | future information used in past prediction | leakage table | P1 | Diagnose |
| Train-Test Time Split | chronological split validity | train-test timeline | P0 | Validate |
| Seasonality | repeated periodic pattern | seasonal plot | P2 | Diagnose |
| Lag Candidate | target/feature lag opportunity | lag correlation chart | P2 | Decide |
| Rolling Feature Candidate | rolling mean/std/min/max opportunity | rolling feature table | P2 | Decide |
| Forecast Horizon Alignment | feature availability at prediction time | horizon alignment table | P1 | Validate |

---

## 5.6 Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| datetime feature exists | timeline + date range card |
| invalid parsing exists | parse success chart + invalid date table |
| forecasting problem | target time plot + train/test timeline mandatory |
| irregular time index | interval distribution plot |
| seasonality suspected | seasonal plot + calendar heatmap |
| large time-series | resampled line plot |
| multiple time groups | faceted trend plot / grouped line plot |

---

# 6. Text Analysis

## 6.1 Text Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Text Feature Detection | free-text/string-heavy columns | text feature table | P0 | Overview |
| Text Missing | missing text count/percentage | missing text chart | P1 | Diagnose |
| Empty String Count | blank text values | empty string chart | P1 | Diagnose |
| Unique Text Count | unique text entries | unique text chart | P1 | Diagnose |
| Duplicate Text Count | repeated text values | duplicate text table | P1 | Diagnose |
| Text-like ID | text column behaves like ID | ID-like text table | P1 | Decide |

---

## 6.2 Text Length Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Character Count | number of characters | character length histogram | P1 | Diagnose |
| Word Count | number of words | word count histogram | P1 | Diagnose |
| Sentence Count | number of sentences | sentence count histogram | P2 | Diagnose |
| Average Word Length | text complexity proxy | average word length chart | P2 | Diagnose |
| Length Outlier | extremely short/long text | text length boxplot | P1 | Diagnose |
| Length vs Target | text length relationship with target | length target plot | P2 | Diagnose |

---

## 6.3 Text Quality Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Lowercase/Uppercase Ratio | casing pattern | casing ratio chart | P2 | Diagnose |
| Punctuation Count | punctuation usage | punctuation count chart | P2 | Diagnose |
| Digit Count | numeric tokens in text | digit count chart | P2 | Diagnose |
| Special Character Count | unusual characters | special character chart | P2 | Diagnose |
| URL Count | URLs inside text | URL count chart | P1 | Diagnose |
| Email/Phone Pattern | sensitive pattern existence | pattern table | P1 | Diagnose |
| Language Detection | possible language groups | language count chart | P2 | Diagnose |
| HTML/Markup Noise | tags or encoded entities | markup noise table | P2 | Diagnose |

---

## 6.4 Text Frequency Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Top Words | most frequent words | top word bar chart | P1 | Overview |
| Top Bigrams | most frequent 2-word phrases | bigram bar chart | P2 | Diagnose |
| Top Trigrams | most frequent 3-word phrases | trigram bar chart | P2 | Diagnose |
| Stopword Ratio | stopword percentage | stopword ratio chart | P2 | Diagnose |
| Vocabulary Size | number of unique tokens | vocabulary size card | P1 | Overview |
| Zipf Pattern | word frequency decay | Zipf plot | P3 | Diagnose |
| Word Cloud | stakeholder-friendly word overview | word cloud | P3 | Communicate |

---

## 6.5 Text Target Relationship Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Text Length vs Target | length relationship with target | scatter/bin plot | P2 | Diagnose |
| Keyword vs Target | target behavior for keyword presence | keyword target bar chart | P2 | Diagnose |
| Text Category vs Target | text-derived category relationship | boxplot/bar chart | P2 | Diagnose |
| Sentiment vs Target | sentiment relation if applicable | sentiment target plot | P3 | Diagnose |
| Text Signal Strength | whether text feature is useful | text signal table | P2 | Decide |
| TF-IDF Signal | sparse token predictive signal | top token importance chart | P3 | Diagnose |

---

## 6.6 Text Feature Engineering Hints

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Clean Text Candidate | lowercase, strip, normalize whitespace | cleaning decision table | P1 | Decide |
| Token Feature Candidate | word/char n-grams | n-gram summary | P2 | Decide |
| TF-IDF Candidate | sparse text representation | TF-IDF plan table | P2 | Decide |
| Embedding Candidate | semantic representation | embedding plan table | P2 | Decide |
| Text Drop Candidate | weak/noisy/free-ID text | drop/review table | P1 | Decide |
| Sensitive Data Candidate | email/phone/name-like text | sensitive pattern table | P1 | Diagnose |

---

## 6.7 Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| text column exists | text feature table + length histogram |
| many empty strings | empty string chart |
| long text exists | word count histogram + length boxplot |
| many repeated texts | duplicate text table |
| high-cardinality short text | ID-like text review table |
| classification target | keyword/class association chart |
| large text corpus | sampled examples + top words + top n-grams |
| multilingual text suspected | language count chart |

---

# 7. Train/Test Drift Analysis

## 7.1 Drift Overview Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Shape Drift | train/test row and column differences | shape comparison chart | P0 | Compare |
| Missing Drift | missing rate difference | grouped missing chart | P1 | Compare |
| Numeric Drift | numeric distribution shift | overlay histogram / ECDF | P1 | Compare |
| Categorical Drift | category frequency shift | normalized stacked bar | P1 | Compare |
| Unseen Categories | categories present only in test | unseen category table | P1 | Diagnose |
| Target Drift | target distribution difference, if available | overlay target distribution | P1 | Compare |
| Feature Drift Score | PSI / KS / JS distance | drift score bar chart | P1 | Diagnose |
| Drift Severity | low/moderate/high drift | drift dashboard | P0 | Communicate |

---

## 7.2 Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| numeric feature drift suspected | train/test ECDF overlay |
| numeric distribution is skewed | log-scale train/test histogram |
| categorical cardinality <= 20 | normalized train/test category bar |
| categorical cardinality > 20 | top-N category drift plot |
| missing rate difference exists | grouped missing rate chart |
| test-only categories exist | unseen category table |
| many drifted features | sorted drift score chart |

---

## 7.3 Drift Output

| Output | Description |
|---|---|
| Drifted feature list | features with train/test distribution shift |
| High-risk unseen category list | categorical levels appearing only in test |
| Drift severity dashboard | numeric, categorical, and missing drift summary |
| Modeling action | robust encoding, feature review, validation strategy adjustment |

---

# 8. Bivariate Analysis

## 8.1 Bivariate Basic Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Relationship Type | numeric-target, categorical-target, numeric-numeric, categorical-categorical | relationship type chart | P0 | Overview |
| Valid Pair Count | non-missing pair rows | valid pair table | P1 | Diagnose |
| Relationship Strength | weak/moderate/strong | strength chart | P1 | Overview |
| Direction | positive/negative/mixed/no trend | direction chart | P1 | Diagnose |
| Shape | linear/nonlinear/monotonic/grouped/sparse | shape table | P1 | Diagnose |
| Action Hint | keep/engineer/review/drop | action table | P1 | Decide |

---

## 8.2 Numeric vs Target

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Pearson Correlation | linear numeric-target relation | correlation bar chart | P1 | Diagnose |
| Spearman Correlation | monotonic numeric-target relation | Spearman chart | P1 | Diagnose |
| Scatter Pattern | trend/noise/nonlinearity | scatter plot | P2 | Diagnose |
| Nonlinear Pattern | curved relationship | scatter + LOWESS | P2 | Diagnose |
| Outlier Influence | extreme points affecting relation | scatter with outlier highlight | P2 | Diagnose |
| Zero Group Difference | zero vs non-zero target behavior | zero flag vs target boxplot | P2 | Diagnose |
| Decile Target Trend | target across numeric bins | decile target plot | P1 | Diagnose |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| rows < 10,000 | scatter + trend line |
| rows >= 100,000 | hexbin / sampled scatter / binned target plot |
| monotonic relationship suspected | Spearman chart + decile target trend |
| nonlinear relationship suspected | LOWESS scatter / binned target curve |
| outliers influence relationship | scatter with outlier highlight |
| zero-heavy numeric | zero vs non-zero target plot |

---

## 8.3 Numeric Binned vs Target

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Quantile Bins | binned numeric groups | binned count chart | P1 | Diagnose |
| Bin Target Mean/Median | target by bins | ordered bin target plot | P1 | Diagnose |
| Monotonicity | increasing/decreasing target across bins | monotonic line chart | P1 | Diagnose |
| Nonlinear Trend | curved bin-level trend | bin target curve | P1 | Diagnose |
| Bin Stability | enough samples per bin | bin count annotation | P1 | Validate |

---

## 8.4 Categorical vs Target

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Category Count | rows per category | count plot | P1 | Diagnose |
| Target Mean/Median | target by category | bar chart with count labels | P1 | Diagnose |
| Target Spread | target variation by category | boxplot / violinplot | P2 | Diagnose |
| Eta Squared | category-target separation | eta squared chart | P2 | Validate |
| Rare Category Risk | rare category target instability | rare vs target table | P1 | Diagnose |
| Classification Rate | class rate by category | normalized stacked bar | P1 | Diagnose |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| regression target | target mean/median bar + boxplot |
| classification target | normalized stacked class chart |
| many categories | top-N target chart + Other |
| rare categories exist | target table with sample count |
| category target estimates unstable | confidence interval / count annotation |

---

## 8.5 Ordinal/Binary/Flag vs Target

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Ordinal Trend | ordered levels vs target | ordered line/bar plot | P1 | Diagnose |
| Binary Difference | target difference between 0/1 | binary bar/boxplot | P1 | Diagnose |
| Missing Flag Signal | target differs by missing flag | missing flag vs target plot | P1 | Diagnose |
| Zero Flag Signal | target differs by zero flag | zero flag vs target plot | P1 | Diagnose |
| Outlier Flag Signal | target differs by outlier flag | outlier flag vs target plot | P2 | Diagnose |
| Flag Action | whether to engineer flag feature | action table | P1 | Decide |

---

## 8.6 Feature vs Feature

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Numeric-Numeric | correlation, redundancy, interaction | scatter / heatmap | P1 | Diagnose |
| Numeric-Categorical | numeric distribution by category | boxplot / violinplot | P2 | Diagnose |
| Categorical-Categorical | association / contingency | crosstab heatmap | P2 | Diagnose |
| Datetime-Target | target by time group | time line/bar plot | P1 | Diagnose |
| Text-Target | text-derived feature vs target | text signal plot | P2 | Diagnose |
| Feature Interaction | combined effect of two features | colored scatter / grouped line | P3 | Diagnose |

---

## 8.7 Bivariate Feature Engineering Candidate Summary

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Interaction Candidate | feature pair to combine | interaction table | P2 | Decide |
| Ratio Candidate | feature ratio opportunity | ratio scatter | P2 | Decide |
| Binning Candidate | numeric binning opportunity | binned target plot | P1 | Decide |
| Grouping Candidate | rare/category grouping opportunity | category target chart | P1 | Decide |
| Flag Candidate | missing/zero/outlier flag opportunity | flag target chart | P1 | Decide |
| Final Bivariate Decision | keep/create/review/drop | final bivariate dashboard | P0 | Communicate |

---

# 9. Multivariate Analysis

## 9.1 Correlation Matrix Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Pearson Matrix | linear numeric relation | Pearson heatmap | P1 | Diagnose |
| Spearman Matrix | monotonic numeric relation | Spearman heatmap | P1 | Diagnose |
| Target Correlation | feature-target correlation | target correlation bar chart | P0 | Diagnose |
| High Correlation Pairs | redundant numeric pairs | high-correlation table | P1 | Decide |
| Clustered Correlation | grouped correlated features | clustered heatmap | P1 | Diagnose |
| Correlation Network | feature relationship graph | correlation network graph | P3 | Diagnose |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| numeric features <= 20 | full heatmap |
| numeric features > 20 | clustered heatmap + high-correlation table |
| strong target correlation exists | sorted target correlation chart |
| high redundancy exists | high-correlation pair table |
| nonlinear relationships suspected | Spearman heatmap |

---

## 9.2 Multicollinearity / VIF Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| VIF Score | multicollinearity risk | VIF bar chart | P2 | Diagnose |
| High VIF Features | features with high VIF | high VIF table | P2 | Decide |
| Related Feature Group | correlated feature group | grouped heatmap | P2 | Diagnose |
| Linear Model Risk | coefficient instability risk | linear risk table | P2 | Decide |
| Action Hint | drop/regularize/keep/review | VIF action table | P2 | Decide |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| linear/logistic model planned | VIF bar chart |
| tree-only model planned | VIF optional |
| high VIF exists | high VIF table + correlated group heatmap |
| many numeric features | top-N VIF chart |

---

## 9.3 Feature Redundancy / Cluster Report

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Redundant Feature Pair | duplicate/overlapping features | redundancy table | P1 | Decide |
| Feature Cluster | related feature group | cluster table | P1 | Diagnose |
| Cluster Theme | semantic feature group | theme table | P2 | Diagnose |
| Representative Feature | best feature to represent group | representative table | P1 | Decide |
| Drop Candidate | redundant feature candidate | keep/drop table | P1 | Decide |
| Cluster Overview | groups of similar numeric features | clustered heatmap | P1 | Diagnose |

---

## 9.4 PCA / Dimensional Structure

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Explained Variance | variance per component | explained variance bar chart | P3 | Diagnose |
| Cumulative Variance | total variance explained | cumulative curve | P3 | Diagnose |
| Component Loadings | important features per component | loading bar chart | P3 | Diagnose |
| PC Scatter | projected sample structure | PC1 vs PC2 scatter | P3 | Diagnose |
| PCA Outlier | unusual samples in component space | PC scatter with outliers | P3 | Diagnose |
| Dimensionality Hint | whether dimensionality reduction may help | PCA decision table | P3 | Decide |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| many correlated numeric features | PCA variance curve |
| dimensionality reduction considered | explained variance + loading chart |
| anomaly pattern suspected | PC scatter with outliers |
| interpretability is critical | PCA optional / low priority |

---

## 9.5 Interaction / Grouped Effects

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Numeric Interaction | numeric pair effect on target | colored scatter | P3 | Diagnose |
| Category-conditioned Effect | numeric-target trend by category | faceted scatter | P3 | Diagnose |
| Ratio Feature Candidate | ratio relation with target | ratio scatter | P2 | Decide |
| Grouped Target Trend | target by group and bin | grouped line plot | P2 | Diagnose |
| Interaction Action | create/test/review/reject | action table | P2 | Decide |
| Segment-specific Signal | feature signal changes by group | segmented target plot | P3 | Diagnose |

---

## 9.6 Missingness Multivariate Pattern

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Co-missing Features | features missing together | co-missing heatmap | P1 | Diagnose |
| Missing Indicator Correlation | correlation among missing flags | missingness correlation heatmap | P1 | Diagnose |
| Missing Pattern Type | MCAR/MAR/MNAR/absence-like | pattern summary chart | P1 | Diagnose |
| Missingness vs Target | target relation with missing patterns | missing flag target plot | P1 | Diagnose |
| Missingness Action | impute/flag/drop/review | action table | P1 | Decide |
| Missing Pattern Combination | common null combinations | UpSet plot | P2 | Diagnose |

---

## 9.7 Categorical Multivariate Pattern

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Categorical Association | association between categorical features | Cramer's V heatmap | P2 | Diagnose |
| Joint Category Count | category combination frequency | joint heatmap | P3 | Diagnose |
| Sparse Combinations | rare category combinations | sparse table | P2 | Diagnose |
| Category Interaction | useful combined category candidate | target by category-pair heatmap | P3 | Diagnose |
| Redundant Categorical Pair | overlapping categorical features | redundancy table | P2 | Decide |
| Encoding Interaction Risk | combined one-hot explosion risk | encoding risk table | P2 | Decide |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| categorical features <= 20 | Cramer's V heatmap |
| many categorical features | top association pairs table |
| sparse combinations exist | sparse combination table |
| interaction suspected | target by category-pair heatmap |
| high-cardinality categories exist | avoid full joint heatmap |

---

## 9.8 Model-driven Multivariate Signal

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Feature Importance | model-based importance | importance chart | P2 | Diagnose |
| Permutation Importance | validation-based feature reliance | permutation chart | P2 | Validate |
| SHAP Importance | contribution-based feature signal | SHAP summary plot | P3 | Diagnose |
| Partial Dependence Candidate | nonlinear effect review | PDP plot | P3 | Diagnose |
| Importance Stability | fold-wise stability | stability chart | P2 | Validate |
| Weak Feature Candidate | consistently low-signal feature | weak feature table | P2 | Decide |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| baseline model trained | feature importance chart |
| validation set available | permutation importance chart |
| interpretability needed | SHAP summary plot |
| nonlinear effect suspected | PDP / ICE plot |
| cross-validation used | fold-wise importance stability chart |

---

## 9.9 Leakage / Data Quality Review

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| Leakage Candidate | target/post-event information | leakage table | P1 | Diagnose |
| Suspicious Target Relation | unusually strong feature-target relation | target relation chart | P1 | Diagnose |
| Duplicate Signal | duplicate or derived target-like feature | duplicate table | P1 | Diagnose |
| Data Quality Risk | inconsistent feature combinations | anomaly table | P1 | Diagnose |
| Final Leakage Decision | keep/drop/review | decision table | P1 | Decide |
| Leakage Risk Dashboard | overall leakage risks | leakage dashboard | P0 | Communicate |

### Dynamic Visualization Rules

| Condition | Visualization |
|---|---|
| feature has near-perfect target relation | leakage review table |
| feature appears post-event | timeline/leakage table |
| duplicate target-like feature exists | duplicate signal table |
| suspicious category-target relation | target relation chart with warning |
| high leakage risk exists | leakage dashboard |

---

## 9.10 Final Multivariate Summary

| Report | What to Check | Visualization | Priority | Intent |
|---|---|---|---|---|
| High Correlation Pairs | redundant numeric feature pairs | pair table + heatmap | P0 | Communicate |
| High VIF Features | linear multicollinearity issues | VIF chart | P2 | Communicate |
| Feature Clusters | related feature groups | clustered heatmap | P1 | Communicate |
| PCA Summary | dimensional structure | variance curve | P3 | Communicate |
| Interaction Candidates | features to combine/test | interaction table | P1 | Communicate |
| Missingness Groups | co-missing feature groups | missingness heatmap | P1 | Communicate |
| Categorical Associations | associated categorical pairs | Cramer's V heatmap | P2 | Communicate |
| Model-important Features | important features from baseline model | importance chart | P1 | Communicate |
| Final Multivariate Decisions | final keep/drop/engineer/review decisions | multivariate dashboard | P0 | Communicate |

---

# 10. Final EDA Decision Dashboard

## 10.1 Dataset Health Dashboard

| Component | Visualization |
|---|---|
| Rows and columns | shape card |
| Target availability | target card |
| Feature type count | feature type bar chart |
| Missingness | missing percentage chart |
| Duplicates | duplicate summary card |
| Column mismatch | column difference table |
| Overall health | dataset health scorecard |

---

## 10.2 Target Dashboard

| Component | Visualization |
|---|---|
| Target type | target card |
| Target distribution | histogram + ECDF / class count plot |
| Target skew/imbalance | skew chart / imbalance chart |
| Target outlier | boxplot / percentile chart |
| Target transformation | raw vs transformed histogram |
| Target modeling risk | risk summary card |

---

## 10.3 Feature Quality Dashboard

| Component | Visualization |
|---|---|
| Missing features | missing percentage chart |
| Constant features | constant feature table |
| Near-constant features | dominant percentage chart |
| High-cardinality features | cardinality chart |
| Rare category features | rare category chart |
| Special value features | special value table |
| Outlier features | outlier percentage chart |

---

## 10.4 Train/Test Drift Dashboard

| Component | Visualization |
|---|---|
| Shape drift | train/test shape chart |
| Missing drift | grouped missing chart |
| Numeric drift | ECDF overlay / drift score chart |
| Categorical drift | normalized category drift chart |
| Test-only categories | unseen category table |
| Drift severity | drift dashboard |

---

## 10.5 Feature-Target Signal Dashboard

| Component | Visualization |
|---|---|
| Top numeric correlations | target correlation bar chart |
| Numeric nonlinear signal | decile target plot |
| Categorical signal | category target chart |
| Missing flag signal | missing flag target chart |
| Zero flag signal | zero flag target chart |
| Important features | feature importance chart |

---

## 10.6 Preprocessing Decision Dashboard

| Component | Visualization |
|---|---|
| Numeric imputation | imputation decision chart |
| Categorical imputation | missing handling chart |
| Missing flags | missing indicator chart |
| Zero flags | zero flag chart |
| Transformations | transformation candidate chart |
| Outlier handling | outlier action chart |
| Scaling | model-wise scaling table |
| Encoding | encoding decision table |
| Drop/review list | final review table |

---

## 10.7 Model Risk Dashboard

| Component | Visualization |
|---|---|
| Leakage risk | leakage table/dashboard |
| Multicollinearity | correlation heatmap / VIF chart |
| High-cardinality risk | encoding risk chart |
| Rare category risk | rare category stability table |
| Train/test drift risk | drift severity chart |
| Outlier risk | outlier severity chart |
| Missingness risk | missingness pattern chart |

---

# 11. Recommended Report Flow

## 11.1 Executive Summary First

| Summary Item | Visualization |
|---|---|
| Dataset shape | shape card |
| Target status | target card |
| Main data quality issues | issue count chart |
| Main modeling risks | risk dashboard |
| Top preprocessing actions | action summary table |

## 11.2 Detailed EDA Order

1. Basic Checking
2. Target Analysis
3. Train/Test Drift Analysis
4. Numeric Analysis
5. Categorical Analysis
6. Datetime / Time-Series Analysis
7. Text Analysis
8. Bivariate Analysis
9. Multivariate Analysis
10. Final Preprocessing Plan

## 11.3 Final Output Format

| Output | Description |
|---|---|
| Summary dashboard | high-level dataset and risk overview |
| Detailed visual report | section-wise charts and tables |
| Feature decision table | keep/drop/transform/flag/engineer/review |
| Model-wise preprocessing plan | linear/tree/KNN/boosting-specific preprocessing |
| Review checklist | items needing human confirmation |
| Reproducible config | thresholds, top-N rules, sample size, chart settings |

---

# 12. Visualization Configuration Template

```yaml
visualization_config:
  priority_levels:
    P0: always_show
    P1: conditional_show
    P2: drill_down
    P3: expert_debug

  top_n:
    categorical_bar: 15
    missing_features: 30
    outlier_features: 30
    correlation_features: 30
    feature_importance: 25

  sampling:
    scatter_max_rows: 10000
    density_plot_min_rows: 100000
    random_state: 42

  missingness:
    high_missing_threshold: 0.40
    very_high_missing_threshold: 0.70
    missing_drift_threshold: 0.05

  categorical:
    low_cardinality_max: 10
    medium_cardinality_max: 30
    high_cardinality_max: 100
    rare_category_threshold: 0.01
    use_top_n_other: true

  numeric:
    skew_moderate_threshold: 0.5
    skew_high_threshold: 1.0
    high_kurtosis_threshold: 3.0
    outlier_method: iqr
    outlier_percentage_threshold: 0.05
    zero_heavy_threshold: 0.50

  drift:
    numeric_drift_methods:
      - ks_test
      - psi
    categorical_drift_methods:
      - psi
      - js_distance
    drift_warning_threshold: 0.10
    drift_high_threshold: 0.25

  style:
    show_counts: true
    show_percentages: true
    sort_bars_descending: true
    annotate_sample_size: true
    annotate_log_scale: true
    consistent_train_test_colors: true
    consistent_target_colors: true
```

---

# 13. Auto-Insight Text Rules

Each visualization should generate a short interpretation.

| Visualization | Auto Insight Example |
|---|---|
| Missing percentage chart | `5 features have more than 40% missing values.` |
| Dtype count chart | `Dataset contains 12 numeric, 8 categorical, and 2 datetime features.` |
| Target histogram | `Target is highly right-skewed; log transformation should be tested.` |
| Class count plot | `Target is imbalanced; minority class is 8.4% of rows.` |
| Boxplot | `Feature X has 6.2% IQR-based outliers.` |
| Category count plot | `Feature Y has 140 unique categories; one-hot encoding may be risky.` |
| Train/test ECDF | `Feature Z shows visible train-test distribution shift.` |
| Correlation heatmap | `4 feature pairs have correlation greater than 0.90.` |
| Feature importance chart | `Top 10 features explain most baseline model signal.` |
| Leakage table | `Feature A may contain post-target information and needs review.` |

---

# 14. Final EDA Action Table

| Feature | Type | Main Issue | Visualization Evidence | Suggested Action | Priority |
|---|---|---|---|---|---|
| feature_name | numeric/categorical/datetime/text | missing/outlier/drift/leakage/cardinality | chart/table name | keep/drop/impute/flag/transform/encode/review | high/medium/low |

---

# 15. Final Preprocessing Plan Template

| Feature Group | Features | Issue | Action | Model Notes |
|---|---|---|---|---|
| Numeric continuous | list | skew/outlier/missing | median impute + transform test + scaling | scaling needed for linear/KNN/SVM |
| Numeric count | list | zero-heavy/skew | zero flag + log1p test | tree models may handle raw better |
| Numeric binary | list | class imbalance/missing | impute mode or explicit missing flag | no scaling usually needed |
| Categorical low-cardinality | list | missing/rare | Unknown impute + one-hot | safe for most models |
| Categorical high-cardinality | list | many levels/unseen risk | rare grouping + frequency/target encoding review | target encoding must be CV-safe |
| Ordinal categorical | list | ordered levels | ordinal mapping | verify mapping manually |
| Datetime | list | leakage/time split | extract components + validate availability | use chronological validation if forecasting |
| Text | list | sparse/noisy/free text | TF-IDF/embedding/drop review | choose based on model and task |
| Leakage suspect | list | target/post-event info | drop or manual review | do not use before confirmation |

---

# 16. Final Principle

A good visualization-first EDA report should answer these questions:

1. Is the dataset structurally valid?
2. Is the target usable and aligned with the metric?
3. Which features have quality issues?
4. Which features show useful target signal?
5. Is there train/test drift?
6. Which features should be kept, dropped, transformed, encoded, flagged, or reviewed?
7. What preprocessing plan should be tested for each model family?

Final principle:

> Do not show every possible chart. Show the chart that answers the next decision