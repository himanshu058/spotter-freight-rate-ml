# Spotter Freight Rate Prediction

Machine learning solution for the Spotter Freight Rate Prediction assessment.

## Overview

This project predicts `posted_rate` for freight loads using shipment, route, equipment, and market-related features.

The workflow covers:

* Exploratory data analysis
* Data quality checks
* Time-based train/validation splitting
* Feature preprocessing
* Model comparison
* Error analysis
* Final prediction generation
* December fixed-lane prediction and scorer validation

## Data

The labeled development dataset is:

```text
data/train_test.csv
```

The final validation dataset is:

```text
data/validation.csv
```

The December prediction inputs are:

```text
data/december_chart_inputs.csv
```

The final validation output is:

```text
validation_predictions.csv
```

with the required columns:

```text
load_id,predicted_rate
```

## Validation Approach

The development data contains 48,000 labeled loads.

I used a chronological split to make validation more representative of predicting future loads:

* Training: 43,147 rows, January through September 2025
* Validation: 4,853 rows, October 2025

This approach avoids randomly mixing future observations into the training set.

## Exploratory Findings

Some of the main findings from the analysis were:

* Distance had a strong relationship with posted rate, with a correlation of approximately 0.91.
* Equipment type showed meaningful differences in average freight rates.
* Rate-per-mile had a long right tail with a small number of unusually high observations.
* The dataset contained negative and missing weight values, which were investigated during data-quality checks.
* Approximately 12% of the validation rows represented routes not seen in the training portion.

## Model Selection

Several approaches were tested, including:

* Numerical HistGradientBoosting
* Categorical HistGradientBoosting
* Log-target HistGradientBoosting
* XGBoost
* Feature-engineered HistGradientBoosting
* Route-based and route-frequency features

The categorical HistGradientBoosting model produced the lowest validation MAE among the tested approaches.

Development validation result:

| Metric | Result |
| ------ | -----: |
| MAE    | 129.18 |
| RMSE   | 654.62 |

These are local development-validation results and are not the hidden Spotter evaluation score.

## Final Model

The final model uses:

### Numerical features

```text
distance
weight
market_index
quote_signal
pickup_lat
pickup_lon
delivery_lat
delivery_lon
```

### Categorical features

```text
pickup
delivery
equipment
```

The final HistGradientBoosting configuration was:

```text
max_iter = 300
learning_rate = 0.05
max_leaf_nodes = 31
random_state = 42
```

## Final Predictions

The final model generated predictions for all 12,000 validation loads.

The output was checked for:

* Exactly 12,000 rows
* Unique `load_id` values
* Missing predictions
* Non-positive predictions
* Correct output column order

The resulting file is:

```text
validation_predictions.csv
```

## December Prediction

The required December scenario uses:

```text
Pickup: Lexington
Delivery: Fort Wayne
Distance: 360 miles
Equipment: Dry Van
Weight: 32,000 lb
Dates: December 1–31, 2025
```

The generated December predictions are included in:

```text
data/december_chart_inputs.csv
```

The required chart is generated at:

```text
scorer_results/candidate_december.png
```

## Running the Scorer

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

Run:

```bash
python score.py --predictions validation_predictions.csv --december-predictions data/december_chart_inputs.csv
```

Successful validation produces:

```text
Validated 12,000 final predictions.
Validated 31 fixed December predictions.
Created chart: scorer_results\candidate_december.png
```

## Repository Structure

```text
spotter-freight-rate-ml/
│
├── data/
│   ├── train-test.csv
│   ├── validation.csv
│   ├── validation-predictions-template.csv
│   └── december_chart_inputs.csv
│
├── notebooks/
│   └── exploration.ipynb
│
├── reports/
│   └── Spotter_Freight_Rate_ML_Assessment_Report.pdf
│
├── scorer_results/
│   └── candidate_december.png
│
├── requirements.txt
├── readme.md
├── score.py
└── validation_predictions.csv
```

## Assessment Report

The detailed analysis, validation approach, model comparison, error analysis, and December prediction chart are available in:

`reports/Spotter_Freight_Rate_ML_Assessment_Report.pdf`
