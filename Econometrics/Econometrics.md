---
tags: [econometrics, statistics, regression, mcrl]
---

# Econometrics

Econometrics is the branch of economics that uses statistical methods, mostly regression analysis, to estimate relationships between economic variables, test theories, and make predictions. In practice, most of applied econometrics starts from one model: the **Classical Linear Regression Model (CLRM)**, estimated by **Ordinary Least Squares (OLS / MQO)**.

## The Classical Linear Regression Model (CLRM)

A regression model relates a dependent variable $Y$ to one or more explanatory variables $X$:

$$Y_i = \beta_0 + \beta_1 X_{1i} + \beta_2 X_{2i} + \dots + \beta_k X_{ki} + u_i$$

$u_i$ is the error term, everything that affects $Y$ but isn't in the model. OLS picks the $\hat\beta$ that minimizes the sum of squared residuals.

## Gauss-Markov assumptions

For OLS to be **BLUE** (Best Linear Unbiased Estimator, unbiased *and* minimum variance among linear unbiased estimators), the CLRM assumes:

1. Linearity in parameters.
2. Strict exogeneity, $E(u_i \mid X) = 0$.
3. No perfect multicollinearity among regressors.
4. Homoscedasticity, $\text{Var}(u_i \mid X) = \sigma^2$ (constant).
5. No autocorrelation, $\text{Cov}(u_i, u_j) = 0$ for $i \neq j$.
6. (For inference) $u_i \sim N(0, \sigma^2)$.

Each of the three "problem" notes below corresponds to one of these assumptions breaking:

- [[Multicollinearity]] — assumption 3 (not a Gauss-Markov assumption itself, but near-violation of it) degrades precision even though OLS stays BLUE.
- [[Heteroscedasticity]] — assumption 4 breaks. OLS stays unbiased but loses efficiency (no longer BLUE), and standard errors become invalid.
- [[Autocorrelation]] — assumption 5 breaks. Same consequence as heteroscedasticity: unbiased but inefficient, standard errors invalid, typically inflated t-stats when $\rho>0$.

The common thread: none of these three problems bias the coefficients themselves. What they break is **efficiency** (variance is no longer minimal) and/or the **validity of standard errors**, which is what makes hypothesis testing (t-tests, F-tests, confidence intervals) unreliable if ignored.

## Beyond continuous Y: qualitative response models

The CLRM assumes a continuous dependent variable. When $Y$ is categorical, most commonly **binary** (0/1), a different toolkit is needed:

- [[Dummy Variables]] — how to bring *qualitative explanatory* variables (sex, region, category...) into a regression.
- [[Linear Probability, Logit and Probit]] — how to model a *qualitative dependent* variable (binary outcome), and why plain OLS (the Linear Probability Model) isn't the right tool for that.

## Worked examples from our own work

- Aggregate consumption model for Brazil (BCB SGS data, monthly, 2012–2022): used to walk through heteroscedasticity, autocorrelation and multicollinearity diagnostics end to end. See [[Heteroscedasticity]] and [[Autocorrelation]] for the actual test results from that model.
- Labor force participation model (Mroz, 1987 dataset, 753 married women): used to compare Linear Probability Model, Logit and Probit side by side. See [[Linear Probability, Logit and Probit]] for the full walkthrough with real coefficients and marginal effects.

## Reference

GUJARATI, Damodar N. *Econometria Básica*. 5th ed. Porto Alegre: AMGH, 2011. — main reference for all notes in this folder (chapters cited individually in each note).

## See also

- [[Multicollinearity]]
- [[Heteroscedasticity]]
- [[Autocorrelation]]
- [[Dummy Variables]]
- [[Linear Probability, Logit and Probit]]

Write by **Samuel**
