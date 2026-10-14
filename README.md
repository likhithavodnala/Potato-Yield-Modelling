
# Potato Yield Modelling Using Weather Data — Mehsana, Gujarat

## Project overview

This project investigates whether weather conditions during the potato-growing season can help explain year-to-year variation in potato productivity in the Mehsana region of Gujarat. The modelling workflow combines weekly weather indicators with historical productivity and compares several regression and machine-learning models.

The practical motivation is that potato yield varies from season to season, and understanding weather-related variation may support crop monitoring and agricultural planning. The models are statistical tools, not guarantees of future yield.

## Study area and data

- **Study area:** Mehsana, Gujarat, India
- **Weather data period:** 1996–2025
- **Modelled seasons in the prediction file:** 1997–98 to 2024–25 (**28 seasons**)
- **Growing-season window:** Standard Meteorological Week (SMW) 42 through SMW 8, crossing the calendar-year boundary
- **Weather indicators:** maximum temperature (MAXT), minimum temperature (MINT), mean relative humidity (MEAN RH), wind speed (WS), and bright sunshine duration/brightness-related measure represented by BSS in the prepared features
- **Target:** potato productivity/yield
- **Additional predictor:** lag-1 yield anomaly, representing the previous season's yield deviation after detrending

The supplied `season_weekly_features.csv` contains 29 rows, including the 1996–97 feature row; the final prediction file contains 28 modelled seasons, from 1997–98 through 2024–25. The model-comparison results should be interpreted for those 28 prediction seasons.

## Methodology

1. Prepare seasonal weather features for SMW 42–52 and SMW 1–8.
2. Organise weather predictors by meteorological week, producing week-specific candidate features.
3. Separate long-term yield trend from year-to-year yield anomalies so the model can focus on interannual variation.
4. Apply feature selection within the validation workflow to reduce the number of predictors.
5. Compare six models using nested leave-one-out cross-validation (LOOCV), as recorded by the project workflow.
6. Evaluate the model comparison using R², RMSE and MAE. Predictions are also reported on the original yield scale.

The model is **Multiple Linear Regression (MLR)**. `SelectKBest (F-test)` is its selected feature-selection method in the reported comparison; it is not a separate model.

## Model comparison: reported results

The following values are copied from `model_comparison_results.csv`.

| Model | Feature-selection method | Number of features | R² | RMSE | MAE |
|---|---|---:|---:|---:|---:|
| **Multiple Linear Regression (MLR)** | SelectKBest (F-test) | 5 | **0.59** | **1.231** | **0.948** |
| XGBoost | Forward Selection | 5 | 0.45 | 1.424 | 1.165 |
| K-Nearest Neighbours (KNN) | Forward Selection | 6 | 0.45 | 1.434 | 1.265 |
| Support Vector Regression (SVR) | Forward Selection | 5 | 0.41 | 1.483 | 1.258 |
| Random Forest | SelectKBest (F-test) | 5 | 0.39 | 1.506 | 1.120 |
| Artificial Neural Network (ANN/MLP) | SelectKBest (F-test) | 7 | 0.34 | 1.567 | 1.252 |

**Interpretation:** MLR is the best-performing model in this comparison. It has the highest reported R² (0.59) and the lowest RMSE (1.231) and MAE (0.948) among the six models. XGBoost and KNN are tied at the displayed R² of 0.45, but their error values differ. Random Forest has a lower MAE than XGBoost, KNN and SVR, despite its lower R². This shows why model quality should be assessed using more than one metric.

The `**` suffix attached to R² in the CSV has been omitted in the table for readability; the CSV does not define the meaning of that suffix. Do not interpret it as a significance result unless the notebook explains it.

## Original-scale observed versus LOOCV-predicted yield

The following summary is calculated from all 28 rows in `final_predictions.csv`. It uses the **LOOCV predictions**, which are more appropriate for describing held-out prediction performance than predictions from a model refitted on the full dataset.

| Metric | LOOCV predictions on original yield scale |
|---|---:|
| Number of seasons | 28 |
| R² | 0.932 |
| RMSE | 1.230 MT/ha |
| MAE | 0.948 MT/ha |
| Mean observed yield | 23.378 MT/ha |
| Minimum observed yield | 16.46 MT/ha |
| Maximum observed yield | 29.98 MT/ha |

The LOOCV original-scale metrics above are recalculated from `Observed Yield (MT/ha)` and `Predicted Yield - LOOCV (MT/ha)` in the supplied prediction file. They are **not the same target-scale calculation** as the R²/RMSE/MAE in the model-comparison table, which reports the detrended-anomaly model comparison. Therefore, the two sets of R² values should not be compared as if they measured exactly the same target.

For reference, the in-sample/refit predictions in the same file produce R² ≈ 0.960, RMSE ≈ 0.938 MT/ha and MAE ≈ 0.748 MT/ha. These are fitted-data results and are expected to look more optimistic than held-out LOOCV estimates, so they are not used as the main performance claim.

## Output files

- `Potato_Yield_Weather_Modelling_CORRECTED (1).ipynb` — modelling workflow and analysis
- `model_comparison_results.csv` — model metrics, feature-selection method and selected features
- `final_predictions.csv` — observed yield, refit predictions and LOOCV predictions by season
- `season_weekly_features.csv` — seasonal features organised by meteorological week
- `potato_yield_model_results.xlsx` — consolidated results workbook

## Limitations and interpretation

- The prediction file contains 28 seasons. A small number of annual observations limits how confidently results can generalise to future seasons.
- Weather relationships may vary by cultivar, irrigation, soil, planting date, pest and disease pressure, and management practices. These factors are not represented explicitly in the listed selected features.
- LOOCV is useful with small datasets, but its estimates can still be uncertain when observations are few or seasons are not fully independent.
- Selected features indicate predictive usefulness within this dataset and workflow; they should not be interpreted as proof of causation.
- Before operational use, evaluate the model on newly observed seasons and confirm that the data preparation and yield units are consistent.

## Conclusion

In the reported model comparison, **Multiple Linear Regression with SelectKBest (F-test)** performed best, with R² = 0.59, RMSE = 1.231 and MAE = 0.948 for the detrended-yield-anomaly comparison. Its selected predictors were WS in SMW 49, 52 and 1, MINT in SMW 43, and lag-1 yield anomaly. On the original yield scale, the 28 LOOCV predictions give a recalculated R² of approximately 0.932, RMSE of 1.230 MT/ha and MAE of 0.948 MT/ha. These findings are promising within the available data, but should be validated with future seasons before the model is used for practical forecasting.
