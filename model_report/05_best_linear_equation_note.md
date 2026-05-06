
# Simplified Linear Equation - ElasticNet_RobustScaler_baseline

The equation below uses the top coefficient terms from the best linear model.

Important note:

The model was trained inside a RobustScaler pipeline.  
Therefore, coefficients are in scaled feature space, not raw feature space.

This equation is useful for interpretation, not for manual raw-price calculation.

```text
Predicted log1p(SalePrice) ≈ 12.0177 - 0.2734×MSZoning__C (all) + 0.1087×Neighborhood__Crawfor + 0.1000×Neighborhood__StoneBr + 0.0927×LivingPlusBasementSF + 0.0863×OverallQual + 0.0762×Neighborhood__NoRidge + 0.0723×GrLivArea + 0.0705×Exterior1st__BrkFace - 0.0614×Condition1__RRAe - 0.0611×KitchenAbvGr + 0.0593×KitchenQual__Ex - 0.0552×Heating__Grav