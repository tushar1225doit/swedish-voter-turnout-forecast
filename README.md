# Forecasting Voter Turnout Across Swedish Municipalities

### Comparing statistical and machine-learning models across Swedish Riksdag elections, including a prospective evaluation of the 2026 election

This project investigates whether pre-election socioeconomic and demographic characteristics can improve municipality-level voter-turnout forecasts beyond a simple historical-turnout baseline.

Using data from Sweden's 290 municipalities, the project compares a historical-turnout baseline with Linear Regression, Decision Tree Regression, and Random Forest Regression. Models are evaluated using later elections as out-of-time test sets rather than random train-test splits.

Prospective forecast: Municipality-level predictions for the 2026 Swedish Riksdag election were generated and frozen on **9 September 2026**, before the election. They are now evaluated against the official final municipality-level turnout results without retraining or modifying the frozen forecasts.

## Research question

**Can pre-election socioeconomic and demographic characteristics improve predictions of voter turnout across Swedish municipalities beyond a simple historical-turnout baseline, and do nonlinear machine-learning models generalize better than conventional linear regression?**

## Key findings

- Municipal voter turnout shows strong persistence across elections.
- Linear Regression and Random Forest substantially improved on the historical-turnout baseline when forecasting the 2018 election.
- The simple historical-turnout baseline performed best in the 2022 historical out-of-time test and again in the prospective 2026 evaluation.
- In 2026, the baseline achieved **MAE = 0.714**, **RMSE = 0.966**, and **R² = 0.892**, outperforming Linear Regression, Decision Tree, and Random Forest at the municipality level.
- The learned models systematically overpredicted 2026 turnout, with mean bias of approximately **+1.2 to +1.4 percentage points**, compared with **+0.144** for the baseline.
- The learned models improved on the baseline in only about **18–20% of municipalities** in 2026.
- Model performance varied substantially across election periods, providing no consistent evidence that the socioeconomic and demographic predictors used here improve future municipality-level turnout forecasts beyond a strong persistence baseline.
- Greater nonlinear flexibility did not consistently improve temporal generalization.

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
| `Val_2026_slutlig_00_RD.zip` | Official final 2026 Riksdag results from Valmyndigheten | 2026 municipality turnout, eligible voters, total votes |

The historical voter-turnout data correspond to Statistics Sweden (SCB) matrix **ME0104B8**, *Voting rates in elections to the Riksdag, County Councils and Municipal Councils by region*. The original raw extracts are retained in the repository to support reproducibility.

### Processed datasets

- `data/processed/master_model_dataset_2014_2022.csv`  
  Historical modelling dataset with **870 rows × 12 columns**.

- `data/processed/forecast_features_2026.csv`  
  Prospective 2026 forecasting dataset with **290 rows × 11 columns**.

- `data/processed/actual_turnout_2026_municipalities.csv`  
  Clean municipality-level 2026 turnout results extracted from the official final Valmyndigheten results, with **290 municipalities** and no missing or duplicate municipality codes.

Historical data preparation is performed in `notebooks/01_data_preparation.ipynb`. The official 2026 result extraction and post-election checks are performed in `notebooks/05_post_election_evaluation.ipynb`.

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
| Predict 2026 | 2014 + 2018 + 2022 elections | Observed 2026 results (prospective test) |

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

Because only two historical out-of-time elections are available before the prospective 2026 test, these historical results should not be interpreted as establishing universal superiority of one model class.

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

The frozen forecast remains unchanged after the 2026 election. Post-election evaluation uses the original frozen predictions exactly as generated before election day; any later model revisions should be stored separately as post-election or exploratory analyses.

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
│   │   ├── scb_population_density_2012_2024.csv
│   │   └── Val_2026_slutlig_00_RD.zip
│   │
│   └── processed/
│       ├── master_model_dataset_2014_2022.csv
│       ├── forecast_features_2026.csv
│       └── actual_turnout_2026_municipalities.csv
│
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_exploratory_data_analysis.ipynb
│   ├── 03_statistical_analysis.ipynb
│   ├── 04_predictive_modeling.ipynb
│   └── 05_post_election_evaluation.ipynb
│
├── results/
│   ├── historical_model_validation.csv
│   ├── forecast_summary_2026.csv
│   ├── turnout_predictions_2026_frozen_2026-09-09.csv
│   ├── post_election_evaluation_2026.csv
│   ├── post_election_metrics_2026.csv
│   ├── weighted_aggregate_comparison_2026.csv
│   ├── model_comparison_2018_2026.csv
│   └── direction_diagnostics_2026.csv
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
5. `05_post_election_evaluation.ipynb` — extracts official 2026 results, evaluates the frozen forecasts, compares 2018/2022/2026 performance, and performs post-election diagnostics

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
- Model generalization is assessed on two historical out-of-time elections (2018 and 2022) plus one genuinely prospective election (2026), so the number of independent election periods remains small.
- Election-wide political or social shocks are not directly represented by the municipality-level predictors.
- The study is ecological: municipality-level associations cannot be interpreted as individual-level voting behaviour.
- The analysis is predictive and observational rather than causal.
- The frozen 2026 forecast consists of municipality-level predictions. The retrospective weighted national aggregates use final 2026 eligible-voter counts and therefore should **not** be interpreted as national forecasts that were frozen before election day.
- Poor 2022 and 2026 generalization is consistent with temporal distribution shift and calibration problems, but these results alone do not establish classical model overfitting.

## Post-election evaluation

The frozen municipality-level predictions from **9 September 2026** were evaluated against the official final 2026 Riksdag municipality results without retraining or modifying the forecasts.

### 2026 prospective municipality-level performance

| Model | MAE | RMSE | R² | Bias |
|---|---:|---:|---:|---:|
| **Historical baseline** | **0.714** | **0.966** | **0.892** | **+0.144** |
| Linear Regression | 1.512 | 1.858 | 0.602 | +1.231 |
| Decision Tree | 1.535 | 1.838 | 0.611 | +1.336 |
| Random Forest | 1.550 | 1.777 | 0.636 | +1.356 |

The historical-turnout baseline clearly performed best at the municipality level in 2026. The learned models also showed substantial positive bias, overpredicting turnout by roughly **1.2–1.4 percentage points on average**.

### Performance across elections

The best model by MAE changed across election periods:

| Election | Best model by MAE | MAE |
|---:|---|---:|
| 2018 | Random Forest | 0.583 |
| 2022 | Historical baseline | 2.340 |
| 2026 | Historical baseline | 0.714 |

The learned models performed strongly in 2018 but did not maintain that advantage in 2022 or in the prospective 2026 test. This provides no consistent evidence that the socioeconomic and demographic predictors used in this project improve future municipality-level turnout forecasts beyond turnout persistence.

### 2026 diagnostic findings

The learned models adjusted the persistence baseline upward for most municipalities:

| Model | Upward adjustment | Correct direction* | Better than baseline |
|---|---:|---:|---:|
| Linear Regression | 90.7% | 35.3% | 20.3% |
| Decision Tree | 93.4% | 37.4% | 18.6% |
| Random Forest | 96.2% | 35.3% | 17.9% |

\*Direction is evaluated among the 289 municipalities whose official turnout changed relative to the previous election.

Official turnout decreased in **190 of those 289 municipalities**, while the learned models overwhelmingly pushed predictions upward. Their adjustments therefore often moved in the wrong direction and improved on the frozen baseline in only about one fifth of municipalities.

### Retrospective weighted aggregation

The voter-weighted actual turnout reconstructed from municipality-level vote and eligible-voter totals was approximately **84.885%**.

Using final 2026 eligible-voter counts to retrospectively weight the frozen municipality predictions gives:

| Model | Weighted prediction | Aggregate error |
|---|---:|---:|
| Historical baseline | 84.188% | -0.697 pp |
| Linear Regression | 85.078% | +0.193 pp |
| Decision Tree | 85.349% | +0.464 pp |
| Random Forest | 85.264% | +0.379 pp |

Linear Regression happened to be closest to the overall weighted turnout after aggregation, even though the historical baseline was much more accurate across individual municipalities. This is possible because positive and negative municipality errors can cancel when aggregated.

These weighted values are **retrospective aggregates of the frozen municipality forecasts**, not pre-election national forecasts, because the final 2026 eligible-voter counts used as weights were incorporated after the election.

### Overall interpretation

The prospective 2026 evaluation strengthens the central conclusion of the project: statistical association does not automatically translate into stable future forecasting value. Socioeconomic and demographic variables were associated with municipal turnout in the historical regression analysis, but the learned forecasting models did not consistently outperform a strong persistence baseline across elections.

The 2018, 2022, and 2026 results together point to temporal instability and calibration challenges rather than a universally superior model class. Greater nonlinear flexibility did not consistently improve generalization, and the small number of independent election periods means these findings should be interpreted cautiously.
