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

The project uses individual forecaster-level data from the ECB SPF. The focus is on:

- HICP inflation
- real GDP growth
- rolling one-year-ahead forecasts

The rolling one-year horizon is used because it gives a consistent forecasting setup across survey rounds.

External predictors are added from public sources, including Eurostat, ECB SDW, Yahoo Finance, FRED, and European Commission sentiment indicators. Step 2 also includes EU country-level macro variables such as inflation, GDP growth, unemployment, and industrial production, since country-level conditions can affect euro-area outcomes.

## Project Workflow

The current pipeline is split into three notebooks:

1. `step1_data_cleaning.ipynb`
   - cleans the raw SPF individual files
   - builds long and wide SPF datasets
   - classifies rolling forecast horizons

2. `step2_predictor_dataset.ipynb`
   - builds the external predictor dataset
   - adds euro-area macro-financial variables
   - adds EU country-level macro predictors
   - saves `Data/external_predictors.csv`

3. `step3_final_dataset.ipynb`
   - merges SPF forecasts with external predictors
   - adds realized Eurostat outcomes
   - filters to valid rolling one-year forecasts
   - calculates the neural-network target variables
   - saves `Data/final_model_dataset.csv`

## Main Output Files

```text
Data/spf_clean_long.csv          # Clean SPF data in long panel format
Data/spf_clean_wide.csv          # Clean SPF data in wide forecaster-level format
Data/external_predictors.csv     # Survey-round external predictors
Data/final_model_dataset.csv     # Final dataset for model training
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
