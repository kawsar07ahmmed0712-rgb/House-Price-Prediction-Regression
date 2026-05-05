
# Final Feature Engineering Report

## Scope

This notebook completed feature engineering only.

No model training was performed here.

## Final Dataset Outputs

| Dataset | Shape | Purpose |
|---|---:|---|
| Tree train | (1458, 343) | Tree-based model training |
| Tree test | (1459, 343) | Tree-based model prediction |
| Linear train | (1458, 343) | Linear model training |
| Linear test | (1459, 343) | Linear model prediction |

## Target

- Raw target: `SalePrice`
- Modeling target: `log1p(SalePrice)`

## Main Feature Engineering Decisions

- Removed selected training outliers only.
- Treated `MSSubClass` as categorical.
- Filled absence-related categorical missing values as `NoFeature`.
- Filled absence-related numeric missing values as `0`.
- Created core engineered features.
- Created ordinal score features.
- Created component score features.
- Created quality × size interaction features.
- Applied rare grouping and one-hot encoding.
- Created tree-based final dataset without scaling.
- Created linear final dataset with selected log transformations but without scaling.

## Final Validation

All final datasets passed:

- row alignment check
- train/test column alignment check
- missing value check
- infinite value check
- object column check
- duplicate column check

## Important Modeling Note

Scaling was not applied in this notebook.

For linear models, scaling should be fitted inside the future model training CV pipeline.

## Next Notebook

Use these outputs in:

`Model_Training_Research.ipynb`
