---
tags: [econometrics, statistics, regression, autocorrelation, time-series]
---

# Autocorrelation

Autocorrelation (serial correlation) is a violation of another core [[Econometrics|CLRM]] assumption: independence of the error terms. Formally, $\text{Cov}(u_t, u_s) \neq 0$ for $t \neq s$. It's mostly a time-series problem — the most common form is first-order autocorrelation, AR(1):

$$u_t = \rho\, u_{t-1} + \varepsilon_t$$

where the current period's error is correlated with the previous period's.

## When it shows up

- Inertia / persistence in the economic phenomenon being modeled.
- Omitted variables that themselves have a time pattern (a classic case: not removing seasonality from a series that has it).
- Wrong functional form.
- Data manipulation issues (e.g. interpolation).

## Consequences

Same structure as [[Heteroscedasticity]]: OLS coefficients remain **unbiased**, but are no longer **efficient** (not BLUE anymore), and estimated standard errors become biased — typically *underestimated* when $\rho > 0$ (the common case), which inflates t-statistics and leads to spurious rejections of $H_0$ (finding "significance" that isn't really there).

## Detecting it

**Durbin-Watson test**:
- $H_0$: no first-order autocorrelation ($\rho = 0$)
- $H_1$: first-order autocorrelation ($\rho \neq 0$)
- The statistic $d$ ranges from 0 to 4. Values near 2 = no autocorrelation; near 0 = positive autocorrelation; near 4 = negative autocorrelation. No single rejection region, there's an indecision zone bounded by tabulated $d_L$ and $d_U$.

**Breusch-Godfrey test**: more general — works with lagged dependent variables as regressors (where Durbin-Watson is invalid) and tests higher-order autocorrelation, not just AR(1).

**Residuals-over-time plot**: visually, look for cycles — runs of several consecutive positive residuals followed by runs of negative ones, instead of random oscillation around zero.

## Fixing it

If only autocorrelation is present (no heteroscedasticity), or if both are present together, the standard fix is **HAC standard errors** (Heteroscedasticity and Autocorrelation Consistent), most commonly the **Newey-West** estimator. Like robust SEs for heteroscedasticity, this corrects the variance-covariance matrix without touching the OLS coefficients.

For a bigger gain, recovering *efficiency* (not just fixing inference) requires **GLS/FGLS** (Generalized / Feasible Generalized Least Squares): model the error structure explicitly, transform the data, and re-run OLS on the transformed data. Worth it when the error structure is reasonably well understood or estimable, and when the efficiency gain actually matters (e.g. for forecasting or tighter confidence intervals).

## From our aggregate consumption model

Worked example: monthly aggregate consumption model (non-seasonally-adjusted retail index as the dependent variable). Durbin-Watson $d = 1.293$, below the tabulated lower bound $d_L \approx 1.65$ for $n=122$, $k=4$ → reject $H_0$. Breusch-Godfrey confirmed it strongly (LM = 16.44, p = 0.0025). The residuals-over-time plot showed exactly the expected cyclical pattern. Makes sense: the dependent variable wasn't seasonally adjusted, so leftover seasonality got absorbed into the error term. Re-estimating with Newey-West (HAC) widened some standard errors moderately but didn't flip any conclusion about which coefficients were significant.

## See also

- [[Econometrics]]
- [[Heteroscedasticity]]
- [[Multicollinearity]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 12.

Write by **Samuel**