---
tags: [econometrics, statistics, time-series, unit-root, dfgls, kpss, ng-perron]
---

# Unit Root Tests Beyond ADF

The Augmented Dickey-Fuller test (see [[Unit Roots and Spurious Regression]]) is the standard first tool, but it has known weaknesses, especially in small samples. This note covers why, and the tests used to complement it: Phillips-Perron, KPSS, DF-GLS (ERS) and Ng-Perron.

## Why the Dickey-Fuller test has problems

1. **Finite-sample bias of the autoregressive coefficient.** The OLS estimate of $\rho$ in $Y_t = \rho Y_{t-1} + u_t$ is biased downward when $\rho$ is close to 1. For an AR(1) with an intercept, Kendall's approximation is

$$E(\hat{\rho}) \approx \rho - \frac{1 + 3\rho}{T}$$

   so for a random walk the bias is about $-4/T$. With $T = 30$ the bias is large; with $T = 500$ it practically disappears. A Monte Carlo with random walks gave an average $\hat{\rho} - 1$ of about $-0.16$ for $T = 30$ and about $-0.01$ for $T = 500$.
2. **Non-standard distribution.** Under the null, $Y_{t-1}$ is non-stationary, so the t statistic does not follow a Student t. The correct critical values (Dickey-Fuller, MacKinnon) are more negative, and the test is not "biased" in size when they are used, but it is still imprecise.
3. **Low power.** When the true $\rho$ is close to, but below, 1 (for example 0.95), the ADF often fails to reject the unit root, especially with few observations and when a constant and trend have to be estimated.
4. **Sensitivity to deterministic terms and lag length.** Including a trend that is not needed, or choosing too few lags, changes the conclusion.

## Phillips-Perron (PP)

Same regression as DF, but instead of adding lagged differences, it corrects the t statistic **non-parametrically** for serial correlation and heteroskedasticity (a Newey-West long-run variance). Null: unit root. It is robust to unspecified error dynamics but can have size distortions when the errors have strong negative MA components.

## KPSS

The roles of the hypotheses are **reversed**:

- $H_0$: the series is stationary (around a constant or a trend).
- $H_1$: the series has a unit root.

The statistic is based on the partial sums $S_t$ of the residuals:

$$\text{KPSS} = \frac{\sum_{t=1}^{T} S_t^2}{T^2 \hat{\sigma}^2_{lr}}$$

Using it alongside ADF gives a **confirmatory analysis**: unit root not rejected by ADF and stationarity rejected by KPSS point to a unit root; the opposite pattern points to stationarity; conflicting outcomes mean the data are not informative enough.

## DF-GLS (Elliott, Rothenberg and Stock)

Increases power with a two-step approach:

1. **Detrend by GLS.** Quasi-difference the series with $\bar{\alpha} = 1 + \bar{c}/T$, where $\bar{c} = -7$ for a constant and $\bar{c} = -13.5$ for a constant and trend, and estimate the deterministic terms from the quasi-differenced data. Subtract them from $Y_t$ to get $\tilde{Y}_t$.
2. **Run an ADF-type regression on $\tilde{Y}_t$** without deterministic terms.

The null is still a unit root, but the critical values differ from the standard DF ones. In small samples DF-GLS is clearly more powerful than ADF, which is why it is preferred when the sample is short.

## Ng-Perron

Modified tests (MZ$_\alpha$, MZ$_t$, MSB, MPT) built on the GLS-detrended series, with a modified information criterion for the lag choice. They fix the size distortions of PP and DF-GLS when the error process has a root near $-1$. With $\tilde{y}$ the detrended series and $f_0$ an estimate of the spectral density at frequency zero:

$$MZ_\alpha = \frac{T^{-1}\tilde{y}_T^2 - f_0}{2 T^{-2}\sum_{t=1}^{T}\tilde{y}_{t-1}^2}, \qquad MSB = \sqrt{\frac{T^{-2}\sum_{t=1}^{T}\tilde{y}_{t-1}^2}{f_0}}, \qquad MZ_t = MZ_\alpha \times MSB$$

All reject for sufficiently **negative** values (MSB and MPT reject for small values).

## A decision routine

1. Plot the series and choose the deterministic terms (constant, or constant and trend).
2. Run ADF and DF-GLS; add KPSS (and PP or Ng-Perron when the sample is small).
3. If the tests agree on a unit root, difference once and repeat the tests on the differenced series.
4. Stop at the smallest $d$ for which the tests reject the unit root; differencing more is over-differencing (see [[ARIMA and the Box-Jenkins Approach]]).

## Worked example

Seasonally adjusted Brazilian electricity consumption (60 months, constant and trend): ADF statistic about $-3.16$ (p about 0.09) and DF-GLS about $-2.30$ (p about 0.18), so the unit root is not rejected. After one difference, ADF about $-7.22$ and DF-GLS about $-3.56$, both with p below 0.001, so the series is $I(1)$ and $d = 1$.

## See also

- [[Stochastic Processes and Stationarity]]
- [[Unit Roots and Spurious Regression]]
- [[ARIMA and the Box-Jenkins Approach]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011, Chapter 21 (Time Series Econometrics: Some Basic Concepts).

ELLIOTT, G.; ROTHENBERG, T. J.; STOCK, J. H. Efficient tests for an autoregressive unit root. *Econometrica*, 64(4), 1996.

NG, S.; PERRON, P. Lag length selection and the construction of unit root tests with good size and power. *Econometrica*, 69(6), 2001.

KWIATKOWSKI, D.; PHILLIPS, P.; SCHMIDT, P.; SHIN, Y. Testing the null hypothesis of stationarity against the alternative of a unit root. *Journal of Econometrics*, 54, 1992.

Write by **Samuel**
