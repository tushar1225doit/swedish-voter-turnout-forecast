# Forecasting Voter Turnout Across Swedish Municipalities

### Comparing statistical and machine-learning models across Swedish Riksdag elections, with a prospective evaluation on the 2026 election

This project investigates whether pre-election socioeconomic and demographic characteristics can improve municipality-level voter-turnout forecasts beyond a simple historical-turnout baseline.

Using data from Sweden's 290 municipalities, the project compares a historical-turnout baseline with Linear Regression, Decision Tree Regression, and Random Forest Regression. Models are evaluated using later elections as out-of-time test sets rather than random train-test splits.

Prospective forecast:- Municipality-level predictions for the 2026 Swedish Riksdag election were generated and frozen on **9 September 2026**, before observed 2026 turnout was incorporated into the analysis.

## Research question

**Can pre-election socioeconomic and demographic characteristics improve predictions of voter turnout across Swedish municipalities beyond a simple historical-turnout baseline, and do nonlinear machine-learning models generalize better than conventional linear regression?**

## Key findings

- Municipal voter turnout shows strong persistence across elections.
- Linear Regression and Random Forest substantially improved on the historical-turnout baseline when forecasting the 2018 election.
- All fitted statistical and machine-learning models struggled with the election-wide turnout decline in 2022.
- The simple historical-turnout baseline performed best in the 2022 out-of-time test.
- Greater model flexibility did not automatically lead to better generalization across elections.
- The 2026 municipality-level forecast has been frozen for prospective evaluation after the election.

## Study design

The unit of analysis is a **municipality-election observation**.

Sweden has 290 municipalities. Historical modelling data were constructed for three target elections: 2014, 2018, and 2022. The final historical modelling dataset therefore contains **870 municipality-election observations**.

Socioeconomic and demographic predictors are aligned approximately two years before each target election:

| Predictor year | Election predicted |
|---:|---:|
| 2012 | 2014 |
| 2016 | 2018 |
| 2020 | 2022 |
| 2024 | 2026 |

Previous-election turnout is aligned separately:

| Previous turnout | Election predicted |
|---:|---:|
| 2010 | 2014 |
| 2014 | 2018 |
| 2018 | 2022 |
| 2022 | 2026 |

This timing is intended to keep the forecasting setup prospective and avoid using information observed after the target election.

## Data

Raw source files are stored in `data/raw/`, while analysis-ready datasets produced by the data-preparation notebook are stored in `data/processed/`.

### Raw data files

| Raw file | Information used | Final variable(s) |
|---|---|---|
| `ME0104B8_20260906-003101.csv` | Municipality-level turnout in Swedish Riksdag elections | `turnout_pct`, `previous_turnout_pct` |
| `income_data.csv` | Municipality-level median income | `median_income_ksek` |
| `scb_unemployment_2012_2020.csv` | Registered unemployment for historical predictor years | `registered_unemployed_pct` |
| `scb_unemployment_2024.csv` | Registered unemployment for the 2026 forecast | `registered_unemployed_pct` |
| `scb_education_2012_2024.csv` | Post-secondary educational attainment | `higher_education_pct` |
| `scb_average_age_2012_2024.csv` | Average age | `average_age` |
| `scb_population_density_2012_2024.csv` | Population density | `population_density` |

The voter-turnout data correspond to Statistics Sweden (SCB) matrix **ME0104B8**, *Voting rates in elections to the Riksdag, County Councils and Municipal Councils by region*. The original raw extracts are retained in the repository to support reproducibility.

### Processed datasets

- `data/processed/master_model_dataset_2014_2022.csv`  
  Historical modelling dataset with **870 rows × 12 columns**.

- `data/processed/forecast_features_2026.csv`  
  Prospective 2026 forecasting dataset with **290 rows × 11 columns**.

All cleaning, reshaping, year alignment, feature construction, and quality checks are performed in `notebooks/01_data_preparation.ipynb`.

## Predictors

Six predictors are used in the forecasting models:

1. `previous_turnout_pct` — turnout in the preceding Riksdag election
2. `median_income_ksek` — municipality-level median income
3. `registered_unemployed_pct` — registered unemployment percentage
4. `higher_education_pct` — percentage of residents aged 25–64 with post-secondary education among those with known education level
5. `average_age` — average age of the municipal population
6. `log_population_density` — natural-log transformation of population density

Municipality codes and municipality names are retained as identifiers but are **not** used as machine-learning predictors.

Raw population density is retained for transparency, while its log-transformed version is used in the models because the original distribution is strongly right-skewed.

## Forecasting models

Four forecasting approaches are compared:

| Model | Purpose in the comparison |
|---|---|
| Historical-turnout baseline | Tests whether a model can beat simple turnout persistence |
| Linear Regression | Provides an interpretable linear benchmark |
| Decision Tree Regression | Allows nonlinear thresholds and interactions |
| Random Forest Regression | Tests whether an ensemble of nonlinear trees improves generalization |

The nonlinear model set was deliberately kept small. The aim is not to search across many algorithms for the highest score, but to test whether additional nonlinear flexibility provides useful forecasting value relative to linear regression and the historical-turnout baseline.

## Temporal validation

A random train-test split is not used because the intended task is forecasting a **future election**.

Instead, models are evaluated using out-of-time election tests:

| Forecast exercise | Training data | Out-of-time test data |
|---|---|---|
| Predict 2018 | 2014 election | 2018 election |
| Predict 2022 | 2014 + 2018 elections | 2022 election |
| Predict 2026 | 2014 + 2018 + 2022 elections | Future observed 2026 results |

Each election contributes observations for all 290 municipalities.

Performance is evaluated using:

- Mean Absolute Error (MAE)
- Root Mean Squared Error (RMSE)
- R²

This design tests whether relationships learned from earlier elections generalize to a later election.

## Historical forecasting performance

### 2018 out-of-time test

| Model | MAE | RMSE | R² |
|---|---:|---:|---:|
| Historical baseline | 1.339 | 1.473 | 0.664 |
| Linear Regression | 0.593 | **0.785** | **0.904** |
| Decision Tree | 0.710 | 0.999 | 0.845 |
| Random Forest | **0.583** | 0.859 | 0.886 |

Linear Regression and Random Forest substantially improved on the historical-turnout baseline. Random Forest achieved the lowest MAE, while Linear Regression achieved the lowest RMSE and highest R².

### 2022 out-of-time test

| Model | MAE | RMSE | R² |
|---|---:|---:|---:|
| **Historical baseline** | **2.340** | **2.589** | **0.299** |
| Linear Regression | 3.564 | 3.718 | -0.446 |
| Decision Tree | 3.554 | 3.764 | -0.482 |
| Random Forest | 3.491 | 3.690 | -0.425 |

The historical-turnout baseline performed best in 2022.

The fitted models systematically overpredicted turnout, showing that relationships that perform well in one election may fail when an election-wide shift occurs. The 2022 test therefore provides an important example of temporal distribution shift and calibration failure.

Because only two historical out-of-time elections are available, these results should not be interpreted as establishing universal superiority of one model class.

## Statistical analysis

Regression analysis shows strong persistence in municipal voter turnout across elections.

- A simple model using previous-election turnout explains approximately **56%** of the observed variation in municipal turnout.
- Adding election-year effects increases R² to approximately **0.896**, highlighting the importance of election-wide changes.
- The full regression model, including socioeconomic and demographic predictors, has an R² of approximately **0.918**.
- Municipality-clustered standard errors are used because each municipality is observed repeatedly across elections.
- A joint Wald test indicates that the socioeconomic and demographic predictors are collectively associated with turnout after controlling for previous turnout and election-year effects.

The analysis is observational and municipality-level. These associations should **not** be interpreted as causal effects or as statements about individual voting behaviour.

## Prospective 2026 forecast

The final forecasting models are trained on all **870 historical municipality-election observations** from 2014, 2018, and 2022.

The 2026 forecasting dataset uses:

- 2022 voter turnout as previous turnout
- 2024 socioeconomic and demographic characteristics

No observed 2026 turnout information is used to generate the forecasts.

| Model | Mean predicted turnout | Change vs 2022 municipal mean |
|---|---:|---:|
| Historical baseline | 84.840% | 0.000 pp |
| Linear Regression | 85.926% | +1.086 pp |
| Decision Tree | 86.032% | +1.192 pp |
| Random Forest | 86.052% | +1.212 pp |

These are **unweighted municipality-level means** and should not be interpreted as forecasts of Sweden's population-weighted national turnout.

## Frozen forecast

The prospective municipality-level 2026 predictions were frozen on **9 September 2026**:

`results/turnout_predictions_2026_frozen_2026-09-09.csv`

The predictive-modeling notebook contains overwrite protection: if the frozen file already exists, newly generated predictions are checked against it rather than silently replacing it.

The frozen forecast should remain unchanged after observed 2026 election results become available. Any later model revisions should be stored separately as post-election or exploratory analyses.

## Repository structure

```text
swedish-voter-turnout-forecast/
│
├── data/
│   ├── raw/
│   │   ├── ME0104B8_20260906-003101.csv
│   │   ├── income_data.csv
│   │   ├── scb_unemployment_2012_2020.csv
│   │   ├── scb_unemployment_2024.csv
│   │   ├── scb_education_2012_2024.csv
│   │   ├── scb_average_age_2012_2024.csv
│   │   └── scb_population_density_2012_2024.csv
│   │
│   └── processed/
│       ├── master_model_dataset_2014_2022.csv
│       └── forecast_features_2026.csv
│
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_exploratory_data_analysis.ipynb
│   ├── 03_statistical_analysis.ipynb
│   └── 04_predictive_modeling.ipynb
│
├── results/
│   ├── historical_model_validation.csv
│   ├── forecast_summary_2026.csv
│   └── turnout_predictions_2026_frozen_2026-09-09.csv
│
├── README.md
├── requirements.txt
└── .gitignore
```

## Notebook workflow

Run the notebooks in numerical order:

1. `01_data_preparation.ipynb` — builds the historical modelling and 2026 forecasting datasets
2. `02_exploratory_data_analysis.ipynb` — descriptive analysis, correlations, distributions, and multicollinearity checks
3. `03_statistical_analysis.ipynb` — regression models, clustered standard errors, and joint hypothesis testing
4. `04_predictive_modeling.ipynb` — temporal validation, final model training, and the frozen 2026 forecast

## Reproducing the analysis

Create and activate a Python virtual environment, install the project dependencies, and open Jupyter or the notebooks in VS Code.

```bash
python -m venv .venv
```

On Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Then run the notebooks in numerical order from the `notebooks/` directory.

## Limitations

- Only three historical target elections are available for model development.
- Historical predictive performance can be evaluated on only two later elections: 2018 and 2022.
- Election-wide political or social shocks are not directly represented by the municipality-level predictors.
- The study is ecological: municipality-level associations cannot be interpreted as individual-level voting behaviour.
- The analysis is predictive and observational rather than causal.
- The 2026 figures shown here are municipality-level forecasts, not national population-weighted turnout forecasts.

## Post-election evaluation

After observed 2026 municipality-level turnout becomes available, the frozen predictions will be evaluated using the same MAE, RMSE, and R² metrics used for the historical out-of-time tests.

The frozen September 2026 forecast will remain unchanged so that the prospective evaluation can distinguish genuine pre-election forecasting performance from post-election model revisions.
