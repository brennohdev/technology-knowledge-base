---
tags: [econometrics, statistics, multicollinearity, regression, data-science]
---

# Multicollinearity

Multicollinearity happens when **two or more independent variables in a regression model are highly correlated**. In that situation, the model struggles to separate the individual effect of each variable, because they move together.

It's one of the most common problems in applied econometrics, and it doesn't invalidate your predictions — but it does wreak havoc on inference.

## Intuition

Imagine you're trying to estimate how **height** and **leg length** both affect **jump distance**. Since taller people tend to have longer legs, these variables carry almost the same information. The regression can't reliably tell you "how much of the jump is due to height vs leg length" because when one goes up, the other goes up too.

That's multicollinearity in a nutshell: **redundant information across predictors**.

## Perfect vs Imperfect

There are two flavors:

- **Perfect multicollinearity**: an exact linear relationship, e.g., including both `height_in_cm` and `height_in_inches` in the same model. OLS literally cannot estimate the coefficients — the design matrix is singular.
- **Imperfect multicollinearity (the real-world case)**: high but not exact correlation. OLS still runs, but estimates become **unstable and imprecise**.

In practice, we almost always worry about the imperfect kind.

## Why is it a problem?

1. **Inflated standard errors**: coefficients stay unbiased, but their standard errors blow up. A variable that *is* significant in reality might look insignificant just because of noise from collinearity.
2. **Unstable coefficients**: add or remove a correlated variable and the estimated effects swing wildly.
3. **Interpretation difficulty**: you can't trust the individual coefficient as "the effect of X holding everything else constant", because the "everything else" isn't really constant when it moves in lockstep with X.

## How to detect it

### Pairwise correlation
Look at the correlation matrix of independent variables. High absolute correlations (e.g., > 0.8) are a red flag, but they don't tell the whole story — multicollinearity can exist even without any single pair being extremely correlated.

### Variance Inflation Factor (VIF)
VIF quantifies how much the variance of an estimated coefficient is inflated by collinearity:

```
VIF_j = 1 / (1 - R_j²)
```

Where `R_j²` is the R-squared from regressing variable `j` on all the other independent variables.

Rule of thumb:
- VIF < 5: generally okay
- VIF 5–10: moderate collinearity, investigate
- VIF > 10: serious collinearity, consider remedial action

### Condition Number
The condition number of `(X'X)` tells you how "ill-conditioned" the matrix is. Values above 30 suggest moderate problems; above 100 are concerning.

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

## See also

- [[Econometrics]]

Write by **Samuel**