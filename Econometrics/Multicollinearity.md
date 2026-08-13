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

### Practical Python example

```python
import pandas as pd
import numpy as np
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor

# Simulated data: height (cm), weight (kg), bmi
np.random.seed(42)
n = 200
height = np.random.normal(170, 10, n)
weight = 0.5 * height + np.random.normal(0, 5, n)  # weight correlated with height
bmi = weight / ((height / 100) ** 2) + np.random.normal(0, 1, n)

df = pd.DataFrame({"height": height, "weight": weight, "bmi": bmi})

# Adding a target: daily calorie expenditure
expenditure = 1500 + 10 * weight + np.random.normal(0, 100, n)
df["expenditure"] = expenditure

# VIF
X = df[["height", "weight", "bmi"]]
X_const = sm.add_constant(X)

vif_data = pd.DataFrame()
vif_data["feature"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X_const.values, i + 1) for i in range(len(X.columns))]

print(vif_data)
```

Typical output:

```
  feature       VIF
0   height  45.231891
1   weight  52.184203
2     bmi  38.745612
```

VIFs above 10 confirm severe multicollinearity. Notice that `weight` and `height` are highly correlated, and `bmi` is a deterministic function of both.

## What to do about it

### 1. Theory first
Before dropping variables, ask: what does economic theory say? If two variables measure almost the same thing theoretically (e.g., `height` and `leg_length`), you should probably keep only one.

### 2. Remove redundant variables
If `VIF > 10`, consider dropping one of the highly correlated variables. Use theoretical criteria, not just statistical ones.

### 3. Combine variables
Transform correlated variables into an index or principal component (PCA) can capture the shared information without redundancy.

### 4. Ridge / Regularization
Models with L2 penalty (Ridge regression) reduce the impact of multicollinearity by shrinking correlated coefficients toward stable values. It's a common fix for prediction, but the trade-off is that individual coefficients become **biased** — what matters is predictive performance.

### 5. Collect more data
Sometimes multicollinearity stems from a small sample or a study design constraint. More observations can alleviate the problem.

## Common pitfalls

- **Thinking high correlation between two variables is enough to drop one**: it's not. Sometimes both are theoretically relevant, and you can use regularization instead of removal.
- **Ignoring moderate VIFs (5–10)**: they may not require action, but it's worth documenting and monitoring.
- **Confusing multicollinearity with endogeneity**: these are different problems. Multicollinearity is about correlation between *regressors*. Endogeneity is about correlation between a regressor and the error term.

## Visual diagnostic example

```python
import seaborn as sns
import matplotlib.pyplot as plt

corr = df[["height", "weight", "bmi"]].corr()
sns.heatmap(corr, annot=True, cmap="coolwarm", center=0)
plt.title("Correlation Matrix")
plt.show()
```

## Quick summary

| Aspect | What happens |
|---------|---------------|
| Meaning | Highly correlated explanatory variables |
| Main effect | Inflated standard errors, unstable coefficients |
| Impact on prediction | Low (predictions still work) |
| Impact on inference | High (significance tests unreliable) |
| Detection | Correlation, VIF, condition number |
| Solutions | Theory, removal, PCA, regularization, more data |

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 10.

## See also

- [[Econometrics]]
- [[Heteroscedasticity]]
- [[Autocorrelation]]

Write by **Samuel**
