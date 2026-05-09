# Neural Networks for Correcting Professional Macroeconomic Forecasts

This project tests whether a neural network can improve professional macroeconomic forecasts from the **ECB Survey of Professional Forecasters (SPF)**.

The model does not forecast inflation or GDP growth directly. It learns the forecast error made by the SPF forecaster:

```text
forecast_error = actual_value - rolling_1y_forecast
```

The corrected forecast is then:

```text
corrected_forecast = rolling_1y_forecast + predicted_error
```

The main question is whether professional forecast errors contain systematic patterns that a neural network can learn out of sample.

## Research Question

Can a neural network predict professional forecast errors and improve ECB SPF forecasts for euro-area inflation and real GDP growth?

## Data

The project combines three data sources:

1. **ECB SPF individual forecasts**
   - forecaster-level survey data
   - HICP inflation and real GDP growth
   - rolling one-year-ahead forecasts are the main target horizon

2. **EA-MD-QD euro-area macro dataset**
   - maintained quarterly euro-area macro and financial panel
   - includes GDP, inflation, labour market, industrial production, interest rates, confidence, money, and market variables
   - processed output used here: `Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx`

3. **Eurostat realized outcomes**
   - official realized HICP inflation and real GDP growth
   - used to calculate the realized forecast-error target

The final model dataset starts in **2000Q2**, because earlier SPF rows do not have matched EA-MD-QD macro predictors. Rows without macro predictors or realized target values are dropped before training.

## How the Final Dataset Is Built

`step3_final_dataset.ipynb` builds the modelling file from the cleaned SPF wide panel. It keeps only observations with a valid `rolling_1y_forecast`, downloads realized HICP and RGDP values from Eurostat, and calculates:

```text
forecast_error = actual_value - rolling_1y_forecast
```

The final dataset also adds:

- SPF consensus, disagreement, and distance-from-consensus features for available forecast horizons
- horizon-shape features such as `near_term_tilt`, `forward_acceleration`, and `curvature`
- leak-safe past forecast-error features
- lag-aligned EA-MD-QD macro and financial predictors
- engineered macro predictors: `gdp_yoy_growth`, `hicp_yoy_growth`, and `term_spread`
- a 95% missing-value filter for sparse candidate predictors

The EA-MD-QD merge is conservative. For an SPF survey in quarter `t`, the model uses the latest completed EA-MD-QD quarter before that survey. For example, a `2012Q1` survey uses `2011Q4` macro predictors. This avoids giving the model information that forecasters may not have had yet.

Past forecast-error features are also built conservatively. A past error is only used after its target period is old enough to plausibly be known, so future realized values do not leak into the model.

## Project Workflow

All notebooks resolve files from the repository root. After cloning, keep the
`Data/` folder in the project root with the structure shown below; the notebooks
will work whether Jupyter starts in the repo root or inside a subfolder.

`data_download_and_processing.ipynb` is the combined version of the data
pipeline: it runs the SPF cleaning step first and then builds the final modelling
dataset.

1. `step1_data_cleaning.ipynb`
   - cleans the raw SPF individual files
   - builds long and wide SPF datasets
   - classifies rolling one-year, rolling two-year, and longer forecast horizons

2. `Data/EA-MD-QD-2026-04/RUN.ipynb`
   - runs the maintained EA-MD-QD preprocessing routine, `routine_data.py`
   - processes `EAdata.xlsx` into a quarterly euro-area predictor panel
   - saves `Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx`

3. `step3_final_dataset.ipynb`
   - merges SPF forecasts with EA-MD-QD predictors
   - adds realized Eurostat outcomes
   - calculates the neural-network target variables
   - saves `Data/final_model_dataset.csv`

4. `nn_replication_a100.ipynb`
   - loads the final dataset from Google Drive or the local `Data/` folder
   - trains separate neural networks for HICP and RGDP forecast errors
   - uses chronological expanding-window out-of-sample splits
   - compares raw SPF forecasts with NN-corrected forecasts

## Main Output Files

```text
Data/spf_clean_long.csv          # Clean SPF data in long panel format
Data/spf_clean_wide.csv          # Clean SPF data in wide forecaster-level format
Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx  # Processed EA macro predictors
Data/final_model_dataset.csv     # Final dataset for model training
```

Current final dataset:

```text
Rows: 9,810
Columns: 178
Survey rounds: 2000Q2 to 2025Q3
Variables: HICP and RGDP
Rows by variable: 4,865 HICP and 4,945 RGDP
```

The same final CSV was also uploaded to Google Drive as:

```text
final_model_dataset.csv
final_model_dataset_macro_matched.csv
```

## Running the EA-MD-QD Script

Use `Data/EA-MD-QD-2026-04/RUN.ipynb` when the raw EA-MD-QD workbook changes or when the processed file needs to be rebuilt.

That notebook is a small wrapper around the authors' own `routine_data.py` script. We use their script because it applies the dataset's intended transformations and imputation rules, instead of manually recreating them.

Current settings:

```text
Country: EA
Frequency: QM
Transformation: light
Impute missing values: yes
Imputation method: 0
```

The resulting file is:

```text
Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx
```

## Neural Network Setup

`nn_replication_a100.ipynb` trains separate models for HICP and RGDP. The current model input has **156 numeric features** after excluding identifiers, target columns, and raw non-target forecast horizons.

The notebook currently uses:

- feed-forward neural networks `NN1` to `NN5`
- batch normalization, ReLU activations, and dropout
- train-only median imputation and standardization
- Huber loss for robustness to large COVID and post-COVID errors
- train-window target winsorization
- train-window prediction clipping
- validation-window OLS shrinkage of NN corrections
- target-period sample weights so survey rounds with more forecasters do not dominate the loss
- annual expanding-window refits with out-of-sample test years from 2012 to 2025

The raw SPF one-year forecast remains in the feature set because it is the forecast being corrected. Other raw SPF horizon forecasts are dropped by default, while their consensus and engineered horizon-shape information is retained.

## Latest Results

The latest full training run includes `NN1`, `NN2`, `NN3`, `NN4`, `NN5`, and an average across trained networks called `NN_AVG`.

At the forecaster-row level, the strongest result is for inflation:

```text
HICP NN1:
Raw SPF RMSE:          2.3535
NN-corrected RMSE:     2.1727
R2_zero:              14.7766%
R2_bias:              19.1669%
```

At the target-period average level, `HICP NN1` also improves RMSE:

```text
Raw SPF RMSE:          2.2856
NN-corrected RMSE:     2.1344
RMSE improvement:      6.6151%
DM p-value:            0.3495
```

So the inflation model improves the SPF forecast economically, but the Diebold-Mariano-style test does not reject equal predictive accuracy at the 5% level.

For real GDP growth, the neural network is weaker. The best RGDP models reduce some bias-benchmark errors, but they do not beat the raw SPF forecast in RMSE. In the latest run, `RGDP NN4` has the best `R2_bias`:

```text
RGDP NN4:
Raw SPF RMSE:          2.4701
NN-corrected RMSE:     2.5268
R2_zero:              -4.6440%
R2_bias:               1.3052%
```

This means the RGDP model can sometimes beat a simple historical-bias correction, but it still performs worse than leaving the raw SPF forecast unchanged.

## Environment

The project uses a local Python 3.12.12 virtual environment in `.venv`. The notebooks are configured to use the kernel:

```text
Python 3.12.12 (.venv Final Assignment)
```

## Notes

This is a data science project, not a full macroeconomic forecasting paper. The pipeline is now clean enough to support the main experiment: build a leakage-aware forecast-error dataset, train neural networks out of sample, and test whether the corrections improve professional SPF forecasts.
