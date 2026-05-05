# Neural Networks for Correcting Professional Macroeconomic Forecasts

This project tests whether a neural network can improve professional macroeconomic forecasts from the **ECB Survey of Professional Forecasters (SPF)**.

The main idea is simple: instead of forecasting inflation or GDP growth directly, the model tries to predict the **forecast error** made by professional forecasters. If these errors are partly systematic, a neural network may be able to learn a correction and improve the original forecast.

## Project idea

Professional forecasters provide predictions for euro area macroeconomic variables such as:

- HICP inflation
- real GDP growth

This project focuses on:

- **inflation**
- **real GDP growth**

For each forecast, the forecast error is defined as:

```text
forecast_error = actual_value - SPF_forecast
```

The neural network is trained to predict this error:

```text
predicted_error = NN(features)
```

The corrected forecast is then:

```text
corrected_forecast = SPF_forecast + predicted_error
```

The goal is to compare the original SPF forecast with the neural-network-corrected forecast.

## Research question

Can a neural network predict professional macroeconomic forecast errors and improve ECB SPF forecasts for inflation and real GDP growth?

## Data

The main dataset is the **ECB Survey of Professional Forecasters (SPF)**.

The SPF individual files contain forecasts by individual forecasters. Each file corresponds to one survey round and includes several tables, including forecasts for inflation, real GDP growth, and unemployment.

For this project, the notebook uses:

- individual forecaster point forecasts
- rolling one-year-ahead forecasts
- HICP inflation forecasts
- real GDP growth forecasts

The rolling one-year-ahead horizon is used because it gives a more consistent forecasting setup than mixing current-year and next-year forecasts.

## Methodology

The notebook follows these steps:

1. Download the ECB SPF data directly in the notebook.
2. Parse the individual survey-round CSV files.
3. Extract inflation and real GDP growth forecasts.
4. Keep rolling one-year-ahead point forecasts.
5. Build a clean forecaster-level dataset.
6. Add realized inflation and GDP growth outcomes.
7. Define forecast errors.
8. Create predictor variables.
9. Train benchmark models.
10. Train a neural network.
11. Compare raw SPF forecasts with corrected forecasts.

## Main features

Possible predictors include:

- individual SPF forecast
- average SPF forecast
- median SPF forecast
- disagreement across forecasters
- number of forecasters in the survey round
- distance between individual forecast and consensus forecast
- previous forecast error
- survey year
- survey quarter
- crisis or high-inflation period indicators

## Models

The neural network is compared against simple benchmarks:

- raw SPF forecast, with no correction
- mean forecast-error correction
- linear regression
- neural network correction

The neural network only adds value if it improves out-of-sample performance compared with these baselines.

## Evaluation

Forecast performance is evaluated using:

- RMSE
- MAE
- out-of-sample performance
- comparison of raw SPF forecasts and corrected forecasts

The main output is a table comparing model performance for inflation and real GDP growth.

## Repository structure

```text
.
├── notebook.ipynb          # Main project notebook
├── data/                   # Optional local data folder
├── outputs/                # Figures and results tables
├── nn_replication_a100.ipynb  # Neural network replication notebook that will be used as base, now it need a lot of changes
└── README.md               # Project description
```

## Expected output

The final notebook should show whether a neural network can improve professional forecasts by learning systematic forecast errors.

A successful result would mean that the neural-network-corrected forecast has lower prediction error than the original SPF forecast.

## Notes

This is a data science project, not a full macroeconomic forecasting paper. The focus is on building a clean, reproducible notebook that downloads the data, prepares the forecast-error target, trains models, and evaluates the results clearly.
