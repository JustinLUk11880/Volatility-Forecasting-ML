# volatility-forecasting-ml

Results (Out-of-sample: 2024–2026)

We benchmarked all models against a strong persistence baseline (future volatility ≈ current realized volatility).

Overall forecast accuracy (volatility units):

Baseline: MAE 0.0217, RMSE 0.0415

GBR: MAE 0.0199, RMSE 0.0344
GBR improves MAE by ~8% and RMSE by ~17%, reducing large forecast errors.

Regime analysis (top 20% “high-vol” days vs the rest):

Low/normal-vol regime (80% of days): MAE 0.01524 → 0.01452, RMSE 0.02045 → 0.01940

High-vol regime (20% of days): MAE 0.04725 → 0.04138, RMSE 0.08311 → 0.06629
Improvements are concentrated during stress regimes, where risk management value is highest.

Practical use-case: volatility targeting
We used volatility forecasts to scale exposure via a simple volatility-targeting overlay (position size ∝ target_vol / forecast_vol, clipped). This demonstrates how risk forecasts translate into portfolio sizing decisions.
