
---

# Improved EDA Markdown

## 1. Basic Checking

| Report                    | What to Check                                             | Visualization                                     |
| ------------------------- | --------------------------------------------------------- | ------------------------------------------------- |
| Problem Type              | regression / classification / clustering / forecasting    | none                                              |
| Target Column             | target exists, target type, target availability           | target card                                       |
| ID Column                 | unique ID, duplicate ID, ID-like behavior                 | duplicate ID card                                 |
| Train/Test Shape          | rows, columns, train-test shape difference                | shape comparison bar chart                        |
| Column Overview           | column names, total columns, column mismatch              | column difference table                           |
| Data Types                | numeric, categorical, datetime, text, bool, object        | dtype count bar chart                             |
| Duplicate Rows            | full row duplicates in train/test                         | duplicate summary card                            |
| Missing Overview          | total missing values, missing columns, missing percentage | missing percentage bar chart / missingness matrix |
| Feature Type Count        | numeric/categorical/datetime/text/id/target count         | feature type count chart                          |
| Initial Excluded Features | ID, target, leakage-like, unusable features               | excluded feature table                            |
| Data Dictionary Review    | semantic meaning of features                              | data dictionary table                             |

**Dynamic Visualization Rule**

| Condition                   | Visualization                  |
| --------------------------- | ------------------------------ |
| many missing columns        | missingness matrix             |
| train/test shape difference | grouped bar chart              |
| many dtypes                 | dtype count chart              |
| duplicate rows/IDs exist    | duplicate summary card         |
| feature type detection done | feature type summary bar chart |

---

## 2. Target Analysis

| Report                  | What to Check                                                   | Visualization                |
| ----------------------- | --------------------------------------------------------------- | ---------------------------- |
| Target Basic            | dtype, missing count, missing percentage                        | target summary card          |
| Target Summary          | min, max, mean, median, mode, std, variance, range              | target summary table         |
| Target Distribution     | skewness, kurtosis, distribution shape                          | histogram + KDE              |
| Target Outlier          | target outlier count, outlier percentage, extreme target values | boxplot                      |
| Target Percentile       | P01, P05, P25, P50, P75, P95, P99                               | percentile range plot        |
| Target Transformation   | raw target vs log/other transformed target                      | raw vs transformed histogram |
| Target Q-Q Check        | normality-style visual check                                    | Q-Q / probability plot       |
| Target Metric Alignment | whether target transform aligns with metric                     | metric decision table        |

**Dynamic Visualization Rule**

| Condition              | Visualization                      |
| ---------------------- | ---------------------------------- |
| target right-skewed    | histogram + KDE + log histogram    |
| target heavy-tailed    | Q-Q plot + boxplot                 |
| target outliers exist  | boxplot + percentile table         |
| regression problem     | target distribution mandatory      |
| classification problem | class count plot / imbalance chart |

---

# 3. Numeric Analysis

## 3.1 Numeric Basic Report

| Report               | What to Check                                                           | Visualization              |
| -------------------- | ----------------------------------------------------------------------- | -------------------------- |
| Numeric Dtype        | int, float, nullable numeric                                            | dtype count chart          |
| Primary Type         | continuous, count, ordinal, year-like, month-like, numeric-code, binary | numeric type summary chart |
| Manual Type          | manually corrected feature type                                         | manual type table          |
| Detection Confidence | confidence of automated type detection                                  | confidence count chart     |
| Semantic Review      | whether numeric meaning is clear                                        | semantic review table      |

**Dynamic Visualization Rule**

| Condition                                | Visualization               |
| ---------------------------------------- | --------------------------- |
| many numeric types                       | horizontal type count chart |
| numeric-code/year/month candidates exist | review table                |
| low confidence types exist               | confidence table            |

---

## 3.2 Numeric Uniqueness Report

| Report                   | What to Check                           | Visualization                   |
| ------------------------ | --------------------------------------- | ------------------------------- |
| Unique Count             | number of unique values                 | unique count bar chart          |
| Unique Ratio             | unique count / row count                | unique ratio chart              |
| Cardinality Level        | low, medium, high cardinality           | cardinality summary chart       |
| Constant Feature         | one unique value                        | constant feature table          |
| Near-constant Feature    | one value dominates heavily             | dominant value percentage chart |
| Categorical-like Numeric | numeric but behaves like category       | review table                    |
| ID-like Numeric          | too many unique values with ID behavior | ID-like review table            |

**Dynamic Visualization Rule**

| Condition                      | Visualization                    |
| ------------------------------ | -------------------------------- |
| `Unique_Ratio` high            | ID-like review table             |
| `Dominant_Value_%` high        | dominant value chart             |
| low-cardinality numeric exists | count plot for selected features |
| constant/near-constant exists  | drop/review table                |

---

## 3.3 Numeric Missingness Report

| Report                        | What to Check                                | Visualization                    |
| ----------------------------- | -------------------------------------------- | -------------------------------- |
| Missing Count                 | number of missing values                     | missing count bar chart          |
| Missing Percentage            | percentage of missing values                 | missing percentage bar chart     |
| Missing Status                | no, low, moderate, high, very high           | missing status summary chart     |
| Missing Pattern               | MCAR-like, MAR-like, MNAR-like, absence-like | missingness heatmap              |
| Missing Indicator Candidate   | whether missing flag may help                | candidate table                  |
| Missing vs Target             | whether missingness relates to target        | missing flag vs target boxplot   |
| Train-Test Missing Difference | missing gap between train and test           | grouped train-test missing chart |
| Imputation Hint               | mean/median/constant/zero/drop/review        | imputation decision chart        |

**Dynamic Visualization Rule**

| Condition                          | Visualization               |
| ---------------------------------- | --------------------------- |
| missing exists                     | missing percentage chart    |
| many missing columns               | missingness matrix          |
| train-test gap exists              | grouped missing chart       |
| missing indicator candidate exists | missing flag vs target plot |
| high missing                       | high-missing feature table  |

---

## 3.4 Numeric Central Tendency Report

| Report               | What to Check                      | Visualization              |
| -------------------- | ---------------------------------- | -------------------------- |
| Mean                 | average value                      | mean vs median chart       |
| Median               | middle value                       | mean vs median chart       |
| Mode                 | most frequent value                | mode table                 |
| Mode Percentage      | dominance of mode                  | mode dominance chart       |
| Mean-Median Gap      | difference between mean and median | mean-median gap chart      |
| Center Stability     | stable, shifted, mode-dominated    | center status chart        |
| Imputation Candidate | mean/median/mode suitability       | imputation candidate table |

**Dynamic Visualization Rule**

| Condition                  | Visualization                             |
| -------------------------- | ----------------------------------------- |
| mean-median gap high       | mean vs median bar chart                  |
| mode percentage high       | mode dominance chart                      |
| skew suspected             | histogram with mean/median vertical lines |
| imputation decision needed | mean/median/mode comparison table         |

---

## 3.5 Numeric Spread Report

| Report         | What to Check                  | Visualization        |
| -------------- | ------------------------------ | -------------------- |
| Min / Max      | lowest and highest value       | min-max range chart  |
| Range          | max - min                      | range bar chart      |
| Variance / Std | spread around center           | std/variance chart   |
| Q1 / Q3        | lower and upper quartile       | boxplot              |
| IQR            | Q3 - Q1                        | IQR bar chart        |
| Spread Level   | no, low, moderate, wide spread | spread summary chart |
| IQR Bounds     | lower/upper outlier boundary   | boxplot              |

**Dynamic Visualization Rule**

| Condition                    | Visualization                   |
| ---------------------------- | ------------------------------- |
| many features                | IQR/std horizontal chart        |
| selected continuous features | boxplot grid                    |
| wide spread                  | min-Q1-median-Q3-max range plot |
| scale comparison needed      | log-scale range chart           |

---

## 3.6 Numeric Percentile Report

| Report            | What to Check                      | Visualization           |
| ----------------- | ---------------------------------- | ----------------------- |
| Percentiles       | P01, P05, P25, P50, P75, P95, P99  | percentile range plot   |
| Upper Tail        | Max-to-P99 gap/ratio               | max-to-P99 chart        |
| Lower Tail        | Min-to-P01 gap                     | lower-tail gap chart    |
| Tail Label        | normal, moderate, extreme tail     | tail summary chart      |
| Percentile Review | whether tail behavior needs review | percentile review table |

**Dynamic Visualization Rule**

| Condition               | Visualization        |
| ----------------------- | -------------------- |
| upper tail extreme      | max-to-P99 bar chart |
| lower tail extreme      | min-to-P01 chart     |
| selected features       | percentile line plot |
| all continuous features | box-and-whisker grid |
| percentile compression  | percentile table     |

---

## 3.7 Numeric Distribution Report

| Report                   | What to Check                        | Visualization            |
| ------------------------ | ------------------------------------ | ------------------------ |
| Skewness                 | distribution asymmetry               | skewness bar chart       |
| Kurtosis                 | tail heaviness                       | kurtosis bar chart       |
| Distribution Shape       | symmetric, right-skewed, left-skewed | distribution shape chart |
| Tail Behavior            | normal, heavy, extreme               | tail behavior chart      |
| Transformation Candidate | log/sqrt/Yeo-Johnson/no-transform    | transformation table     |
| Distribution Review      | features needing manual review       | review table             |

**Dynamic Visualization Rule**

| Condition              | Visualization                              |
| ---------------------- | ------------------------------------------ |
| `abs(skewness) <= 0.5` | histogram + optional KDE                   |
| `skewness > 0.5`       | histogram + KDE + Q-Q plot                 |
| `skewness > 1`         | histogram + KDE + log histogram + Q-Q plot |
| `skewness < -0.5`      | histogram + KDE + Q-Q plot                 |
| high kurtosis          | Q-Q plot + boxplot                         |
| many features          | skewness/kurtosis bar chart                |
| important feature      | histogram + KDE + boxplot + Q-Q panel      |

---

## 3.8 Numeric Special Values Report

| Report               | What to Check                               | Visualization             |
| -------------------- | ------------------------------------------- | ------------------------- |
| Zero Count           | number of zeros                             | zero count chart          |
| Zero Percentage      | percentage of zeros                         | zero percentage chart     |
| Zero Status          | no, low, moderate, zero-heavy, dominated    | zero status chart         |
| Negative Count       | number of negative values                   | negative count chart      |
| Negative Percentage  | percentage of negative values               | negative percentage chart |
| Sentinel Values      | hidden values like -999, 9999, NA-as-string | sentinel table            |
| Absence-like Zero    | whether zero means absence                  | zero-absence table        |
| Special Value Action | keep, flag, replace, review                 | action table              |

**Dynamic Visualization Rule**

| Condition               | Visualization                    |
| ----------------------- | -------------------------------- |
| zero-heavy exists       | zero percentage chart            |
| negative exists         | negative value chart             |
| sentinel suspected      | special value table              |
| zero means absence      | zero vs non-zero count plot      |
| sparse numeric features | sorted zero percentage bar chart |

---

## 3.9 Numeric Outlier Report

| Report              | What to Check                             | Visualization            |
| ------------------- | ----------------------------------------- | ------------------------ |
| Lower Outlier Count | below lower IQR boundary                  | lower outlier chart      |
| Upper Outlier Count | above upper IQR boundary                  | upper outlier chart      |
| Total Outlier Count | total outliers                            | outlier count chart      |
| Outlier Percentage  | percentage of outliers                    | outlier percentage chart |
| Outlier Direction   | upper, lower, both                        | direction chart          |
| Outlier Severity    | no, low, moderate, high                   | severity chart           |
| Extreme Tail        | max/P99, min/P01 behavior                 | percentile plot          |
| Capping Candidate   | whether capping should be tested          | capping review table     |
| Outlier Action      | keep, review, cap-test, remove-error-only | action table             |

**Dynamic Visualization Rule**

| Condition              | Visualization                         |
| ---------------------- | ------------------------------------- |
| outliers exist         | outlier percentage chart              |
| high severity          | boxplot grid                          |
| upper outlier dominant | boxplot + percentile plot             |
| lower outlier dominant | boxplot + min/P01 table               |
| zero-heavy outlier     | zero/non-zero plot + non-zero boxplot |
| capping experiment     | before/after capping comparison plot  |

---

## 3.10 Numeric Stability Report

| Report                  | What to Check                | Visualization             |
| ----------------------- | ---------------------------- | ------------------------- |
| Dominant Value          | most frequent value          | dominant value table      |
| Dominant Count          | count of dominant value      | dominant count chart      |
| Dominant Percentage     | percentage of dominant value | dominant percentage chart |
| Constant Stability      | fully constant feature       | constant table            |
| Near-constant Stability | nearly constant feature      | near-constant table       |
| Zero-heavy Stability    | zero-dominated feature       | zero-heavy chart          |
| Stable Feature          | acceptable variation         | stable feature chart      |
| Stability Review        | features needing review      | stability review table    |

**Dynamic Visualization Rule**

| Condition                 | Visualization             |
| ------------------------- | ------------------------- |
| dominant value high       | dominant percentage chart |
| near-constant exists      | near-constant table       |
| zero-heavy exists         | zero percentage chart     |
| stable vs unstable needed | stability summary chart   |

---

## 3.11 Numeric Manual Review Report

| Report                 | What to Check                                                  | Visualization      |
| ---------------------- | -------------------------------------------------------------- | ------------------ |
| Review Needed          | whether feature needs manual review                            | review count chart |
| Review Reason          | missing, type, outlier, special value, leakage, semantic issue | reason table       |
| Data Quality Issue     | invalid value or suspicious value                              | issue table        |
| Semantic Issue         | unclear feature meaning                                        | semantic table     |
| Modeling Risk          | leakage, instability, sparse signal                            | risk table         |
| Final Numeric Decision | keep, drop, transform, flag, engineer, review                  | decision chart     |

---

## 3.12 Numeric Feature Engineering Hints Report

| Report                     | What to Check                           | Visualization            |
| -------------------------- | --------------------------------------- | ------------------------ |
| Imputation Decision        | mean/median/constant/zero/drop/review   | imputation summary chart |
| Missing Indicator Decision | create or not create missing flag       | missing indicator chart  |
| Zero Flag Decision         | create or not create zero/presence flag | zero flag chart          |
| Transformation Decision    | raw/log/sqrt/Yeo-Johnson/no-transform   | transformation chart     |
| Outlier Handling Decision  | keep/cap-test/remove-error-only/review  | outlier action chart     |
| Scaling Decision           | scaler needed for linear/KNN, not tree  | model-wise scaling table |
| Model-wise Treatment       | linear vs tree vs KNN plan              | model treatment table    |
| Final Numeric Action       | final feature engineering action        | final action chart       |

---

## 3.13 Final Numeric EDA Summary

| Report                          | What to Check                                         | Visualization           |
| ------------------------------- | ----------------------------------------------------- | ----------------------- |
| Final Numeric Groups            | continuous, count, ordinal, year, month, numeric-code | feature group chart     |
| Final Imputation Features       | features needing imputation                           | imputation group chart  |
| Final Missing Flags             | missing indicator features                            | missing flag chart      |
| Final Zero Flags                | zero/presence flag features                           | zero flag chart         |
| Final Transformation Candidates | features for transform experiment                     | transformation chart    |
| Final Outlier Review            | outlier review features                               | outlier review table    |
| Final Numeric Plan              | model-wise numeric preprocessing plan                 | final numeric dashboard |

---

# 4. Categorical Analysis

## 4.1 Categorical Basic Report

| Report                   | What to Check                   | Visualization           |
| ------------------------ | ------------------------------- | ----------------------- |
| Dtype                    | object/category/bool/string     | dtype count chart       |
| Unique Count             | number of categories            | unique count chart      |
| Unique Ratio             | unique count / row count        | unique ratio chart      |
| Sample Values            | example categories              | sample table            |
| Basic Type               | binary/low/medium/high/constant | type count chart        |
| Possible Ordinal         | natural order candidate         | ordinal candidate table |
| Possible Absence-related | missing may mean absence        | absence table           |
| Manual Type              | final semantic categorical type | manual type chart       |

---

## 4.2 Categorical Missingness Report

| Report             | What to Check                 | Visualization            |
| ------------------ | ----------------------------- | ------------------------ |
| Train/Test Missing | count and percentage          | train-test missing chart |
| Missing Type       | absence/unknown/review        | missing type chart       |
| Imputation Hint    | None/Unknown/mode/drop/review | imputation chart         |
| Missing Review     | unclear missing meaning       | review table             |

**Dynamic Visualization Rule**

| Condition              | Visualization            |
| ---------------------- | ------------------------ |
| missing exists         | missing percentage chart |
| train-test gap exists  | grouped missing chart    |
| absence-related exists | absence table            |
| high missing exists    | high-missing chart       |

---

## 4.3 Categorical Cardinality Report

| Report            | What to Check                             | Visualization       |
| ----------------- | ----------------------------------------- | ------------------- |
| Category Count    | number of unique categories               | unique count chart  |
| Cardinality Level | binary/low/medium/high                    | cardinality chart   |
| Encoding Risk     | one-hot risk / high-cardinality risk      | encoding risk chart |
| Action Hint       | one-hot/rare-group/target-encoding-review | action table        |

---

## 4.4 Dominant Category Report

| Report              | What to Check                        | Visualization             |
| ------------------- | ------------------------------------ | ------------------------- |
| Dominant Category   | most frequent category               | dominant table            |
| Dominant Percentage | dominance level                      | dominant percentage chart |
| Second Category     | second most frequent category        | top category table        |
| Dominance Status    | balanced/moderate/high/near-constant | dominance chart           |
| Action Hint         | keep/drop/review/rare-group          | action table              |

---

## 4.5 Rare Category Report

| Report                  | What to Check                     | Visualization     |
| ----------------------- | --------------------------------- | ----------------- |
| Rare Category Count     | categories below threshold        | rare count chart  |
| Rare Row Percentage     | rows belonging to rare categories | rare row chart    |
| Rare List               | rare category names               | rare table        |
| Rare Impact             | low/moderate/high                 | rare status chart |
| Rare Grouping Candidate | whether to group as Rare          | candidate table   |

**Dynamic Visualization Rule**

| Condition                | Visualization             |
| ------------------------ | ------------------------- |
| many rare labels         | top-N + rare bucket plot  |
| rare row percentage high | rare row percentage chart |
| rare grouping needed     | rare grouping table       |

---

## 4.6 Train-Test Category Consistency Report

| Report                | What to Check                  | Visualization          |
| --------------------- | ------------------------------ | ---------------------- |
| Train Categories      | categories in train            | train-test count chart |
| Test Categories       | categories in test             | train-test count chart |
| Common Categories     | shared categories              | consistency table      |
| Train-only Categories | only in train                  | train-only table       |
| Test-only Categories  | unseen in train                | test-only table        |
| Unseen Risk           | low/moderate/high              | unseen risk chart      |
| Encoding Safety       | handle unknown / align columns | safety table           |

---

## 4.7 Ordinal Categorical Report

| Report              | What to Check                 | Visualization         |
| ------------------- | ----------------------------- | --------------------- |
| Manual Ordinal      | confirmed ordered feature     | ordinal feature table |
| Mapping             | category-to-rank mapping      | mapping table         |
| Mapping Coverage    | mapped vs unmapped categories | coverage chart        |
| Unmapped Categories | categories missing in mapping | unmapped table        |
| Ordinal Action      | encode/review/fix mapping     | action table          |

---

## 4.8 Nominal Categorical Report

| Report          | What to Check                                            | Visualization       |
| --------------- | -------------------------------------------------------- | ------------------- |
| Nominal Feature | unordered categorical feature                            | nominal count chart |
| Nominal Subtype | binary/standard/dominant/rare-sensitive/high-cardinality | subtype chart       |
| Encoding Hint   | one-hot/rare-group/frequency/target-encoding-review      | encoding table      |
| Review Features | encoding risk features                                   | review table        |

---

## 4.9 Categorical Target Relationship Report

| Report             | What to Check                          | Visualization        |
| ------------------ | -------------------------------------- | -------------------- |
| Category Count     | rows per category                      | count plot           |
| Target Mean/Median | target by category                     | target bar chart     |
| Target Spread      | target variability by category         | boxplot / violinplot |
| Eta Squared        | categorical-target separation strength | eta squared chart    |
| Target Strength    | weak/moderate/strong                   | strength chart       |
| Target Action      | keep/group/review/encoding candidate   | action table         |

---

## 4.10 Categorical Encoding Hints Report

| Report                    | What to Check                     | Visualization       |
| ------------------------- | --------------------------------- | ------------------- |
| Missing Handling          | None/Unknown/mode/drop            | decision chart      |
| Rare Decision             | group or keep rare labels         | rare decision chart |
| Ordinal Encoding          | mapping needed or not             | ordinal chart       |
| One-hot Encoding          | one-hot needed or not             | one-hot chart       |
| Target Encoding Candidate | CV-safe target encoding candidate | candidate table     |
| Final Action              | final categorical action          | final action chart  |

---

## 4.11 Categorical Leakage / Drop Review

| Report                | What to Check                          | Visualization  |
| --------------------- | -------------------------------------- | -------------- |
| Constant Feature      | one category only                      | constant table |
| Near-constant Feature | one category dominates                 | dominant chart |
| ID-like Feature       | category behaves like ID               | ID-like table  |
| Leakage Suspect       | post-target or target-like information | leakage table  |
| Drop Decision         | keep/drop/review                       | decision chart |

---

## 4.12 Final Categorical EDA Summary

| Report                    | What to Check                          | Visualization          |
| ------------------------- | -------------------------------------- | ---------------------- |
| Final Ordinal Features    | ordinal encoding list                  | group chart            |
| Final Nominal Features    | one-hot/frequency/target encoding list | group chart            |
| Final Rare Group Features | rare grouping list                     | rare group chart       |
| Final Missing Groups      | None/Unknown/mode groups               | imputation group chart |
| Final Review Features     | features needing review                | review table           |
| Final Categorical Plan    | model-wise categorical preprocessing   | categorical dashboard  |

---

# 5. Datetime / Time-Series Analysis

## 5.1 Datetime Basic Report

| Report                     | What to Check                      | Visualization          |
| -------------------------- | ---------------------------------- | ---------------------- |
| Datetime Feature Detection | date/time/year/month columns       | datetime feature table |
| Parsed Datetime Validity   | valid vs invalid parsed dates      | parse success chart    |
| Min / Max Date             | earliest and latest date           | date range card        |
| Date Range                 | total time span                    | timeline plot          |
| Time Granularity           | year/month/day/hour/minute level   | granularity table      |
| Missing Datetime           | missing date count/percentage      | missing datetime chart |
| Timezone Check             | timezone consistency if applicable | timezone table         |

## 5.2 Datetime Component Report

| Report      | What to Check           | Visualization      |
| ----------- | ----------------------- | ------------------ |
| Year        | extracted year          | year count plot    |
| Quarter     | extracted quarter       | quarter count plot |
| Month       | extracted month         | month count plot   |
| Week        | extracted week          | week count plot    |
| Day         | day of month            | day count plot     |
| Day of Week | weekday/weekend pattern | weekday count plot |
| Hour        | hourly pattern          | hour count plot    |
| Season      | season feature          | season count chart |

## 5.3 Time-Series Trend Report

| Report            | What to Check                | Visualization       |
| ----------------- | ---------------------------- | ------------------- |
| Target Trend      | target movement over time    | line plot           |
| Rolling Mean      | smoothed target trend        | rolling mean plot   |
| Rolling Std       | time-varying volatility      | rolling std plot    |
| Trend Direction   | increasing/decreasing/stable | trend line          |
| Time Group Target | target by year/month/season  | grouped target plot |

## 5.4 Time-Series Quality Report

| Report                | What to Check           | Visualization              |
| --------------------- | ----------------------- | -------------------------- |
| Duplicate Timestamp   | repeated timestamps     | duplicate timestamp table  |
| Missing Timestamp Gap | gaps in time index      | gap plot                   |
| Irregular Frequency   | inconsistent intervals  | interval distribution plot |
| Future Date           | date beyond valid range | future date table          |
| Invalid Date          | impossible date values  | invalid date table         |

## 5.5 Time-Series Modeling Risk Report

| Report                    | What to Check                              | Visualization         |
| ------------------------- | ------------------------------------------ | --------------------- |
| Temporal Leakage          | future information used in past prediction | leakage table         |
| Train-Test Time Split     | chronological split validity               | train-test timeline   |
| Seasonality               | repeated periodic pattern                  | seasonal plot         |
| Lag Candidate             | target/feature lag opportunity             | lag correlation chart |
| Rolling Feature Candidate | rolling mean/std/min/max opportunity       | rolling feature table |

---

# 6. Text Analysis

## 6.1 Text Basic Report

| Report                 | What to Check                  | Visualization        |
| ---------------------- | ------------------------------ | -------------------- |
| Text Feature Detection | free-text/string-heavy columns | text feature table   |
| Text Missing           | missing text count/percentage  | missing text chart   |
| Empty String Count     | blank text values              | empty string chart   |
| Unique Text Count      | unique text entries            | unique text chart    |
| Duplicate Text Count   | repeated text values           | duplicate text table |

## 6.2 Text Length Report

| Report              | What to Check             | Visualization              |
| ------------------- | ------------------------- | -------------------------- |
| Character Count     | number of characters      | character length histogram |
| Word Count          | number of words           | word count histogram       |
| Sentence Count      | number of sentences       | sentence count histogram   |
| Average Word Length | text complexity proxy     | average word length chart  |
| Length Outlier      | extremely short/long text | text length boxplot        |

## 6.3 Text Quality Report

| Report                    | What to Check               | Visualization           |
| ------------------------- | --------------------------- | ----------------------- |
| Lowercase/Uppercase Ratio | casing pattern              | casing ratio chart      |
| Punctuation Count         | punctuation usage           | punctuation count chart |
| Digit Count               | numeric tokens in text      | digit count chart       |
| Special Character Count   | unusual characters          | special character chart |
| URL Count                 | URLs inside text            | URL count chart         |
| Email/Phone Pattern       | sensitive pattern existence | pattern table           |
| Language Detection        | possible language groups    | language count chart    |

## 6.4 Text Frequency Report

| Report          | What to Check                | Visualization        |
| --------------- | ---------------------------- | -------------------- |
| Top Words       | most frequent words          | top word bar chart   |
| Top Bigrams     | most frequent 2-word phrases | bigram bar chart     |
| Top Trigrams    | most frequent 3-word phrases | trigram bar chart    |
| Stopword Ratio  | stopword percentage          | stopword ratio chart |
| Vocabulary Size | number of unique tokens      | vocabulary size card |

## 6.5 Text Target Relationship Report

| Report                  | What to Check                        | Visualization            |
| ----------------------- | ------------------------------------ | ------------------------ |
| Text Length vs Target   | length relationship with target      | scatter/bin plot         |
| Keyword vs Target       | target behavior for keyword presence | keyword target bar chart |
| Text Category vs Target | text-derived category relationship   | boxplot/bar chart        |
| Sentiment vs Target     | sentiment relation if applicable     | sentiment target plot    |
| Text Signal Strength    | whether text feature is useful       | text signal table        |

## 6.6 Text Feature Engineering Hints

| Report                  | What to Check                          | Visualization           |
| ----------------------- | -------------------------------------- | ----------------------- |
| Clean Text Candidate    | lowercase, strip, normalize whitespace | cleaning decision table |
| Token Feature Candidate | word/char n-grams                      | n-gram summary          |
| TF-IDF Candidate        | sparse text representation             | TF-IDF plan table       |
| Embedding Candidate     | semantic representation                | embedding plan table    |
| Text Drop Candidate     | weak/noisy/free-ID text                | drop/review table       |

---

# 7. Bivariate Analysis

## 7.1 Bivariate Basic Report

| Report                | What to Check                                                                | Visualization           |
| --------------------- | ---------------------------------------------------------------------------- | ----------------------- |
| Relationship Type     | numeric-target, categorical-target, numeric-numeric, categorical-categorical | relationship type chart |
| Valid Pair Count      | non-missing pair rows                                                        | valid pair table        |
| Relationship Strength | weak/moderate/strong                                                         | strength chart          |
| Direction             | positive/negative/mixed/no trend                                             | direction chart         |
| Shape                 | linear/nonlinear/monotonic/grouped/sparse                                    | shape table             |
| Action Hint           | keep/engineer/review/drop                                                    | action table            |

## 7.2 Numeric vs Target

| Report                | What to Check                     | Visualization                  |
| --------------------- | --------------------------------- | ------------------------------ |
| Pearson Correlation   | linear numeric-target relation    | correlation bar chart          |
| Spearman Correlation  | monotonic numeric-target relation | Spearman chart                 |
| Scatter Pattern       | trend/noise/nonlinearity          | scatter plot                   |
| Nonlinear Pattern     | curved relationship               | scatter + LOWESS               |
| Outlier Influence     | extreme points affecting relation | scatter with outlier highlight |
| Zero Group Difference | zero vs non-zero target behavior  | zero flag vs target boxplot    |

## 7.3 Numeric Binned vs Target

| Report                 | What to Check                            | Visualization           |
| ---------------------- | ---------------------------------------- | ----------------------- |
| Quantile Bins          | binned numeric groups                    | binned count chart      |
| Bin Target Mean/Median | target by bins                           | ordered bin target plot |
| Monotonicity           | increasing/decreasing target across bins | monotonic line chart    |
| Nonlinear Trend        | curved bin-level trend                   | bin target curve        |

## 7.4 Categorical vs Target

| Report             | What to Check                    | Visualization        |
| ------------------ | -------------------------------- | -------------------- |
| Category Count     | rows per category                | count plot           |
| Target Mean/Median | target by category               | bar chart            |
| Target Spread      | target variation by category     | boxplot / violinplot |
| Eta Squared        | category-target separation       | eta squared chart    |
| Rare Category Risk | rare category target instability | rare vs target table |

## 7.5 Ordinal/Binary/Flag vs Target

| Report              | What to Check                  | Visualization               |
| ------------------- | ------------------------------ | --------------------------- |
| Ordinal Trend       | ordered levels vs target       | ordered line/bar plot       |
| Binary Difference   | target difference between 0/1  | binary bar/boxplot          |
| Missing Flag Signal | target differs by missing flag | missing flag vs target plot |
| Zero Flag Signal    | target differs by zero flag    | zero flag vs target plot    |
| Outlier Flag Signal | target differs by outlier flag | outlier flag vs target plot |

## 7.6 Feature vs Feature

| Report                  | What to Check                        | Visualization        |
| ----------------------- | ------------------------------------ | -------------------- |
| Numeric-Numeric         | correlation, redundancy, interaction | scatter / heatmap    |
| Numeric-Categorical     | numeric distribution by category     | boxplot / violinplot |
| Categorical-Categorical | association / contingency            | crosstab heatmap     |
| Datetime-Target         | target by time group                 | time line/bar plot   |
| Text-Target             | text-derived feature vs target       | text signal plot     |

## 7.7 Bivariate FE Candidate Summary

| Report                   | What to Check                      | Visualization             |
| ------------------------ | ---------------------------------- | ------------------------- |
| Interaction Candidate    | feature pair to combine            | interaction table         |
| Ratio Candidate          | feature ratio opportunity          | ratio scatter             |
| Binning Candidate        | numeric binning opportunity        | binned target plot        |
| Grouping Candidate       | rare/category grouping opportunity | category target chart     |
| Final Bivariate Decision | keep/create/review/drop            | final bivariate dashboard |

---

# 8. Multivariate Analysis

## 8.1 Correlation Matrix Report

| Report                 | What to Check               | Visualization                |
| ---------------------- | --------------------------- | ---------------------------- |
| Pearson Matrix         | linear numeric relation     | Pearson heatmap              |
| Spearman Matrix        | monotonic numeric relation  | Spearman heatmap             |
| Target Correlation     | feature-target correlation  | target correlation bar chart |
| High Correlation Pairs | redundant numeric pairs     | high-correlation table       |
| Clustered Correlation  | grouped correlated features | clustered heatmap            |

## 8.2 Multicollinearity / VIF Report

| Report                | What to Check                | Visualization     |
| --------------------- | ---------------------------- | ----------------- |
| VIF Score             | multicollinearity risk       | VIF bar chart     |
| High VIF Features     | features with high VIF       | high VIF table    |
| Related Feature Group | correlated feature group     | grouped heatmap   |
| Linear Model Risk     | coefficient instability risk | linear risk table |
| Action Hint           | drop/regularize/keep/review  | VIF action table  |

## 8.3 Feature Redundancy / Cluster Report

| Report                 | What to Check                   | Visualization        |
| ---------------------- | ------------------------------- | -------------------- |
| Redundant Feature Pair | duplicate/overlapping features  | redundancy table     |
| Feature Cluster        | related feature group           | cluster table        |
| Cluster Theme          | semantic feature group          | theme table          |
| Representative Feature | best feature to represent group | representative table |
| Drop Candidate         | redundant feature candidate     | keep/drop table      |

## 8.4 PCA / Dimensional Structure

| Report              | What to Check                      | Visualization                |
| ------------------- | ---------------------------------- | ---------------------------- |
| Explained Variance  | variance per component             | explained variance bar chart |
| Cumulative Variance | total variance explained           | cumulative curve             |
| Component Loadings  | important features per component   | loading bar chart            |
| PC Scatter          | projected sample structure         | PC1 vs PC2 scatter           |
| PCA Outlier         | unusual samples in component space | PC scatter with outliers     |

## 8.5 Interaction / Grouped Effects

| Report                      | What to Check                    | Visualization     |
| --------------------------- | -------------------------------- | ----------------- |
| Numeric Interaction         | numeric pair effect on target    | colored scatter   |
| Category-conditioned Effect | numeric-target trend by category | faceted scatter   |
| Ratio Feature Candidate     | ratio relation with target       | ratio scatter     |
| Grouped Target Trend        | target by group and bin          | grouped line plot |
| Interaction Action          | create/test/review/reject        | action table      |

## 8.6 Missingness Multivariate Pattern

| Report                        | What to Check                         | Visualization                   |
| ----------------------------- | ------------------------------------- | ------------------------------- |
| Co-missing Features           | features missing together             | co-missing heatmap              |
| Missing Indicator Correlation | correlation among missing flags       | missingness correlation heatmap |
| Missing Pattern Type          | MCAR/MAR/MNAR/absence-like            | pattern summary chart           |
| Missingness vs Target         | target relation with missing patterns | missing flag target plot        |
| Missingness Action            | impute/flag/drop/review               | action table                    |

## 8.7 Categorical Multivariate Pattern

| Report                     | What to Check                            | Visualization                   |
| -------------------------- | ---------------------------------------- | ------------------------------- |
| Categorical Association    | association between categorical features | Cramer's V heatmap              |
| Joint Category Count       | category combination frequency           | joint heatmap                   |
| Sparse Combinations        | rare category combinations               | sparse table                    |
| Category Interaction       | useful combined category candidate       | target by category-pair heatmap |
| Redundant Categorical Pair | overlapping categorical features         | redundancy table                |

## 8.8 Model-driven Multivariate Signal

| Report                       | What to Check                     | Visualization     |
| ---------------------------- | --------------------------------- | ----------------- |
| Feature Importance           | model-based importance            | importance chart  |
| Permutation Importance       | validation-based feature reliance | permutation chart |
| SHAP Importance              | contribution-based feature signal | SHAP summary plot |
| Partial Dependence Candidate | nonlinear effect review           | PDP plot          |
| Importance Stability         | fold-wise stability               | stability chart   |

## 8.9 Leakage / Data Quality Review

| Report                     | What to Check                            | Visualization         |
| -------------------------- | ---------------------------------------- | --------------------- |
| Leakage Candidate          | target/post-event information            | leakage table         |
| Suspicious Target Relation | unusually strong feature-target relation | target relation chart |
| Duplicate Signal           | duplicate or derived target-like feature | duplicate table       |
| Data Quality Risk          | inconsistent feature combinations        | anomaly table         |
| Final Leakage Decision     | keep/drop/review                         | decision table        |

## 8.10 Final Multivariate Summary

| Report                       | What to Check                             | Visualization          |
| ---------------------------- | ----------------------------------------- | ---------------------- |
| High Correlation Pairs       | redundant numeric feature pairs           | pair table + heatmap   |
| High VIF Features            | linear multicollinearity issues           | VIF chart              |
| Feature Clusters             | related feature groups                    | clustered heatmap      |
| PCA Summary                  | dimensional structure                     | variance curve         |
| Interaction Candidates       | features to combine/test                  | interaction table      |
| Missingness Groups           | co-missing feature groups                 | missingness heatmap    |
| Categorical Associations     | associated categorical pairs              | Cramer's V heatmap     |
| Model-important Features     | important features from baseline model    | importance chart       |
| Final Multivariate Decisions | final keep/drop/engineer/review decisions | multivariate dashboard |


