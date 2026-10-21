---
tags: [econometrics, statistics, time-series, forecasting, mse, mae, mape]
---

# Forecast Evaluation Metrics

A model that fits the sample well is not necessarily a good forecaster. The honest test is **out-of-sample**: hold part of the series back, forecast it, and compare with what actually happened. This is the last step of [[ARIMA and the Box-Jenkins Approach]].

## Train and test split

Time series have an order, so the split cannot be random: the **training** set is the earlier part and the **test** set the later part. Estimate (and select) the model using training data only, forecast the test horizon, and compute the errors. With very short samples, a **rolling-origin** evaluation (re-estimate and forecast repeatedly as the origin moves forward) uses the data better than a single split.

The forecast error at time $t$ is

$$e_t = Y_t - \hat{Y}_t \quad \text{(observed minus predicted)}$$

## The three standard metrics

| Metric | Formula | Unit | Notes |
|---|---|---|---|
| **MSE** | $\dfrac{1}{h}\sum e_t^2$ | Squared unit of the series | Penalizes large errors heavily |
| **RMSE** | $\sqrt{MSE}$ | Same as series | Interpretable version of MSE |
| **MAE** | $\dfrac{1}{h}\sum \lvert e_t \rvert$ | Same as series | Typical error, less sensitive to outliers |
| **MAPE** | $\dfrac{100}{h}\sum \left\lvert \dfrac{e_t}{Y_t} \right\rvert$ | Percent | Scale-free, easy to communicate |

Reading them together is informative: RMSE much larger than MAE means a few big misses; MAPE allows comparison across series of different scales.

### Limits of MAPE

It explodes when $Y_t$ is close to zero, is undefined at zero, and penalizes over-forecasts and under-forecasts asymmetrically. It is fine for a level series far from zero, such as total energy consumption, but not for series that cross zero (growth rates, returns).

## Compare with a benchmark

A number such as "MAPE of 1.1%" means little alone. Compare with simple benchmarks: the **naive forecast** (last observed value, which is exactly the forecast of a random walk), the seasonal naive, or a drift model. A forecasting model is only useful if it beats them. Theil's U statistic is the ratio between the model's RMSE and the naive RMSE, and values below 1 mean the model beats the naive. To test whether the difference in accuracy is statistically significant, the Diebold-Mariano test is used.

## Prediction intervals

A point forecast should come with an interval. For a random walk with innovation variance $\sigma^2$, the $h$-step forecast variance is $h\sigma^2$, so the interval widens with $\sqrt{h}$. In general, the width grows with the horizon, and a good interval should contain the observed values at roughly its nominal rate, for example 95% of the time. Under non-normal errors the bootstrap can be used (see [[Bootstrap for Time Series]]).

## Practical points

- State the **scale** of the metrics: if the model was fitted to the seasonally adjusted series, the errors are for that series; to evaluate the original series, add the seasonal factors back to the forecast and recompute (see [[Seasonal Adjustment and Decomposition]]).
- Check the **sign** of the errors: if almost all are positive (or negative), there is a bias that the average hides.
- With few test points (12 months, for example) the metrics are noisy and should be read with caution.

## Worked example

$ARIMA(0,1,0)$ fitted to 48 months of seasonally adjusted electricity consumption and used to forecast the next 12 months. Because the model is a random walk without drift, the forecast was constant at 46.73 million. Results: MSE 0.4532, RMSE 0.67, MAE 0.5402 and MAPE 1.14%. All 12 errors were positive, so the model slightly underestimated 2025 (bias equal to the MAE), and all observed values fell inside the 95% interval. The largest errors were in February (1.40) and March (1.10).

## See also

- [[ARIMA and the Box-Jenkins Approach]]
- [[Information Criteria AIC and BIC]]
- [[Residual Diagnostics for Time Series]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011, Chapter 22 (Time Series Econometrics: Forecasting).

HYNDMAN, Rob J.; KOEHLER, Anne B. Another look at measures of forecast accuracy. *International Journal of Forecasting*, 22(4), 2006.

Write by **Samuel**
