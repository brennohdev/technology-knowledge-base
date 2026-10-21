---
tags: [econometrics, statistics, time-series, seasonality, decomposition]
---

# Seasonal Adjustment and Decomposition

Before testing for unit roots or fitting an ARIMA, a monthly or quarterly series usually has to be cleaned of its seasonal pattern. A repeating calendar effect (higher consumption every March, a retail peak every December) is not part of the stochastic dynamics we want to model, and it distorts both the unit root tests (see [[Unit Roots and Spurious Regression]]) and the ACF used in [[ARIMA and the Box-Jenkins Approach]].

## The components of a time series

A series is usually split into four pieces:

- **Trend ($T_t$)**: the long-run movement, growth or decline.
- **Seasonal ($S_t$)**: a pattern that repeats with a fixed period (12 for monthly data).
- **Cycle ($C_t$)**: slower, irregular swings, usually longer than a year.
- **Irregular ($I_t$)**: what is left, ideally close to white noise (see [[Stochastic Processes and Stationarity]]).

## Additive vs multiplicative

**Additive** model: the seasonal swing has roughly the same size at every level of the series.

$$Y_t = T_t + S_t + I_t$$

**Multiplicative** model: the swing grows with the level, so it is proportional to the trend.

$$Y_t = T_t \times S_t \times I_t$$

A quick visual rule: if the seasonal peaks and troughs keep the same height as the series grows, use additive; if they fan out, use multiplicative (or take logs, which turns a multiplicative model into an additive one).

## Classical decomposition, step by step

This is what `seasonal_decompose` in Python (statsmodels) does for an additive model with period 12:

1. **Trend** by a centered moving average of length 12 (a 2x12 moving average, so the window is symmetric).
2. **Detrend**: $Y_t - \hat{T}_t$.
3. **Seasonal factors**: average the detrended values of each month across all years, then center these 12 numbers so they sum to zero.
4. **Irregular**: $\hat{I}_t = Y_t - \hat{T}_t - \hat{S}_t$.

The **seasonally adjusted series** is simply $Y_t - \hat{S}_t$ (additive) or $Y_t / \hat{S}_t$ (multiplicative). It keeps trend and irregular, and loses only the repeating calendar pattern.

## Practical points

- **Seasonal adjustment does not make a series stationary.** It removes the seasonal pattern only; a trend or a unit root is still there. In the electricity consumption example, the seasonally adjusted series still failed the ADF and DF-GLS tests in levels and needed one difference.
- **Test unit roots after adjusting.** Deterministic seasonality can bias the tests, so the usual order is: plot, adjust, test, difference if needed.
- **The moving average loses observations at the ends.** A centered window of 12 cannot be computed for the first and last 6 months, so the trend there is estimated differently or is missing.
- **Adjusting with the full sample leaks information.** If the seasonal factors are estimated using the test period, the forecast evaluation is slightly optimistic. A stricter design estimates the factors using only the training data.
- **Professional alternatives.** X-13ARIMA-SEATS (US Census Bureau, used by many statistical agencies) first cleans outliers and calendar effects with a regARIMA model and then extracts the components; STL is a flexible, robust alternative to classical decomposition.
- **Seasonal ARIMA.** Instead of adjusting first, the seasonality can be modeled directly with a seasonal ARIMA, $(p,d,q)(P,D,Q)_s$. Adjusting first is simpler and easier to explain; the seasonal ARIMA keeps everything in one model.

## Worked example

Monthly Brazilian electricity consumption, January 2021 to December 2025 (60 observations). The plot by year showed lines stacked upward (trend) with a similar shape in every year (seasonality), and the amplitude was stable, so an additive decomposition with period 12 was used. The adjusted series was smoother and kept the upward trend, which is why the next step was a unit root test.

## See also

- [[Stochastic Processes and Stationarity]]
- [[Unit Roots and Spurious Regression]]
- [[ARIMA and the Box-Jenkins Approach]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011, Chapter 22 (Time Series Econometrics: Forecasting).

HYNDMAN, Rob J.; ATHANASOPOULOS, George. *Forecasting: Principles and Practice*. 3rd ed. OTexts, 2021, chapter on time series decomposition.

Write by **Samuel**
