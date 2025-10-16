# London Housing Prices: End-to-End Analytics Pipeline (2018–2022)

This project builds an analysis-ready, monthly panel of London property transactions and borough attributes (interest rates, earnings, crime, bedroom counts). It harmonizes disparate public datasets, imputes gaps, engineers features, and ships notebooks/artefacts for EDA and price-driver modeling (fixed-effects OLS and gradient-boosted trees).

---

## Background

London’s housing market is large, heterogeneous, and sensitive to macro (rates) and local fundamentals (income, crime, stock mix). The pipeline consolidates transaction-level data (2018–2022) and augments it with borough-monthly features to quantify key drivers of prices and provide a robust base for forecasting/valuation models.

**Targets & Questions**

* What features explain most price variation across boroughs and time?
* How do structure (type, tenure, bedrooms) and local economics (income, crime) relate to prices?
* How much usable signal do macro rates contribute over this window?

---

## Machine Learning Models


* **Fixed-Effects OLS (Hedonic)** — Linear model with **borough** and **month** fixed effects to capture spatial and temporal heterogeneity; optional **log-price** target for variance stabilization; robust SEs recommended.
* **Regularized Linear Models** — **Ridge**, **Lasso**, and **ElasticNet** on one-hot encoded categoricals to handle multicollinearity and shrink high-cardinality dummies; hyperparameters tuned via **blocked time-series CV**.
* **Tree Ensembles** — **RandomForestRegressor** as a variance-reduction baseline; **Gradient Boosting** via **LightGBM/XGBoost** for nonlinearities and interaction effects; early stopping on a time-ordered validation fold.
* **CatBoost Regressor** — Native handling of categorical features (no heavy one-hot), strong default performance on mixed numeric/categorical tabular data; compares directly to fixed-effects OLS for explainability vs. accuracy trade-offs.

**Evaluation setup:** temporal split (**train 2018–2021 / test 2022**), metrics **RMSE/MAE/R²/MAPE**, blocked time-series CV for tuning; diagnostics include residual stratification by borough, property type, and bedroom count, plus error vs. price-quantile analysis.


## Data & Sources

* **UK Price Paid** (2018–2022; nationwide → filtered to London)
* **Bank of England** base rate changes → monthly **`InterestRate`** (step-function to `MM-YYYY`)
* **UK Government Earnings** (borough-level) → **`MeanSalary`**

  * 2019 **Bromley** imputed via YoY interpolation
  * **2022** extrapolated per borough with linear regression on 10-year trend
* **Metropolitan Police** crime counts → **`CrimeRate`**

  * 2010–2021 and 2021–2023 harmonized; columns re-keyed to `MM-YYYY`
  * **City of London** gap filled via *area-weighted average* of adjacent boroughs
* **Bedroom proxy** (scraped price bands by borough/year) → **`BedroomCount`** (1–5 where 5=5+)

  * Borough name normalization (e.g., *Hammersmith* → *Hammersmith and Fulham*)

**Scale:** ~**547k** London transactions (2018–2022), monthly granularity.

---

## Cleaning & Feature Engineering

* **Schema harmonization:** `Period (MM-YYYY)`; categorical remaps

  * `PropertyType`: {Detached, Semi-Detached, Terraced, Flats/Maisonettes}
  * `PropertyAgeGroup`: {New, Established}; `Tenure`: {Freehold, Leasehold}
  * `Borough`: title-case + disambiguations (e.g., *City of Westminster*)
* **Joins:** (`Borough`,`Period`) for salary/crime/bedrooms; (`Period`) for interest rate
* **Bedroom inference:** per-transaction **nearest-price match** within same borough-month bedroom grid
* **Outliers:** **1st/99th percentile winsorization** on `Price`
* **Final schema:**
  `Price, Period, Postcode, PropertyType, PropertyAgeGroup, Tenure, Borough, InterestRate, MeanSalary, CrimeRate, BedroomCount`

---

## Exploratory Data Analysis (EDA)

* Skewed price distribution; log views used for interpretability
* Borough medians show large spatial dispersion (Westminster/Kensington & Chelsea at the top end)
* Structure effects: **Detached/Semi-Detached > Terraced > Flats**; **Freehold > Leasehold**
* Time series (monthly average): upward trend across 2018–2022; higher-rate months show lower price concentrations and fewer sales
* Binned distributions by `InterestRate`, `CrimeRate`, `MeanSalary`; heatmap of Borough × PropertyType medians

---

## Modeling Approach

**Baselines (ready to plug in):**

* **Hedonic OLS** with **time fixed effects** (month) and **borough fixed effects**
* **Regularized linear:** Lasso / Ridge / ElasticNet
* **Tree ensembles:** RandomForest, **LightGBM/XGBoost**, **CatBoost** (handles categorical interactions well)

**Evaluation protocol:**

* **Temporal split:** train 2018–2021, test 2022
* **Metrics:** RMSE, MAE, **R²**, MAPE
* **CV:** **blocked time-series CV** (expanding window)
* **Diagnostics:** residuals by borough/type/bedroom; bias vs. price quantiles

---

## ✅ Outcomes & Key Findings

* **Clean, analysis-ready panel** (2018–2022, ~546,999 rows) with aligned categories and borough-monthly features per transaction
* **Top correlates with Price:** `BedroomCount` **0.585**, `MeanSalary` **0.399**
* **Weak within-window associations:** `CrimeRate` **0.131**, `InterestRate` **0.015**
* **Structural premiums:** Detached/Semi-Detached and Freehold transact at higher medians
* **Temporal signal:** rising average prices; higher rates correspond to fewer/lower transactions
* **Completeness:** City of London crime imputed; Bromley 2019 salary fixed; borough naming harmonized; bedrooms inferred per record

---

## Tools & Technologies

**Language & Env:** Python 3.x, Jupyter Notebook
**Core libs:** `pandas`, `numpy`, `scikit-learn`, `scipy` (`winsorize`), `matplotlib`, `seaborn`
**Utilities:** `re`, `IPython.display`
**(Suggested)** Modeling add-ons: `lightgbm`, `xgboost`, `catboost`, `statsmodels`

---

## ⚠️ Limitations & Assumptions

* **City of London** crime is **imputed** from neighboring boroughs (area-weighted)
* **`BedroomCount`** is derived from price bands (5 aggregates 5+) → measurement error
* **`InterestRate`** is a monthly step proxy; intra-month mortgage dynamics not captured

---

### 🔖 Keywords / Topics

`london-housing` `real-estate` `price-paid-data` `panel-data` `feature-engineering`
`hedonic-pricing` `crime-data` `interest-rates` `salary-data` `pandas` `scikit-learn`

*PRs and ideas welcome.*
