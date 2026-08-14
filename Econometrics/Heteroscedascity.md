---
tags: [econometrics, statistics, regression, heteroscedasticity]
---

# Heteroscedasticity

Heteroscedasticity is a violation of one of the core [[Econometrics|CLRM]] (Gauss-Markov) assumptions: constant error variance. Under homoscedasticity, $\text{Var}(u_i \mid X_i) = \sigma^2$ for every observation. Under heteroscedasticity, $\text{Var}(u_i \mid X_i) = \sigma_i^2$ — the spread of the errors changes systematically with the level of one or more explanatory variables (classic example: income and expenditure, richer households have much more *variable* spending than poorer ones, even if the average relationship is linear).

## Why it matters

The Gauss-Markov theorem needs homoscedastic errors for OLS to be **BLUE**. Break that assumption and:

- Coefficients stay **unbiased and consistent** — this is the part people often get wrong. Heteroscedasticity does *not* bias $\hat\beta$.
- OLS is no longer **efficient** — it's no longer the minimum-variance estimator among linear unbiased ones.
- Conventional standard errors become **biased**, usually underestimated, which inflates t-statistics and invalidates the usual t-tests, F-tests, and confidence intervals.

So the real damage is to *inference*, not to the point estimates.

## Detecting it

**Visual check**: plot squared residuals against fitted values. A funnel / fan shape (variance growing or shrinking with the fitted value) is the classic tell.

**Breusch-Pagan test**:
- $H_0$: homoscedasticity (residual variance doesn't depend on the regressors)
- $H_1$: heteroscedasticity
- Regresses squared residuals on the original regressors; under $H_0$, $nR^2$ from that auxiliary regression is asymptotically $\chi^2$ with degrees of freedom equal to the number of regressors.

**White test**: a more general version — also includes squared terms and cross-products of the regressors, so it doesn't assume a specific functional form for the heteroscedasticity. More general, but has less power in small samples.

## Fixing it: robust standard errors

If detected, the most direct fix, without touching the coefficients, is re-estimating with **heteroscedasticity-robust standard errors** (HC0/HC1/HC2/HC3 — HC3 is generally recommended for small/moderate samples). This recalculates the variance-covariance matrix of the estimators without changing $\hat\beta$ at all.

## From our aggregate consumption model

Worked example: Breusch-Pagan (LM = 6.09, p = 0.192) and White (LM = 21.63, p = 0.086) on a monthly aggregate consumption model. Neither rejected $H_0$ at 5% (White was borderline at 10%), so no strong evidence of heteroscedasticity. As expected, re-estimating with HC3 barely moved any standard error or p-value, no variable flipped significance status, since when $H_0$ isn't rejected, robust and conventional standard errors converge to estimating the same asymptotic variance.

## See also

- [[Econometrics]]
- [[Autocorrelation]] — the other classic error-term problem, often discussed together
- [[Multicollinearity]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 11.

Write by **Samuel**