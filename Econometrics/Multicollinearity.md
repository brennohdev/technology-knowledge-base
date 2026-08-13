---
tags: [econometrics, statistics, regression, multicollinearity]
---

# Multicollinearity

Multicollinearity is when two or more explanatory variables in a regression are strongly correlated with each other. It's not one of the core Gauss-Markov assumptions of the [[Econometrics|CLRM]], but the CLRM does require the *absence of perfect* collinearity for OLS to even be computable — and near-perfect (imperfect) collinearity, while technically allowed, causes real practical problems.

## Perfect vs. imperfect

- **Perfect multicollinearity**: one explanatory variable is an exact linear combination of others. $X'X$ isn't invertible, OLS literally can't be estimated.
- **Imperfect multicollinearity**: the realistic case. Two or more regressors are highly but not perfectly correlated. The model *can* be estimated, but with a cost.

## Effect on the estimators

This is the counter-intuitive part: under imperfect multicollinearity, OLS estimators are **still BLUE** — no violation of Gauss-Markov assumptions occurs. What happens instead is:

- Standard errors of the affected coefficients become very large.
- Confidence intervals widen.
- Coefficient estimates become unstable / sensitive to small changes in the sample or specification.
- Individual t-tests can fail to reject $H_0: \beta_j = 0$ even when the joint F-test rejects it and $R^2$ is high — the classic multicollinearity symptom.

## Why it hurts interpretation

With strongly correlated regressors, it's hard to isolate the marginal effect of one variable holding the others constant — which is exactly what a *ceteris paribus* coefficient is supposed to mean. This can also flip a coefficient's sign relative to what theory predicts.

## Diagnosing it

**Correlation matrix** between explanatory variables — a quick first look, pairwise correlations above ~0.8 are a warning sign.

**Variance Inflation Factor (VIF)** — the real diagnostic. For each regressor $X_j$:

$$\text{VIF}_j = \frac{1}{1 - R_j^2}$$

where $R_j^2$ comes from regressing $X_j$ on all the *other* explanatory variables. Rule of thumb: $\text{VIF}_j > 10$ (equivalent to $R_j^2 > 0.90$ in that auxiliary regression) signals problematic multicollinearity.

## Corrective strategies (and their trade-offs)

1. **Drop one of the collinear variables** — simple, immediately fixes VIF, but risks omitted-variable bias if the dropped variable actually mattered.
2. **Get more data** — reduces estimator variance without touching the specification, but more historical/cross-sectional data isn't always available.
3. **Combine collinear variables into an index** (e.g. principal components) — keeps the joint information, but loses individual economic interpretation.
4. **Reformulate the model** (e.g. first differences instead of levels) — can remove collinearity driven by a common trend, but changes what the coefficient means (short-run vs. level effect).

## From our aggregate consumption model

Worked example (see [[Econometrics#Worked examples from our own work]]): regressing an aggregate retail sales index on income, interest rate, inflation and consumer confidence, all VIFs came out low (renda 2.75, juros 2.05, inflação 1.01, confiança 2.34), well under the threshold of 10. Conclusion: no problematic multicollinearity, so the large standard errors on some coefficients in that model weren't a multicollinearity artifact — they reflected genuinely weak statistical relationships with the dependent variable.

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 10.

## See also

- [[Econometrics]]
- [[Heteroscedasticity]]
- [[Autocorrelation]]

Write by **Samuel**
