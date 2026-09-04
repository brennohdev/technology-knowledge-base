---
tags: [econometrics, statistics, time-series, unit-root, dickey-fuller]
---

# Unit Roots and Spurious Regression

[[Stochastic Processes and Stationarity]] introduces *why* stationarity matters and the visual/informal ways to suspect its absence. This note is about the formal test, the **unit root test**, and about the specific disaster that happens if you skip it: **spurious regression**.

## The unit root, formally

Take a simple AR(1) process:

$$Y_t = \rho\, Y_{t-1} + u_t, \qquad -1 \le \rho \le 1$$

- If $|\rho| < 1$: the process is stationary, shocks decay over time, $Y_t$ reverts toward its mean.
- If $\rho = 1$: this is exactly the random walk from [[Stochastic Processes and Stationarity]], $Y_t = Y_{t-1} + u_t$. The process has a **unit root**, it's non-stationary, and shocks never decay, they accumulate permanently.

Testing for a unit root is testing $H_0: \rho = 1$ against $H_1: \rho < 1$.

## Why you can't just run a t-test on $\rho$

Subtract $Y_{t-1}$ from both sides:

$$\Delta Y_t = (\rho - 1)\,Y_{t-1} + u_t = \gamma\, Y_{t-1} + u_t, \qquad \gamma = \rho - 1$$

Now $H_0: \rho=1$ is the same as $H_0: \gamma = 0$, which looks like an ordinary t-test on a regression coefficient. But it isn't one: **under $H_0$, $Y_{t-1}$ itself is non-stationary**, so the usual t-distribution doesn't apply. Dickey and Fuller worked out the correct (non-standard) critical values for this exact situation, that's the **Dickey-Fuller (DF) test**.

## Dickey-Fuller test

Three common specifications, depending on what you believe about the series:

1. No constant, no trend: $\Delta Y_t = \gamma\, Y_{t-1} + \varepsilon_t$
2. With constant (drift): $\Delta Y_t = \alpha + \gamma\, Y_{t-1} + \varepsilon_t$
3. With constant and trend: $\Delta Y_t = \alpha + \beta t + \gamma\, Y_{t-1} + \varepsilon_t$

In all three: $H_0: \gamma = 0$ (unit root, non-stationary) vs. $H_1: \gamma < 0$ (stationary). Compare the t-statistic on $\gamma$ against **Dickey-Fuller critical values** (more negative than standard t critical values), not the usual t-table.

## Augmented Dickey-Fuller (ADF)

The plain DF test assumes $u_t$ is white noise, if there's [[Autocorrelation]] left in the errors, the test is invalid. The **Augmented Dickey-Fuller** test fixes this by adding lagged difference terms as controls:

$$\Delta Y_t = \alpha + \beta t + \gamma\, Y_{t-1} + \sum_{i=1}^{p} \delta_i\, \Delta Y_{t-i} + \varepsilon_t$$

Same hypothesis test on $\gamma$, but now robust to autocorrelation in the residuals, as long as enough lags $p$ are included (chosen via information criteria like AIC/BIC, or by adding lags until the residuals are white noise). ADF is what's actually used in practice, plain DF is mostly a teaching stepping stone.

## Making a series stationary

If a unit root is found ($H_0$ not rejected), the standard fix is **differencing**: work with $\Delta Y_t$ instead of $Y_t$. Test the differenced series for a unit root again, if it's now stationary, the original series was $I(1)$ (see [[Stochastic Processes and Stationarity]]). Occasionally a series needs a second difference, $I(2)$, worth double-checking the data in that case, since it's less common for well-constructed economic series.

## Spurious regression

This is the concrete cost of skipping all of the above. Run a regression between two **independent, unrelated, non-stationary** series, and you can still get:

- A high $R^2$ (sometimes above 0.9)
- Statistically significant t-statistics on the slope
- A low Durbin-Watson statistic (a giveaway sign, see [[Autocorrelation]], strongly autocorrelated residuals are a symptom, not a coincidence)

None of it means anything, the "relationship" is an artifact of both series trending, not a real economic connection (Granger and Newbold's classic 1974 paper demonstrated this with simulated, entirely unrelated random walks). The practical rule this leads to: **check for unit roots before regressing trending time series on each other in levels.** If both series are $I(1)$ but a linear combination of them is stationary, that's a different, legitimate case, **cointegration**, worth its own note later, since it changes the right modeling approach rather than just being a warning sign.

## See also

- [[Stochastic Processes and Stationarity]]
- [[ARIMA and the Box-Jenkins Approach]]
- [[Autocorrelation]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 21.

Write by **Samuel**
