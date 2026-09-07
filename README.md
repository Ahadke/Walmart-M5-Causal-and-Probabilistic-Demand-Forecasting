# Walmart M5 Enterprise-Scale Causal & Probabilistic Demand Forecasting System

An end-to-end retail demand forecasting system built on the **Walmart M5 dataset**, combining causal feature engineering, probabilistic forecasting, hierarchical reconciliation, inventory optimization, and production-style drift monitoring.

The system moves beyond point forecasting by quantifying demand uncertainty and translating model outputs into actionable inventory decisions.

---

## Project Overview

Retail demand is influenced by more than historical sales. Prices, promotions, holidays, events, seasonality, and changing customer behavior can all affect future demand.

This project addresses the business problem:

> **How can historical sales, pricing, promotions, events, and calendar effects be transformed into reliable demand forecasts and inventory decisions under uncertainty?**

The pipeline combines:

- Causal and time-series feature engineering
- Probabilistic demand forecasting
- Quantile-based uncertainty estimation
- Hierarchical forecast reconciliation
- Inventory policy optimization
- Concept-drift detection and monitoring

The result is an end-to-end analytical workflow connecting **raw retail data → statistical modeling → uncertainty quantification → inventory decisions → model monitoring**.

---

## Model Performance

The probabilistic forecasting models were evaluated on held-out data using both point-forecast error and prediction-interval coverage.

| Metric | Result |
|---|---:|
| **P50 Mean Absolute Error (MAE)** | **1.07 units** |
| **P10–P90 Empirical Coverage** | **87.24%** |
| Forecast Quantiles | P10, P50, P90 |
| Quantile Models | 3 |
| Estimators per Model | 200 |
| Drift Monitoring Window | 14 days |
| Drift Detection Threshold | Rolling Mean + 2σ |

The median **P50 forecast achieved an MAE of approximately 1.07 units**, providing an interpretable measure of point-forecast error.

The **P10–P90 prediction interval captured approximately 87.24% of observed demand**, demonstrating the model's ability to quantify forecast uncertainty for downstream inventory planning.

Rather than relying solely on a single demand estimate, the system generates lower, median, and upper demand scenarios that support risk-aware inventory decisions.

---

## Dataset

The project uses the **Walmart M5 Forecasting dataset**, which contains hierarchical daily retail sales across Walmart stores in **California, Texas, and Wisconsin**.

The data includes:

- Historical daily unit sales
- Product and item identifiers
- Department and category information
- Store and state hierarchy
- Product prices
- Calendar information
- Events and holidays
- SNAP indicators

The retail hierarchy can be represented as:

```text
State
 └── Store
      └── Category
           └── Department
                └── Item / SKU
```

This structure enables forecasting and evaluation at multiple levels of business aggregation.

---

## End-to-End Pipeline

```text
Walmart M5 Raw Data
        │
        ▼
Data Validation & EDA
        │
        ▼
Hierarchical Structure
        │
        ▼
Causal & Time-Series Feature Engineering
        │
        ▼
Probabilistic Forecasting
     P10 / P50 / P90
        │
        ▼
Model Evaluation
  MAE + Interval Coverage
        │
        ▼
Hierarchical Reconciliation
        │
        ▼
Inventory Optimization
        │
        ▼
Safety Stock & Reorder Policies
        │
        ▼
Drift Detection & Monitoring
```

---

## 1. Data Understanding & Exploratory Analysis

The first stage analyzes the structure and quality of the Walmart M5 data before modeling.

Exploratory analysis covers:

- Demand distributions
- Missing values
- Product-level demand patterns
- Store-level sales behavior
- State-level demand differences
- Price variation
- Calendar seasonality
- Event and holiday effects

The goal is to understand both the statistical characteristics of demand and the business structure represented in the dataset.

---

## 2. Hierarchical Demand Structure

Retail demand exists simultaneously across multiple aggregation levels.

The pipeline constructs relationships between:

```text
Item → Department → Category
Item → Store → State
```

This hierarchy is important because independently generated forecasts can become mathematically inconsistent.

For example:

```text
Forecast(Store A) + Forecast(Store B) ≠ Forecast(State Total)
```

The hierarchy is therefore explicitly modeled so forecasts can later be reconciled across aggregation levels.

---

## 3. Causal & Time-Series Feature Engineering

Historical sales alone may not capture the factors associated with changing demand.

The pipeline engineers features from several potential demand drivers, including:

- Historical demand lags
- Rolling demand statistics
- Price changes
- Promotions
- Events
- Holidays
- Calendar variables
- Seasonal patterns

These variables allow the forecasting system to incorporate contextual information associated with demand changes instead of relying only on previous sales.

### Important Causal Interpretation

The project treats pricing, promotions, events, and calendar variables as **potential causal drivers and causal features**.

Because the Walmart M5 dataset is observational, relationships between these variables and demand are not automatically interpreted as experimentally identified causal effects.

This distinction avoids conflating predictive relationships with formal causal inference.

---

## 4. Probabilistic Demand Forecasting

Traditional forecasting models often produce a single point estimate.

This project instead trains **quantile regression models** to estimate multiple points in the conditional demand distribution.

The forecasting system generates:

```text
P10 → Lower-demand scenario
P50 → Median expected demand
P90 → Higher-demand scenario
```

The models are implemented using **scikit-learn GradientBoostingRegressor with quantile loss**.

Each quantile model uses:

```text
Estimators:     200
Learning Rate:  0.05
Maximum Depth:  3
Loss:           Quantile
```

Three independent models estimate the P10, P50, and P90 demand quantiles.

---

## 5. Forecast Evaluation

The forecasting system is evaluated from two complementary perspectives.

### Point-Forecast Accuracy

The median P50 forecast is evaluated using **Mean Absolute Error (MAE)**.

```text
P50 MAE ≈ 1.07 units
```

This means the median demand forecast differs from observed demand by approximately **1.07 units on average** in the evaluated data.

### Probabilistic Calibration

The uncertainty estimates are evaluated using empirical prediction-interval coverage.

```text
P10–P90 Empirical Coverage ≈ 87.24%
```

Approximately **87.24% of observed demand values fall within the model's P10–P90 prediction interval**.

This evaluation is particularly important for inventory decisions because the system must estimate not only expected demand but also the uncertainty surrounding that estimate.

---

## 6. Hierarchical Forecast Reconciliation

Forecasts generated independently at different levels of a retail hierarchy may not agree with one another.

The reconciliation stage enforces consistency across:

- SKU / item
- Store
- State

Conceptually:

```text
Individual Forecasts
        │
        ▼
Hierarchy Mapping
        │
        ▼
Forecast Reconciliation
        │
        ▼
Coherent Retail Forecasts
```

The resulting predictions can therefore be interpreted consistently across different levels of the organization.

---

## 7. Inventory Optimization

Forecasts become more valuable when they are connected to operational decisions.

The system converts probabilistic demand estimates into inventory planning signals.

The inventory workflow uses forecast uncertainty to support:

- Safety-stock estimation
- Reorder-point calculations
- Demand-risk assessment
- Service-level-aware inventory decisions

Conceptually:

```text
P10 / P50 / P90 Forecasts
           │
           ▼
   Demand Uncertainty
           │
           ▼
      Safety Stock
           │
           ▼
     Reorder Point
           │
           ▼
    Inventory Policy
```

Using probabilistic forecasts allows inventory decisions to account for uncertainty rather than assuming future demand is known precisely.

---

## 8. Drift Detection & Model Monitoring

Forecasting models can degrade when demand patterns change over time.

Changes may arise from:

- Customer behavior
- Pricing
- Promotions
- Seasonality
- Product demand
- External events

The monitoring pipeline tracks forecast errors at the **SKU-date level**.

For each SKU, the system calculates:

- Absolute forecast error
- **14-day rolling mean error**
- **14-day rolling error standard deviation**

Potential drift is flagged when:

```text
Current Absolute Error
>
Rolling Mean Error + 2 × Rolling Error Standard Deviation
```

This creates a **dynamic 2σ threshold** based on recent model behavior.

SKU-level drift indicators are then aggregated into a daily drift rate, providing a system-level view of changes in forecasting performance.

The monitoring layer can therefore provide a signal for further investigation or model retraining.

---

## Business Impact

The project connects statistical modeling directly to operational retail decisions.

### What will demand look like?

**Probabilistic forecasting** estimates future demand using P10, P50, and P90 scenarios.

### How uncertain is the forecast?

**Prediction intervals** quantify the range of plausible demand outcomes, achieving **87.24% empirical P10–P90 coverage** in the evaluated data.

### How accurate is expected demand?

The median forecast achieves approximately **1.07 units MAE**.

### How much inventory should be held?

Forecast distributions are translated into **safety-stock and reorder-point recommendations**.

### When should the model be investigated?

A **14-day rolling error monitor with a 2σ threshold** detects unusual changes in forecast behavior.

Together, these components create a closed-loop decision framework:

```text
Data
  ↓
Feature Engineering
  ↓
Probabilistic Forecasting
  ↓
Model Evaluation
  ↓
Hierarchical Reconciliation
  ↓
Inventory Decisions
  ↓
Drift Monitoring
```

---

## Key Technical Contributions

- Built an end-to-end forecasting workflow on hierarchical Walmart M5 retail data.
- Engineered price, promotion, event, calendar, lag, and seasonal demand features.
- Developed **P10/P50/P90 probabilistic quantile forecasting models**.
- Achieved **1.07-unit P50 MAE** on held-out forecast evaluation.
- Achieved **87.24% empirical P10–P90 prediction-interval coverage**.
- Reconciled forecasts across **SKU, store, and state** hierarchy levels.
- Translated probabilistic forecasts into inventory planning policies.
- Implemented **14-day rolling model monitoring with a dynamic 2σ drift threshold**.

---

## Project Structure

```text
Enterprise-Scale-Causal-Probabilistic-Demand-Forecasting-System/
│
├── data/
│   ├── raw/
│   │   ├── calendar.csv
│   │   ├── sales_train_validation.csv
│   │   └── sell_prices.csv
│   │
│   └── processed/
│
├── notebooks/
│   ├── 01_data_understanding_and_eda.ipynb
│   ├── 02_hierarchical_structure.ipynb
│   ├── 03_causal_feature_engineering.ipynb
│   ├── 04_probabilistic_forecasting.ipynb
│   ├── 05_hierarchical_reconciliation.ipynb
│   ├── 06_inventory_optimization.ipynb
│   └── 07_drift_detection_and_monitoring.ipynb
│
├── outputs/
│   ├── forecasts/
│   ├── inventory/
│   └── monitoring/
│
├── requirements.txt
└── README.md
```

---

## Notebook Workflow

Run the notebooks sequentially:

### `01_data_understanding_and_eda.ipynb`

Explores sales distributions, prices, calendar variables, hierarchy structure, and temporal demand patterns.

### `02_hierarchical_structure.ipynb`

Constructs product and geographic demand hierarchies required for downstream reconciliation.

### `03_causal_feature_engineering.ipynb`

Creates lag, rolling, pricing, promotion, event, calendar, and seasonal features for demand modeling.

### `04_probabilistic_forecasting.ipynb`

Trains P10, P50, and P90 quantile regression models and evaluates forecast accuracy and uncertainty calibration.

### `05_hierarchical_reconciliation.ipynb`

Reconciles independently generated forecasts across SKU, store, and state levels.

### `06_inventory_optimization.ipynb`

Transforms probabilistic demand forecasts into safety-stock and reorder-point recommendations.

### `07_drift_detection_and_monitoring.ipynb`

Tracks rolling forecast errors and identifies potential concept drift using dynamic statistical thresholds.

---

## Technologies

### Data Science

- Python
- pandas
- NumPy
- scikit-learn
- SciPy

### Statistical & Machine Learning Methods

- Quantile Regression
- Gradient Boosting
- Probabilistic Forecasting
- Time-Series Feature Engineering
- Prediction-Interval Evaluation
- Hierarchical Forecast Reconciliation
- Statistical Drift Detection

### Visualization & Development

- Matplotlib
- Jupyter Notebook
- Git / GitHub

---

## Model Configuration

The probabilistic forecasting stage trains separate gradient-boosting models for each demand quantile.

| Parameter | Value |
|---|---:|
| Model | GradientBoostingRegressor |
| Loss | Quantile |
| Quantiles | 0.10, 0.50, 0.90 |
| Number of Models | 3 |
| Estimators | 200 |
| Learning Rate | 0.05 |
| Maximum Depth | 3 |

This approach directly estimates conditional demand quantiles without assuming a fixed parametric distribution for forecast errors.

---

## Outputs

### Forecasting Outputs

```text
P10 demand forecasts
P50 median demand forecasts
P90 demand forecasts
Prediction intervals
```

### Hierarchical Outputs

```text
SKU-level forecasts
Store-level forecasts
State-level forecasts
Reconciled forecasts
```

### Inventory Outputs

```text
Safety-stock recommendations
Reorder-point recommendations
Risk-aware inventory policies
```

### Monitoring Outputs

```text
SKU-level forecast errors
14-day rolling error statistics
2σ drift thresholds
SKU-level drift flags
Daily aggregate drift rates
```

---

## Reproducibility

The project is organized as a sequential notebook pipeline so that each analytical stage builds on outputs from the previous stage.

To reproduce the workflow:

```bash
git clone https://github.com/Ahadke/Enterprise-Scale-Causal-Probabilistic-Demand-Forecasting-System.git

cd Enterprise-Scale-Causal-Probabilistic-Demand-Forecasting-System

pip install -r requirements.txt
```

Then execute notebooks `01` through `07` sequentially.

---

## Future Improvements

Potential extensions include:

- Automated retraining after sustained drift
- Rolling-origin time-series cross-validation
- Additional probabilistic calibration metrics
- Hyperparameter optimization
- Distributed training for larger retail datasets
- Experiment tracking and model versioning
- Automated monitoring alerts
- Interactive demand and inventory dashboards
- Production API deployment
- Scheduled batch inference pipelines

---

## Skills Demonstrated

**Python · Scikit-learn · Statistical Modeling · Machine Learning · Time Series · Probabilistic Forecasting · Quantile Regression · Feature Engineering · Model Evaluation · Uncertainty Quantification · Hierarchical Forecasting · Inventory Optimization · Drift Detection · Data Analysis**

---

## License

This project is available for educational and research purposes.
