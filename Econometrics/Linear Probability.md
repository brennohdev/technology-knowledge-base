---
tags: [econometrics, statistics, regression, logit, probit, binary-outcome]
---

# Linear Probability, Logit and Probit

These are the three standard approaches when the **dependent** variable is binary (0/1), a qualitative outcome, rather than continuous. This is a different problem from [[Dummy Variables]], which is about qualitative *explanatory* variables — here the thing being explained itself only takes two values (participates/doesn't, defaults/doesn't, survives/doesn't...).

## Linear Probability Model (MPL)

The naive approach: just run OLS with a 0/1 dependent variable.

$$P(Y_i=1) = \beta_0 + \beta_1 X_{1i} + \dots + \beta_k X_{ki} + u_i$$

It technically works and coefficients are easy to read (a straight percentage-point change in probability per unit of $X$), but it has three real problems:

1. **Predicted probabilities aren't bounded to $[0,1]$** — nothing in OLS stops a fitted value from being negative or above 1.
2. **Heteroscedasticity by construction** — $\text{Var}(u_i) = P_i(1-P_i)$ mechanically depends on $X_i$, so the [[Heteroscedasticity]] assumption is violated by design, not by bad luck.
3. **Constant marginal effect assumption is implausible** — MPL assumes the effect of $X_j$ on the probability is the same everywhere, but realistically, a nudge in $X$ should move the probability a lot for someone near 50/50 and very little for someone already near 0 or 1.

## Why OLS isn't the right tool here

All three MPL problems above boil down to: OLS doesn't impose any functional restriction that keeps outputs looking like valid probabilities. Logit and Probit fix this directly.

## Logit and Probit

Both model:

$$P(Y_i=1 \mid X_i) = F(X_i'\beta)$$

where $F(\cdot)$ is a CDF that maps any real number into $(0,1)$.

- **Logit** uses the logistic CDF: $F(z) = \dfrac{e^z}{1+e^z}$
- **Probit** uses the standard normal CDF: $F(z) = \Phi(z)$

The two distributions look very similar (logistic has slightly heavier tails), so in practice Logit and Probit tend to agree closely on signs, significance, and marginal effects. Probit coefficients are consistently smaller in magnitude than Logit's, roughly by a factor of ~0.60 (Probit/Logit), reflecting the scale difference between the two distributions — this is a good sanity check when comparing the two side by side.

## Interpreting coefficients

- **Logit coefficients** are directly readable as **log-odds**. Exponentiate to get an **odds ratio**: $e^{\hat\beta_j}$ tells you how the odds of $Y=1$ multiply for a one-unit increase in $X_j$.
- **Probit coefficients** have no such direct reading, only the sign is immediately interpretable. Magnitude requires marginal effects.

## Marginal effects

Since $F(\cdot)$ is non-linear, $\partial P/\partial X_j = f(X'\beta)\,\beta_j$ depends on *where* you evaluate it, unlike the MPL, where the marginal effect is constant everywhere. In practice you report **average marginal effects** (`get_margeff()` in `statsmodels`), which *are* directly comparable in scale, both to each other (Logit vs. Probit) and to the raw MPL coefficients, since all three end up in the same unit: percentage-point change in probability.

## Evaluating predictive performance

For a fitted binary model, classify $\hat y = 1$ when $\hat P(Y=1) \geq 0.5$ (or another chosen threshold), then build a confusion matrix:

|              | Predicted 0 | Predicted 1 |
|--------------|-------------|-------------|
| **Actual 0** | True Neg.   | False Pos.  |
| **Actual 1** | False Neg.  | True Pos.   |

- **Accuracy** = (TP+TN) / total
- **Sensitivity** (recall for the positive class) = TP / (TP+FN)
- **Specificity** (recall for the negative class) = TN / (TN+FP)

A model can have decent accuracy while being lopsided, high sensitivity but poor specificity (or vice versa), which usually means it's systematically over- or under-predicting one class at the default 0.5 threshold.

## From our labor force participation model

Worked example: Mroz (1987) dataset, 753 married women, `inlf` (in labor force) as $Y$, with `nwifeinc`, `educ`, `age`, `kidslt6` (quantitative) and `city` (the one [[Dummy Variables|dummy]]) as regressors.

- MPL: $R^2 = 0.146$; 1.59% of fitted probabilities landed outside $[0,1]$ — the theoretical limitation showing up empirically, not just in theory.
- Logit and Probit: pseudo-$R^2 \approx 0.117$ in both, essentially identical log-likelihood. All three models agreed exactly on which variables were significant: `nwifeinc`, `educ`, `age`, `kidslt6` significant at 1%; `city` not significant in any of the three.
- Odds ratio example (Logit): each extra year of `educ` multiplies the odds of participating by $e^{0.2621}\approx 1.30$ (+30%); each additional young child multiplies the odds by $e^{-1.4585}\approx 0.233$ (~-77%), by far the strongest single driver in the model.
- Average marginal effects (Logit/Probit) landed very close to the raw MPL coefficients (e.g. `kidslt6`: $-0.294$ MPL vs. $-0.304$/$-0.303$ Logit/Probit), which makes sense here since most fitted probabilities sit reasonably close to the 0.5 region, where the non-linearity of Logit/Probit matters least.
- Predictive check (Logit, threshold 0.5): 66.7% accuracy, 79.0% sensitivity, only 50.5% specificity — the model was noticeably better at catching true participants than true non-participants, worth flagging rather than just reporting the headline accuracy number.

## See also

- [[Econometrics]]
- [[Dummy Variables]]
- [[Heteroscedasticity]] — MPL's error term is heteroscedastic by construction, this is where that concept resurfaces

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 15 (Qualitative Response Regression Models).

WOOLDRIDGE, Jeffrey M. *Introductory Econometrics: A Modern Approach*. 7th ed. Boston: Cengage Learning, 2019.

Write by **Samuel**