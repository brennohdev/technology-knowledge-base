---
tags: [econometrics, statistics, time-series, stationarity, stochastic-process]
---

# Stochastic Processes and Stationarity

Everything in [[Econometrics]] so far (CLRM, [[Heteroscedasticity]], [[Autocorrelation]]) assumed a fixed structure for the data, but didn't ask a more basic question that matters a lot once you're doing time series properly: is the *process generating* $Y_t$ itself stable over time, or does its behavior change as $t$ grows? That question is what stationarity is about, and it's the starting point of Econometrics II / time series econometrics.

## What is a stochastic process

A **stochastic process** is just a sequence of random variables indexed by time: $\{Y_t\}$, $t = 1, 2, 3, \dots$. Each $Y_t$ is a random variable, and a time series (the actual data you observe, GDP each quarter, a stock price each day) is one particular *realization* of that process, one draw out of all the paths the process could have taken.

## Stationarity: the core idea

A process is **stationary** if its statistical properties don't change over time, the process "looks the same" whether you examine it now or ten periods from now.

### Strict stationarity

The joint probability distribution of $(Y_t, Y_{t+1}, \dots, Y_{t+k})$ is the same as that of $(Y_{t+s}, Y_{t+s+1}, \dots, Y_{t+s+k})$ for any $s$, any shift in time doesn't change the joint distribution. This is a strong condition, rarely checked directly in practice.

### Weak (covariance) stationarity

What's actually used in applied work. A process is weakly stationary if:

1. $E(Y_t) = \mu$ — constant mean, doesn't depend on $t$.
2. $\text{Var}(Y_t) = \sigma^2$ — constant variance, doesn't depend on $t$.
3. $\text{Cov}(Y_t, Y_{t+k}) = \gamma_k$ — the covariance between two periods depends only on the *lag* $k$ between them, not on $t$ itself.

If any of these three fails, the process is **non-stationary**.

## Two canonical examples

**White noise** (stationary): $u_t$ with $E(u_t)=0$, constant variance $\sigma^2$, and $\text{Cov}(u_t, u_{t+k})=0$ for $k \neq 0$. No memory, no trend, no changing spread. The textbook example of a stationary process, it's also exactly what a well-specified regression's error term should look like (tying back to [[Autocorrelation]]).

**Random walk** (non-stationary): $Y_t = Y_{t-1} + u_t$, with $u_t$ white noise. Innocuous-looking, but:

- $\text{Var}(Y_t) = t\,\sigma^2$ — the variance grows with $t$, so property 2 above fails immediately.
- The process has no tendency to return to any particular mean, it wanders. Stock prices and many macroeconomic series (GDP, price levels) behave a lot like this.

## Why non-stationarity is a real problem, not just a technicality

If you run a regression of one non-stationary series on another, even two series that have *nothing to do with each other*, you can get a high $R^2$ and significant t-statistics purely because both are trending. This is called **spurious regression**, and it's the single biggest reason time series econometrics treats stationarity as a first-class concern instead of an afterthought (full treatment in [[Unit Roots and Spurious Regression]]).

## Deterministic trend vs. stochastic trend

Non-stationarity isn't all the same kind:

- **Trend-stationary**: the series has a deterministic trend ($Y_t = \alpha + \beta t + u_t$, $u_t$ stationary), once you subtract out the trend, what's left is stationary. Shocks are temporary, the series reverts back to the trend line.
- **Difference-stationary** (unit root): the series only becomes stationary after *differencing* ($\Delta Y_t = Y_t - Y_{t-1}$ is stationary, even though $Y_t$ itself isn't). Shocks are permanent, they shift the whole future path, there's no fixed line to revert to. The random walk above is the classic example.

Telling these two apart matters a lot for how you should transform the data before modeling it, and it's exactly what a unit root test is built to do.

## Order of integration

A series that needs to be differenced $d$ times to become stationary is said to be **integrated of order $d$**, written $I(d)$.

- $I(0)$: already stationary, no differencing needed.
- $I(1)$: stationary after one difference (very common for macro/financial levels data).
- $I(2)$: needs two differences, less common, usually a sign something's off in how the series was constructed.

## Spotting non-stationarity before testing formally

- **Line plot**: an obviously trending series, or one whose spread visibly widens over time, is a strong visual hint.
- **Correlogram (ACF)**: for a stationary series, the autocorrelation function drops off quickly as the lag increases. For a non-stationary series, it dies out very slowly, autocorrelations near 1 even at long lags.

These are useful first checks, but the actual decision should come from a formal test, covered next.

## See also

- [[Unit Roots and Spurious Regression]]
- [[ARIMA and the Box-Jenkins Approach]]
- [[Econometrics]]
- [[Autocorrelation]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 21 (Time Series Econometrics: Some Basic Concepts).

Write by **Samuel**
