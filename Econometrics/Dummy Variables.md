---
tags: [econometrics, statistics, regression, dummy-variables]
---

# Dummy Variables

Dummy variables are binary (0/1) variables used to bring qualitative or categorical information, sex, region, urban/rural, a treatment group, into a regression that otherwise works with continuous numeric variables. They don't have a natural numeric scale, so a dummy just flags category membership.

## Purpose

They let a regression capture differences in **intercept** between groups (and, if interacted with other regressors, differences in **slope** too), so group comparisons can happen inside a single unified model instead of running separate regressions per group.

## Reference category

When a qualitative variable has $m$ categories, you only include $m-1$ dummies. The omitted category is the **reference (base) category**, and every dummy's coefficient is interpreted *relative to it*. Example: if `city` = 1 for urban and 0 for rural, the reference category is "rural", and the coefficient on `city` measures the effect of being urban *relative to* being rural.

## The Dummy Variable Trap

If you include dummies for **all** categories of a qualitative variable *and* keep the model's intercept, you get perfect multicollinearity: the sum of all the dummies always equals 1, identical to the constant column, so $X'X$ isn't invertible. Fix: always drop one category (use $m-1$ dummies for $m$ categories). In `pandas`, `pd.get_dummies(df["var"], drop_first=True)` handles this directly.

## Practical note

If a categorical variable is already binary in the source data (like `city` above), there's technically nothing to "construct" — but the same logic (which category is the reference, what does the coefficient mean relative to it) still applies, and it's worth stating explicitly rather than assuming it's obvious.

## From our labor force participation model

Worked example (see [[Linear Probability, Logit and Probit]] for the full model): `city` (urban = 1, rural = 0) was included as the one qualitative regressor alongside four quantitative ones. It came out non-significant in all three specifications (MPL, Logit, Probit), meaning that once you control for family income, education, age and young children, urban vs. rural location wasn't, by itself, a relevant driver of labor force participation in that sample. A dummy not being significant is still a valid, informative result, it doesn't mean the modeling was wrong.

## See also

- [[Econometrics]]
- [[Linear Probability, Logit and Probit]]

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011 — Chapter 9.

Write by **Samuel**