# Freight Rate Prediction Challenge

This project predicts total freight load rates using CatBoost. It includes data exploration, cleaning, chronological validation, model comparison, final predictions, and a fixed-input December prediction chart.

## Files

- `train_model.ipynb`: main notebook for training, evaluation, and generating predictions.
- `data/`: original input datasets and prediction template.
- `requirements.txt`: project dependencies.
- `model_comparison.csv`: validation results.
- `validation_predictions.csv`: predictions for all 12,000 final load IDs.
- `december_predictions.csv`: 31 daily predictions for the fixed December route.
- `score.py`: supplied file checker and chart generator.
- `scorer_results/candidate_december.png`: generated December chart.
- `assessment-readme.md`: original assessment instructions.

## Setup

Run these commands from the project folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install ipykernel
```

Open `train_model.ipynb` in VS Code with the Python and Jupyter extensions installed. Select the `.venv` Python environment as the notebook kernel.

Run the notebook cells in order from top to bottom. The notebook reads files relative to the project folder.

## Data Preparation

The labeled development data contains 48,000 loads from January through October 2025.

- Replaced 292 zero or negative weights with missing values.
- Retained the 300 originally missing weights.
- Used CatBoost's handling of missing numeric values.
- Excluded `load_id` from model inputs.
- Derived month and day-of-week features from the date.
- Treated pickup, delivery, and equipment as categorical features.
- Retained high-rate observations because a high value alone does not establish a data error.

## Validation

I used a chronological split:

- Training: January–August 2025, containing 38,477 loads.
- Validation: September–October 2025, containing 9,523 loads.

The validation period was used for model comparison and early stopping. These are validation results, not independent final test results.

| Model | Validation MAE | Validation RMSE |
|---|---:|---:|
| Distance benchmark | $256.95 | Not calculated |
| Equipment benchmark | $229.10 | Not calculated |
| CatBoost with MAE loss | $115.26 | $635.96 |
| CatBoost with RMSE loss | $108.44 | $632.92 |

The selected model used RMSE training loss and validation MAE for early stopping, with:

- Learning rate: 0.05
- Tree depth: 6
- Maximum iterations: 500
- Early stopping patience: 50 rounds
- Random seed: 42

Validation selected 316 trees. The final model was retrained on all 48,000 labeled loads using that training length.

## December Chart

The December inputs omit coordinates, `market_index`, and `quote_signal`. I trained a separate model using the available fields and derived date features.

This model achieved $120.10 validation MAE. Validation selected 272 trees, which were used when retraining on all labeled loads.

The chart holds the following inputs fixed:

- Lexington to Fort Wayne
- 360 miles
- Dry Van
- 32,000 lb

The weekly pattern results from the weekday feature repeating while the other inputs remain constant. It represents model predictions rather than observed December prices.

## Validate Predictions and Generate the Chart

After running the notebook, run:

```bash
python score.py \
  --predictions validation_predictions.csv \
  --december-predictions december_predictions.csv
```

The checker validates both prediction files and generates:

```text
scorer_results/candidate_december.png
```

It successfully validated 12,000 final predictions and 31 December predictions. The checker does not calculate prediction accuracy; Spotter calculates final metrics after submission.

## Limitations

- Some unusually expensive loads have large prediction errors.
- November and December are outside the labeled development period.
- Month and weekday features do not establish reliable seasonality for unseen months.
- Final prediction accuracy is unknown because the final dataset has no supplied labels.