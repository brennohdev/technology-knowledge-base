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

## The Box-Jenkins methodology

The classic four-step process for picking and fitting an ARIMA model:

1. **Identification** — figure out $d$ (via unit root tests) and candidate $(p,q)$ by inspecting the **ACF** (autocorrelation function) and **PACF** (partial autocorrelation function) of the (differenced) series:
   - AR(p): PACF cuts off sharply after lag $p$; ACF decays gradually.
   - MA(q): ACF cuts off sharply after lag $q$; PACF decays gradually.
   - ARMA(p,q): both decay gradually, harder to read visually, information criteria (AIC/BIC) usually decide between candidate specifications.
2. **Estimation** — fit the candidate model(s), typically by maximum likelihood.
3. **Diagnostic checking** — the residuals should look like white noise (see [[Stochastic Processes and Stationarity]]). Check with a residual ACF plot and a formal test (e.g. Ljung-Box). If residuals still show structure, the model is under-specified, go back to step 1.
4. **Forecasting** — once diagnostics pass, use the fitted model to forecast future values, with the estimated uncertainty widening the further out the forecast horizon goes.

## Why this matters after unit root testing

This is the direct payoff of getting stationarity right first: an ARIMA model fit *without* checking for a unit root is really just fitting noise to a non-stationary series, and its "forecasts" tend to just extrapolate whatever trend happened to be in the sample, unreliable out of sample. Confirming $I(d)$ first, then modeling the properly differenced series, is what makes the whole exercise meaningful rather than curve-fitting.

## Where this leads next

Two natural follow-ups once ARIMA is solid, worth their own notes later as they come up in coursework:

- **Cointegration and error-correction models (ECM)** — the legitimate case where two $I(1)$ series *can* be regressed together in levels, because a linear combination of them is stationary (mentioned as a forward pointer in [[Unit Roots and Spurious Regression]]).
- **Multivariate time series (VAR)** — modeling several time series jointly, each as a function of lags of all of them, common in macroeconometrics for questions where a single-equation model isn't enough.

## See also

- [[Stochastic Processes and Stationarity]]
- [[Unit Roots and Spurious Regression]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 22 (Time Series Econometrics: Forecasting).

Write by **Samuel**
