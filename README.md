# Volatility Forecasting with Machine Learning

Forecasting realized volatility is a core problem in quantitative finance and risk management. This project builds an end-to-end time-series pipeline to predict **future realized volatility** using engineered features and supervised learning, benchmarked against strong classical baselines.

## Project Goals
- Forecast **future realized volatility** (risk) rather than directional returns.
- Compare ML models against a **persistence baseline** (future vol ≈ current vol).
- Evaluate performance **overall** and **by volatility regime** (stress vs calm).
- Demonstrate a practical use-case: **volatility targeting** (risk-based sizing).

---

## Method Summary

### Target Definition
- Compute daily log returns:  
  \
  r_t = \log(P_t) - \log(P_{t-1})
  \
- Compute **20-day realized volatility (annualized)**:
  \
  \sigma_t = \sqrt{252}\cdot \text{std}(r_{t-19:t})
  \
- Forecast target (regression):
  \
  y_t = \sigma_{t+h}
  \
  where `h = 5` trading days (1 week ahead).

### Feature Engineering (examples)
Features are constructed using information available at time *t* only:
- volatility persistence: lagged realized vol (`rv_lag1`, `rv_lag5`, `rv_lag20`)
- volatility dynamics: changes in vol (`rv_change1`, `rv_change5`)
- return shock proxies: abs returns and rolling averages (`abs_ret_mean5`, `abs_ret_mean20`)
- regime proxies: vol-of-vol, drawdown, trend (`vol_of_vol_20`, `drawdown_60`, `ma_ratio_20_60`)

### Models
- **Baseline (persistence):** predict \(\hat{\sigma}_{t+h} = \sigma_t\)
- **Ridge regression:** linear model with L2 regularization
- **Gradient Boosting (GBR):** nonlinear model capturing interactions and regime behavior

### Evaluation
Out-of-sample evaluation with a **time-based split** (no shuffling).
Metrics:
- MAE and RMSE in volatility units (annualized)

---

## Results (Out-of-sample: 2024–2026)

### Overall Forecast Accuracy (volatility units)
| model | MAE | RMSE |
|---|---:|---:|
| baseline (persistence) | 0.02168 | 0.04152 |
| gradient boosting (GBR) | **0.01993** | **0.03442** |

GBR improves over the strong persistence baseline (~8% lower MAE and ~17% lower RMSE), reducing large forecasting errors.

### Regime Analysis (High-vol vs Normal)
High-vol regime defined as the top 20% of days by realized future volatility.

| regime | n_days | baseline_MAE | gbr_MAE | baseline_RMSE | gbr_RMSE |
|---|---:|---:|---:|---:|---:|
| normal (bottom 80%) | 397 | 0.01524 | **0.01452** | 0.02045 | **0.01940** |
| high-vol (top 20%) | 100 | 0.04725 | **0.04138** | 0.08311 | **0.06629** |

The ML model adds the most value during **high-volatility regimes**, where risk forecasting matters most (≈12% MAE and ≈20% RMSE improvement).

---

## Practical Use-case: Volatility Targeting (Risk Overlay)
To show how forecasts translate into decisions, we implement a volatility-targeting overlay:
\
w_t = \text{clip}\left(\frac{\sigma_{\text{target}}}{\hat{\sigma}_t}, 0, w_{\max}\right)
\
and apply it to asset returns with a 1-day lag to avoid lookahead bias.

> Note: If applying sizing to a different asset than the one used for training (e.g., TSLA), interpret results as an **illustrative risk overlay**. For a fully consistent demo, train and apply on the same asset.

---

## Repository Structure
```text
volatility-forecasting-ml/
├── data_raw/                 # optional raw downloads / cached files
├── data_processed/           # engineered datasets and predictions
├── notebooks/
│   ├── 01_exploratory_data_and_problem_definition.ipynb
│   ├── 02_feature_engineering_volatility.ipynb
│   ├── 03_label_creation.ipynb
│   ├── 04_model_training.ipynb
│   └── 05_backtesting_and_evaluation.ipynb
├── src/                      # (optional) reusable functions/modules
├── requirements.txt
└── README.md
```

---

## How to Run

### 1) Create environment
```bash
python -m venv venv
source venv/bin/activate  # macOS/Linux
pip install -r requirements.txt
```

### 2) Run notebooks in order
1. `01_exploratory_data_and_problem_definition.ipynb`
2. `02_feature_engineering_volatility.ipynb`
3. `03_label_creation.ipynb`
4. `04_model_training.ipynb`
5. `05_backtesting_and_evaluation.ipynb`

Artifacts saved to `data_processed/`:
- `volatility_base.csv`
- `volatility_features.csv`
- `train_regression.csv`, `test_regression.csv`
- `predictions_regression.csv`

---

## Notes / Limitations
- This project focuses on **forecast accuracy and methodology**, not trading alpha.
- Transaction costs and execution constraints are not modeled in the overlay (can be added as an extension).
- Forecasting extreme spikes remains challenging due to rarity and rolling-window target dynamics.

---

## Extensions (Future Work)
- Train/apply models across multiple assets and compare generalization.
- Add transaction costs / turnover constraints to vol targeting.
- Predict quantiles (e.g., 90th percentile volatility) for stress testing and VaR.
- Compare to GARCH-family baselines.

---

## Disclaimer
This project is for educational and research purposes only and does not constitute financial advice.
