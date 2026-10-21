---
tags: [econometrics, statistics, time-series, diagnostics, ljung-box, arch, normality]
---

# Residual Diagnostics for Time Series

After estimating an ARIMA (see [[ARIMA and the Box-Jenkins Approach]]), the diagnostic step asks one question: did the model extract all the structure? If so, the residuals should behave like **white noise** (see [[Stochastic Processes and Stationarity]]): zero mean, constant variance and no autocorrelation. For maximum likelihood with Gaussian errors, they should also be approximately normal.

In every test below the null hypothesis is the "good news", so a **high p-value is the desired result**. Failing to reject means there is no evidence of a problem, not proof that the model is right.

## The four tests

| Test | $H_0$ | Detects |
|---|---|---|
| Ljung-Box | No autocorrelation up to lag $h$ | Leftover serial correlation |
| Breusch-Godfrey | No serial correlation up to order $p$ | Leftover serial correlation in a regression |
| ARCH-LM | No ARCH effect (constant conditional variance) | Volatility that changes over time |
| Jarque-Bera | Normality | Skewness and excess kurtosis |

### Ljung-Box

Tests several autocorrelations at once:

$$Q = n(n+2)\sum_{k=1}^{h}\frac{\hat{\rho}_k^2}{n-k} \;\sim\; \chi^2(h - p - q)$$

Choose $h$ by the periodicity, for example 12 for monthly data, and not too large relative to $n$, because power falls. Adjust the degrees of freedom by the number of ARMA parameters estimated ($p+q$); with an $ARIMA(0,1,0)$ there is nothing to subtract.

### Breusch-Godfrey

An LM test for serial correlation of order $p$ that, unlike the Durbin-Watson, allows lagged dependent variables and higher-order autocorrelation. It regresses the residuals on the original regressors and on $p$ lagged residuals:

$$LM = n R^2_{aux} \;\sim\; \chi^2(p)$$

It requires a regression estimated by OLS. For an ARIMA with no regressors, a practical adaptation is to run the equivalent regression, for example $\Delta Y_t$ on a constant for an $ARIMA(0,1,0)$, and apply the test to it.

### ARCH-LM (Engle)

Regress the squared residuals on their own $q$ lags:

$$\hat{\varepsilon}_t^2 = \alpha_0 + \alpha_1 \hat{\varepsilon}_{t-1}^2 + \dots + \alpha_q \hat{\varepsilon}_{t-q}^2 + v_t, \qquad LM = nR^2 \sim \chi^2(q)$$

Rejecting means the variance depends on the past, and a volatility model such as GARCH is needed (the ARIMA mean equation can still be fine, but intervals will be wrong).

### Jarque-Bera

Compares skewness $S$ and kurtosis $K$ with those of a normal distribution (0 and 3):

$$JB = \frac{n}{6}\left(S^2 + \frac{(K-3)^2}{4}\right) \;\sim\; \chi^2(2)$$

Has low power in small samples, so the histogram and a Q-Q plot should accompany it.

## Plots to read together

- **Residuals over time**: should oscillate around zero with no long runs and no change in amplitude.
- **Residual ACF**: bars inside the 95% band at all lags, including the seasonal lag.
- **Histogram or Q-Q plot**: bell shape close to the normal curve.

## If a test rejects

| Rejection | Likely meaning | What to try |
|---|---|---|
| Ljung-Box or Breusch-Godfrey | Missing AR or MA terms, or seasonality left | Increase $p$ or $q$, check $d$, check the seasonal lag |
| ARCH-LM | Time-varying volatility | GARCH on the residuals, or log transformation |
| Jarque-Bera | Outliers or fat tails | Treat outliers, use bootstrap intervals (see [[Bootstrap for Time Series]]) |

## Worked example

Residuals of an $ARIMA(0,1,0)$ for seasonally adjusted Brazilian electricity consumption (47 residuals, excluding the first, which only reflects the initial level): Ljung-Box p-values 0.42 (6 lags) and 0.70 (12 lags), Breusch-Godfrey 0.60 (6 lags), ARCH-LM 0.22 (4 lags), Jarque-Bera 0.60 (skewness about $-0.28$, kurtosis about 2.54). None rejects at 5%, the residual ACF stayed inside the band and the histogram followed the normal curve, so the model is adequate.

## See also

- [[ARIMA and the Box-Jenkins Approach]]
- [[Information Criteria AIC and BIC]]
- [[Autocorrelation]]
- [[Heteroscedascity]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011, Chapters 12 and 22.

LJUNG, G.; BOX, G. On a measure of lack of fit in time series models. *Biometrika*, 65(2), 1978.

ENGLE, R. Autoregressive conditional heteroscedasticity with estimates of the variance of United Kingdom inflation. *Econometrica*, 50(4), 1982.

JARQUE, C.; BERA, A. Efficient tests for normality, homoscedasticity and serial independence of regression residuals. *Economics Letters*, 6(3), 1980.

Write by **Samuel**
