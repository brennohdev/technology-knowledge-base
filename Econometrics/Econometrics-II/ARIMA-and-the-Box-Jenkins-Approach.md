---
tags: [econometrics, statistics, time-series, arima, forecasting]
---

# ARIMA and the Box-Jenkins Approach

Once a series is confirmed stationary, or made stationary by differencing (see [[Stochastic Processes and Stationarity]] and [[Unit Roots and Spurious Regression]]), the next natural question in time series econometrics is how to model its own dynamics, how does $Y_t$ relate to its own past, so it can be forecast. That's what ARIMA models do.

## Building blocks

**Autoregressive (AR) component**, order $p$: $Y_t$ depends on its own past values.

$$Y_t = c + \phi_1 Y_{t-1} + \phi_2 Y_{t-2} + \dots + \phi_p Y_{t-p} + \varepsilon_t$$

**Moving Average (MA) component**, order $q$: $Y_t$ depends on past forecast *errors*, not past levels.

$$Y_t = c + \varepsilon_t + \theta_1 \varepsilon_{t-1} + \dots + \theta_q \varepsilon_{t-q}$$

**ARMA(p,q)** combines both, for a stationary series. **ARIMA(p,d,q)** adds the "I" for Integrated: apply $d$ differences first (from [[Unit Roots and Spurious Regression]]'s order of integration) to make the series stationary, *then* fit an ARMA(p,q) to the differenced series.

A special case worth knowing: **ARIMA(0,1,0)** has no AR and no MA terms, only one difference. It is the **random walk**, $Y_t = Y_{t-1} + \varepsilon_t$, and its forecast for any horizon is simply the last observed value.

## The Box-Jenkins methodology

The classic four-step process for picking and fitting an ARIMA model:

1. **Identification** — figure out $d$ (via unit root tests) and candidate $(p,q)$ by inspecting the **ACF** (autocorrelation function) and **PACF** (partial autocorrelation function) of the (differenced) series:
   - AR(p): PACF cuts off sharply after lag $p$; ACF decays gradually.
   - MA(q): ACF cuts off sharply after lag $q$; PACF decays gradually.
   - ARMA(p,q): both decay gradually, harder to read visually, information criteria (AIC/BIC) usually decide between candidate specifications (see [[Information Criteria AIC and BIC]]).
2. **Estimation** — fit the candidate model(s), typically by maximum likelihood.
3. **Diagnostic checking** — the residuals should look like white noise (see [[Stochastic Processes and Stationarity]]). Check with a residual ACF plot and a formal test (e.g. Ljung-Box). If residuals still show structure, the model is under-specified, go back to step 1. The full set of tests is in [[Residual Diagnostics for Time Series]].
4. **Forecasting** — once diagnostics pass, use the fitted model to forecast future values, with the estimated uncertainty widening the further out the forecast horizon goes. Accuracy is judged on held-out data (see [[Forecast Evaluation Metrics]]).

## Estimation by maximum likelihood

With MA terms, the past errors $\varepsilon_{t-1}, \dots$ are not observed, so the model cannot be estimated by a plain OLS regression. Instead, the parameters are chosen to maximize the Gaussian likelihood of the observed data. In practice the model is written in **state-space form** and the exact likelihood is computed recursively with the **Kalman filter**, which at each month predicts the next value, compares it with the observed one and updates. The one-step prediction errors (the *innovations*) are what enter the likelihood. This is what `ARIMA` in Python's statsmodels does.

For an ARIMA(0,1,0) there are no AR or MA coefficients, so the only parameter estimated is the error variance $\sigma^2$.

## A practical workflow

A compact recipe that follows the four steps above on a monthly series:

1. Plot the series, and remove the seasonal pattern if there is one (see [[Seasonal Adjustment and Decomposition]]).
2. Test for a unit root (ADF and DF-GLS, see [[Unit Root Tests Beyond ADF]]) and difference until the tests reject it, which fixes $d$.
3. Split the sample in time order into training and test sets.
4. On the training set, estimate a grid of $(p,q)$ and compare AIC and BIC, preferring BIC when the sample is small.
5. Run the residual diagnostics of the chosen model.
6. Forecast the test period and evaluate with MSE, MAE and MAPE against a naive benchmark.

## Worked example

Monthly Brazilian electricity consumption, January 2021 to December 2025. After adjusting for seasonality, ADF and DF-GLS did not reject a unit root in levels but rejected it after one difference, so $d = 1$. On 48 training months, a grid of $p,q \in \{0,\dots,3\}$ was estimated, and both AIC and BIC selected ARIMA(0,1,0), a random walk without drift. The residuals passed Ljung-Box, Breusch-Godfrey, ARCH-LM and Jarque-Bera. The 12-month forecast for 2025 was constant at the last training value, with a MAPE of about 1.1% and all observed values inside the 95% interval. A flat forecast is not a bug here, it is what a random walk implies.

## Why this matters after unit root testing

This is the direct payoff of getting stationarity right first: an ARIMA model fit *without* checking for a unit root is really just fitting noise to a non-stationary series, and its "forecasts" tend to just extrapolate whatever trend happened to be in the sample, unreliable out of sample. Confirming $I(d)$ first, then modeling the properly differenced series, is what makes the whole exercise meaningful rather than curve-fitting.

## Where this leads next

Natural follow-ups once ARIMA is solid, worth their own notes later as they come up in coursework:

- **Seasonal ARIMA (SARIMA)**, which models the seasonal pattern directly, $(p,d,q)(P,D,Q)_s$, instead of adjusting the series first.
- **Cointegration and error-correction models (ECM)** — the legitimate case where two $I(1)$ series *can* be regressed together in levels, because a linear combination of them is stationary (mentioned as a forward pointer in [[Unit Roots and Spurious Regression]]).
- **Multivariate time series (VAR)** — modeling several time series jointly, each as a function of lags of all of them, common in macroeconometrics for questions where a single-equation model isn't enough.
- **Bootstrap forecast intervals**, when the residuals are not normal (see [[Bootstrap for Time Series]]).

## See also

- [[Stochastic Processes and Stationarity]]
- [[Unit Roots and Spurious Regression]]
- [[Unit Root Tests Beyond ADF]]
- [[Seasonal Adjustment and Decomposition]]
- [[Information Criteria AIC and BIC]]
- [[Residual Diagnostics for Time Series]]
- [[Forecast Evaluation Metrics]]
- [[Bootstrap for Time Series]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 22 (Time Series Econometrics: Forecasting).

Write by **Samuel**
