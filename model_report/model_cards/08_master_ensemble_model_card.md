
# Master Ensemble Model Card

## Purpose
Create stronger prediction using OOF-based log-space blending and meta-model stacking.

## Runtime
3.68 minutes

## Best Weighted Blend
- OOF log RMSE: 0.10716
- Submission: `submission_master_weighted_blend.csv`

## Best Meta Stack
- Meta model: Ridge_meta_alpha_0.3
- OOF log RMSE: 0.10637
- Submission: `submission_master_stack.csv`

## Selected Blend Models
- Ridge_RobustScaler_baseline: 0.02152
- ElasticNet_robust_tuned: 0.24965
- ElasticNet_RobustScaler_baseline: 0.04861
- SVR_RBF_RobustScaler_master: 0.26967
- Ridge_RobustScaler_alpha10_master: 0.03077
- BayesianRidge_RobustScaler_master: 0.01279
- Ridge_RobustScaler_alpha30_master: 0.01929
- CatBoost_master_v2: 0.34770

## Decision
Use Ridge baseline as public-safe anchor.  
Use OOF blend and stack only if their CV score improves and prediction distribution looks sane.
