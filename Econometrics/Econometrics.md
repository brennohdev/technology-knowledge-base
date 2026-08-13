---
tags: [econometrics, statistics, data-science, economics]
---

# Econometrics

Econometrics is the application of **statistical methods to economic data**, with the goal of giving empirical content to economic relationships. In simple terms: it's how we test whether economic theories actually hold up in the real world, using numbers.

Unlike pure statistics, econometrics is grounded in economic theory — the model structure isn't arbitrary; it's built on hypotheses about how variables relate to each other.

## Why it matters

Economics gives us theories, but theories alone don't tell us *how much* or *how strong* an effect is. Econometrics bridges that gap:

- **Testing theories:** Does increasing the minimum wage actually reduce employment, as some models predict?
- **Forecasting:** What will inflation be next quarter based on current monetary policy?
- **Policy evaluation:** Did the tax cut boost consumption, and by how much?

## The basic workflow

1. **Specify the model** — choose variables based on economic theory (e.g., consumption depends on income and wealth).
2. **Estimate parameters** — use data to fit the model, usually via Ordinary Least Squares (OLS) or similar techniques.
3. **Validate assumptions** — check that the statistical properties required for valid inference actually hold.
4. **Infer and predict** — draw conclusions, test hypotheses, and make forecasts.

## A simple mental model

The classic linear regression setup looks like:

```
Y = β₀ + β₁X₁ + β₂X₂ + ... + ε
```

Where:
- `Y` is the outcome we care about (e.g., wages)
- `X` variables are explanatory factors (education, experience, etc.)
- `β` coefficients tell us the magnitude and direction of each effect
- `ε` is the error term — everything else we didn't measure

## Key assumptions (the "OLS conditions")

For OLS estimates to be the **Best Linear Unbiased Estimators (BLUE)**, several conditions must hold. When they break, things get messy:

- **Linearity** in parameters
- **Random sampling** from the population
- **No perfect collinearity** among independent variables
- **Zero conditional mean** of errors: `E(ε|X) = 0`
- **Homoscedasticity**: constant variance of errors
- **No autocorrelation**: errors are uncorrelated across observations
- **Normality of errors** (for small samples, mainly for hypothesis testing)

Violations are common in practice. The most frequent culprit? **[[Multicollinearity]]**.

## Where it's used

- Macroeconomics: modeling GDP, inflation, interest rates
- Finance: asset pricing, risk models
- Labor economics: wage equations, returns to education
- Program evaluation: causal impact of policies

## See also

- [[Multicollinearity]]

Write by **Samuel**