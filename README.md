# Neural Networks for Correcting Professional Macroeconomic Forecasts

This project tests whether a neural network can improve professional macroeconomic forecasts from the **ECB Survey of Professional Forecasters (SPF)**.

Instead of forecasting inflation or GDP growth directly, the model learns the forecast error made by professional forecasters:

```text
forecast_error = actual_value - SPF_forecast
```

The corrected forecast is then:

```text
corrected_forecast = SPF_forecast + predicted_error
```

The main question is whether these forecast errors contain systematic patterns that a neural network can learn.

## Research Question

Can a neural network predict professional forecast errors and improve ECB SPF forecasts for euro-area inflation and real GDP growth?

## Data

The project combines three data sources:

1. **ECB SPF individual forecasts**
   - forecaster-level survey data
   - HICP inflation and real GDP growth
   - rolling one-year-ahead forecasts are the main horizon

2. **EA-MD-QD euro-area macro dataset**
   - maintained quarterly euro-area macro and financial panel
   - includes GDP, inflation, labour market, industrial production, interest rates, confidence, money, and market variables
   - processed output used here: `Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx`

3. **Eurostat realized outcomes**
   - official realized HICP inflation and real GDP growth
   - used to calculate the forecast-error target

The final model dataset starts in **2000Q2**, because earlier SPF rows do not have matched EA-MD-QD macro predictors. Rows without macro values are dropped before training.

## How the Final Dataset Is Built

The neural network does not predict inflation or GDP growth directly. It predicts:

```text
forecast_error = actual_value - rolling_1y_forecast
```

`step3_final_dataset.ipynb` builds this dataset as follows:

- starts from `Data/spf_clean_wide.csv`
- keeps only rows with a valid rolling one-year-ahead SPF forecast
- downloads realized HICP and RGDP outcomes from Eurostat
- calculates `forecast_error`, `abs_forecast_error`, and `squared_forecast_error`
- adds SPF consensus features, such as the survey-round mean and disagreement
- adds leak-safe past forecast-error features
- merges EA-MD-QD predictors by survey round
- drops rows where EA-MD-QD macro predictors are missing

The EA-MD-QD merge is conservative. For an SPF survey in quarter `t`, the model uses the latest completed EA-MD-QD quarter before that survey. For example, a `2012Q1` survey uses `2011Q4` macro predictors. This avoids giving the neural network information that forecasters may not have had yet.

Past forecast-error features are also built conservatively. A past error is only used after its target period is old enough to plausibly be known, so future realized values do not leak into the model.

## Project Workflow

The current pipeline is split into the main SPF cleaning/final-dataset notebooks plus the maintained EA-MD-QD preprocessing runner:

1. `step1_data_cleaning.ipynb`
   - cleans the raw SPF individual files
   - builds long and wide SPF datasets
   - classifies rolling forecast horizons

2. `Data/EA-MD-QD-2026-04/RUN.ipynb`
   - runs the maintained EA-MD-QD preprocessing routine, `routine_data.py`
   - processes `EAdata.xlsx` into a quarterly euro-area predictor panel
   - saves `Data/EA-MD-QD-2026-04/data_TR2/EAdataQM_TR2.xlsx`

3. `step3_final_dataset.ipynb`
   - merges SPF forecasts with EA-MD-QD predictors
   - adds realized Eurostat outcomes
   - filters to valid rolling one-year forecasts
   - calculates the neural-network target variables
   - saves `Data/final_model_dataset.csv`

4. `nn_replication_a100.ipynb`
   - loads `Data/final_model_dataset.csv`
   - trains separate neural networks for HICP and RGDP forecast errors
   - uses chronological expanding-window splits

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
Columns: 155
Survey rounds: 2000Q2 to 2025Q3
Variables: HICP and RGDP
```

The same final CSV was also uploaded to Google Drive as:

```text
final_model_dataset_macro_matched.csv
```

## Running the EA-MD-QD Script

Use `Data/EA-MD-QD-2026-04/RUN.ipynb` when the raw EA-MD-QD workbook changes or when the processed file needs to be rebuilt.

That notebook is a small wrapper around the authors' own `routine_data.py` script. We use their script because it applies the dataset's intended transformations and imputation rules, instead of manually recreating them. This keeps the macro predictor panel closer to the maintained source.

To run it, open `Data/EA-MD-QD-2026-04/RUN.ipynb` and run all cells. The notebook feeds the required options into `routine_data.py` and writes the processed output automatically.

The current settings are:

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

## Modelling Idea

The neural network will be trained to predict `forecast_error`. Its correction can then be added to the original SPF forecast and compared against:

- the raw SPF forecast
- simple average-error corrections
- linear benchmark models
- the neural-network-corrected forecast

Forecast performance will be evaluated out of sample using RMSE and MAE.

## Environment

The project uses a local Python 3.12.12 virtual environment in `.venv`. The notebooks are configured to use the kernel:

```text
Python 3.12.12 (.venv Final Assignment)
```

## Notes

This is a data science project, not a full macroeconomic forecasting paper. The goal is to build a clean and reproducible pipeline, then test whether a neural network can learn useful corrections to professional forecasts.
