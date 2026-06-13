
# SVR-LSTM

Forecasting the log-transformed Sentiment-Volatility Ratio using Long Short-Term Memory networks.

## Overview

This repository contains the LSTM forecasting implementation developed for the graduate data science capstone:

*Predictive Models for the Diagnostic Ratio of Consumer Sentiment and Volatility*

The project models the relationship between consumer sentiment and market volatility through the Sentiment-Volatility Ratio, or SVR.

The ratio is derived from:

* the University of Michigan Consumer Sentiment Index (`UMCSENT`);
* the CBOE Volatility Index (`VIXCLS`).

The modeled series is:

```text
log(SVR) = log(UMCSI) - log(monthly mean VIX)
```

The script retrieves the source data from the Federal Reserve Economic Data service, constructs the monthly log-SVR series, trains a grid of LSTM models, evaluates their out-of-sample performance, and produces forecast visualizations.

## Data

The historical analysis covers monthly observations from January 1990 through December 2025.

The final modeling dataset contains:

* 432 monthly observations;
* 348 training observations from January 1990 through December 2018;
* 84 test observations from January 2019 through December 2025.

Daily VIX observations are aggregated into monthly means before the ratio is calculated.

## Preprocessing

The modeling workflow:

1. Retrieves `VIXCLS` and `UMCSENT` from FRED.
2. Aggregates daily VIX observations into monthly means.
3. Applies logarithmic transformations to UMCSI and monthly mean VIX.
4. Constructs the log-transformed SVR.
5. Orders the monthly observations chronologically.
6. Reserves the final 84 observations as the test period.
7. Estimates the mean and standard deviation from the training data.
8. Applies the training-derived scaler to both the training and test series.

## Supervised Sequence Construction

The univariate log-SVR series is converted into three-dimensional arrays suitable for LSTM input.

Each input array has the form:

```text
samples x timesteps x features
```

The implementation uses one feature and evaluates the following lag windows:

```text
3, 6, 9, 12, 15, 18, and 24 months
```

Two direct forecast horizons are evaluated:

* one month ahead;
* three months ahead.

For each sample, the lagged observations are ordered from oldest to newest.

## LSTM Architecture

Each candidate model uses:

* one LSTM layer with 16 units;
* one dense output unit;
* a linear regression output;
* the Adam optimizer;
* mean squared error as the training loss;
* mean absolute error as a reported Keras metric.

Training uses:

* a maximum of 300 epochs;
* a batch size of 32;
* the final 20% of the supervised training samples as internal validation data;
* chronological sample order with shuffling disabled;
* early stopping on validation loss;
* patience of 20 epochs;
* restoration of the weights from the best validation epoch.

Seven lag windows across two forecast horizons produce 14 model configurations.

## Evaluation

Each model is evaluated on the out-of-sample test period.

The script reports:

* mean squared error;
* root mean squared error;
* mean absolute error.

Metrics are retained on both:

* the standardized modeling scale;
* the inverse-transformed log-SVR scale.

Model configurations are ranked separately by forecast horizon using test MAE on the log-SVR scale.

The historically selected configurations are:

| Forecast horizon   | Lag window |            Test MAE |
| ------------------ | ---------: | ------------------: |
| One month ahead    |  15 months | Approximately 0.145 |
| Three months ahead |  18 months | Approximately 0.227 |

## Outputs

The script produces:

```text
LSTM Metrics SELECTED FINAL.csv
LSTM CURRENT FORECAST-H1.png
LSTM CURRENT FORECAST-H3.png
```

The CSV contains the ranked evaluation results for all 14 model configurations.

The PNG files compare actual and predicted log-SVR values for the selected one-month-ahead and three-month-ahead models. Recession periods are shaded using the FRED `USREC` series.

Generated CSV and PNG files are excluded from Git tracking and remain available for local analysis.

## Primary Script

```text
SVR-LSTM.R
```

The script contains the complete workflow:

* FRED data retrieval;
* log-SVR construction;
* chronological partitioning;
* training-derived scaling;
* supervised lag-window construction;
* LSTM training and early stopping;
* test evaluation;
* prediction alignment;
* inverse transformation;
* results aggregation;
* CSV export;
* forecast visualization.

## Dependencies

The R script uses:

* `pipewelder`
* `tidyverse`
* `lubridate`
* `keras3`

A functioning Keras backend is also required to train and evaluate the LSTM models.

The `pipewelder` package provides the `get_fred()` helper used to retrieve and prepare the FRED series.

## Running the Analysis

Run the script from an R environment with the required packages and Keras backend configured:

```r
source("SVR-LSTM.R")
```

Training all 14 model configurations may take time depending on the available hardware and backend configuration.

Output files are written to the active R working directory.

## License

This project is licensed under the PolyForm Noncommercial License 1.0.0.

Commercial use requires separate permission from Leg3.
