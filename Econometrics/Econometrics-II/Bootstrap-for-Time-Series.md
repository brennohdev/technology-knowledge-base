---
tags: [econometrics, statistics, time-series, bootstrap, resampling, forecasting]
---

# Bootstrap for Time Series

The bootstrap estimates the sampling distribution of a statistic by **resampling the data itself**, without assuming a formula for the distribution. In time series it needs extra care, because observations are **dependent**, and the classic recipe was built for independent data.

## Why the plain bootstrap fails here

The classic (iid) bootstrap draws observations with replacement. Applied directly to a series, the draws are shuffled, which **destroys the autocorrelation, the trend and the seasonality**. The resampled series has the same marginal distribution but none of the dynamics, so any statistic that depends on dependence (the variance of a mean, an ARIMA coefficient, a forecast interval) is estimated wrongly. It should be used only on things that are really independent, such as the residuals of a well-specified model.

## Methods that respect dependence

| Method | Idea | When it fits |
|---|---|---|
| **Residual (model-based) bootstrap** | Fit the model, resample its residuals, rebuild a new series recursively with the estimated parameters | The model is correctly specified and the residuals are white noise |
| **Moving block bootstrap** | Resample overlapping blocks of $l$ consecutive observations and glue them together | Weakly dependent stationary series, no parametric model needed |
| **Stationary bootstrap** | Blocks with random (geometric) length, which keeps the resampled series stationary | Same as above, less sensitive to the block length |
| **Sieve bootstrap** | Approximate the process by an AR of growing order, then bootstrap its residuals | Linear processes with unknown order |

Block methods preserve the dependence **inside** each block but not across blocks, so the block length $l$ matters: too short loses dependence, too long leaves few distinct blocks. A common rule is to let $l$ grow slowly with $n$, for example of order $n^{1/3}$.

## Uses in time series econometrics

- **Forecast intervals without normality.** Resample the ARIMA residuals to simulate many future paths and take percentiles at each horizon. This is useful when the Jarque-Bera test rejects normality (see [[Residual Diagnostics for Time Series]]).
- **Standard errors and confidence intervals** for estimated parameters in small samples.
- **Better critical values for unit root tests**, since the asymptotic distributions can be poor with small $T$ (see [[Unit Root Tests Beyond ADF]]).
- **Bagging:** forecasts averaged over many bootstrap series, which can reduce the variance of the forecast.

## Practical advice

1. **The series must be (made) stationary** before block resampling, or the model must account for the trend, as in an ARIMA with differencing. Bootstrapping a trending series in blocks mixes levels from different periods.
2. **Check that the resampled data look like the original**: plot a few bootstrap series and compare their ACF with the real one.
3. **Choose the method from the situation**: a series with strong dependence and no clear model calls for blocks; a series with a good ARIMA calls for residual resampling.
4. **Do not use the bootstrap as a substitute for a correct model.** It only reproduces the uncertainty the model assumes.

## Worked example

For the $ARIMA(0,1,0)$ of seasonally adjusted electricity consumption, the residuals passed Ljung-Box, so resampling them is legitimate. Rebuilding 12-month paths from the last observed value by adding randomly drawn residuals, and taking the 2.5% and 97.5% percentiles at each month, gives an interval comparable to the analytical one, whose half-width grows as $1.96\hat{\sigma}\sqrt{h}$.

## See also

- [[Stochastic Processes and Stationarity]]
- [[ARIMA and the Box-Jenkins Approach]]
- [[Residual Diagnostics for Time Series]]
- [[Unit Root Tests Beyond ADF]]
- [[Econometrics]]

## Reference

KÜNSCH, H. R. The jackknife and the bootstrap for general stationary observations. *Annals of Statistics*, 17(3), 1989.

POLITIS, D.; ROMANO, J. The stationary bootstrap. *Journal of the American Statistical Association*, 89(428), 1994.

BÜHLMANN, P. Sieve bootstrap for time series. *Bernoulli*, 3(2), 1997.

Write by **Samuel**
