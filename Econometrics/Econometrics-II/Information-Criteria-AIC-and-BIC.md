---
tags: [econometrics, statistics, time-series, model-selection, aic, bic]
---

# Information Criteria: AIC and BIC

Adding parameters to a model always improves the in-sample fit, so the fit alone cannot decide how many AR and MA terms an ARIMA needs. Information criteria solve this by rewarding fit but **penalizing complexity**. They are the standard way to choose between candidate $(p,q)$ specifications in the identification step of [[ARIMA and the Box-Jenkins Approach]].

## Definitions

With $\hat{L}$ the maximized likelihood, $k$ the number of estimated parameters and $n$ the number of observations actually used:

$$AIC = -2\ln \hat{L} + 2k \qquad\qquad BIC = -2\ln \hat{L} + k \ln n$$

**Smaller is better.** The first term falls as the fit improves; the second grows with $k$.

Other common versions:

- **HQIC**: $-2\ln \hat{L} + 2k\ln(\ln n)$, a penalty between AIC and BIC.
- **AICc**: $AIC + \dfrac{2k(k+1)}{n-k-1}$, a small-sample correction of AIC that converges to AIC when $n$ is large.

## AIC vs BIC

| | AIC | BIC |
|---|---|---|
| Penalty per parameter | 2 | $\ln n$ |
| Goal | Best out-of-sample prediction | Identify the true model |
| Tendency | Picks larger models | Picks smaller, more parsimonious models |
| Consistency | Not consistent (can overfit even with large $n$) | Consistent (recovers the true order as $n \to \infty$, if it is in the candidate set) |

BIC penalizes more as soon as $\ln n > 2$, that is, for $n \geq 8$. For any realistic series, BIC is the stricter criterion.

## When to prefer which

- **Small samples with a large grid of candidates**: BIC (or AICc), because AIC tends to overfit.
- **Forecasting as the only goal with plenty of data**: AIC is a reasonable choice.
- **When both agree**: the choice is robust and the criterion does not matter. When they disagree, report both and check the residual diagnostics (see [[Residual Diagnostics for Time Series]]) and the out-of-sample error (see [[Forecast Evaluation Metrics]]).

## How to read differences

A common rule of thumb for BIC differences against the best model: 0 to 2 is weak evidence, 2 to 6 is positive evidence for the lower value, 6 to 10 is strong, above 10 is very strong.

## Practical rules

1. **Compare models on the same data.** The same sample and the same order of differencing $d$. Likelihoods of series with different $d$ are not comparable, so $d$ is decided first, by unit root tests.
2. **Use the effective sample size.** After differencing, $n$ is reduced by $d$; software such as statsmodels uses this number in the BIC.
3. **Criteria rank candidates, they do not validate them.** The best model of a bad set is still bad, so always check the residuals.
4. **Define the grid in advance.** For example $p, q \in \{0,1,2,3\}$, estimating all 16 combinations, and report the top few.

## Worked example

Training sample of 48 months of seasonally adjusted electricity consumption, $d = 1$, grid with $p, q$ from 0 to 3. The lowest values were $ARIMA(0,1,0)$ with AIC 74.997 and BIC 76.847; the next ones were $ARIMA(0,1,1)$ and $ARIMA(1,1,0)$ with BIC around 80.4, so 3.5 points above. AIC and BIC agreed. With $k = 1$ (only the error variance) and $\ln \hat{L} = -36.498$, $AIC = 72.997 + 2 = 74.997$. The BIC used the 47 effective observations, $\ln 47 \approx 3.85$. $ARIMA(0,1,0)$ is a random walk without drift: today's value equals yesterday's plus a random shock.

## See also

- [[ARIMA and the Box-Jenkins Approach]]
- [[Residual Diagnostics for Time Series]]
- [[Forecast Evaluation Metrics]]
- [[Econometrics]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011, Chapter 22 (Time Series Econometrics: Forecasting).

AKAIKE, H. A new look at the statistical model identification. *IEEE Transactions on Automatic Control*, 19(6), 1974.

SCHWARZ, G. Estimating the dimension of a model. *Annals of Statistics*, 6(2), 1978.

Write by **Samuel**
