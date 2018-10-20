+++
title = "Simple Linear Regression with Heteroskedastic Noise"
date = "2017-08-04"
mathjax = true
+++

### Introduction
The model we consider is \\(Y_i = \alpha + \beta x_i + \epsilon_i\\), where \\( \epsilon_i \\) are uncorrelated, and \\( \mathbb{V}(\epsilon_i) \\) depends on \\( i \\).
We discuss two solutions to finding estimators of \\( \alpha, \beta \\).
Weighted least squares regression leads to best linear unbiased estimators (BLUE).
Also, with stronger assumptions on \\( \epsilon_i \\), maximum likelihood estimators (MLE) can be found.
We begin with a discussion of the homoskedastic case with an emphasis on relations between statistical properties of the least squares estimators and assumptions on \\( \epsilon_i \\), which is conducive to understanding the heteroskedastic case.

### Homoskedastic Case
In a simple linear regression, one assumes the model
\\(Y_i = \alpha + \beta x_i + \epsilon_i \\), where \\( \epsilon_i \\), the noise term, satisfies \\( \mathbb{E}(\epsilon_i) = 0 \; \forall i \\).
(Note: We use the upper case for random variables, and lower case for observed values, i.e. \\( x_i \\) is the observed value of \\( X_i \\).)
Although we have not assumed much about the noise, least squares estimates (LSE) of the coefficients, \\( \hat{\alpha}, \hat{\beta} \\)  can be computed by minimizing the sum of squared residuals \\( \sum\_{i=1}^n (y_i - \alpha - \beta x_i)^2 \\); moreover, these estimators are unbiased.
To say more about the LSEs, additional assumptions on \\( \epsilon_i \\) are needed.
If we assume that \\( \mathbb{V}(\epsilon_i) = \sigma^2 \\) and that \\( \epsilon_i \\) are uncorrelated, which are standard assumptions in simple linear regression, then \\( \hat{\alpha}, \hat{\beta} \\) are BLUE.
If, furthermore, we assume that \\( \epsilon_i \\) are independent (which is a condition stronger than uncorrelated) and normally distributed, then we obtain the likelihood function for \\( \alpha, \beta, \sigma^2 \\).
It can be shown that the MLEs equal LSEs.
Therefore, \\( \alpha, \beta \\) have properties of MLEs.
All of discussion so far on on simple linear regression with homoskedastic noise is explained in detail in [CB].

### Heteroskedastic Case
Now, consider heteroskedastic noise.
Assume \\( \mathbb{V}(\epsilon_i) = \sigma_i^2 \\).
Since deriving LSEs does not involve \\( \mathbb{V}(\epsilon_i) \\), the LSEs remain unbiased.
They are, however, no longer BLUE (why?).
To remedy the issue, consider *weighted least squares*,
$$ S_W^2 \equiv \sum\_{i=1}^n W_i (y_i - \alpha - \beta x_i)^2 $$,
set \\( W_i = 1/\sigma_i^2 \\), and minimize \\( S_W^2 \\).
The \\( \hat{\alpha}, \hat{\beta} \\) that minimize \\( S_W^2 \\) are *weighted least squares estimates* (WLSE).
It is known that WLSEs are BLUE.
Intuitively, weighted least squares regression derives good estimators by putting lower weights where variance is high.
To use WLS regression in data analysis, one needs to find \\( \sigma_i^2 \\), which may be difficult.
In general, one can take ordinary least squares estimates
$$ \alpha\_{OLS}, \beta\_{OLS} $$
to estimate the noise by
$$ \epsilon_i \approx y_i - (\alpha\_{OLS} + \beta\_{OLS} x_i) $$,
then estimate \\( \mathbb{V}(\epsilon_i) \\).
See [S].

Another method to estimate the coefficients is maximum likelihood.
The method requires \\( Y_i \\) to be independent, and that the pdfs \\( f(Y_i \vert X_i) \\) are known, so we need to assume independence of \\( \epsilon_i \\) and their distributions.
We assume that \\( \epsilon_i \\) is normally distributed.
Then, it follows that
\\( f(Y_i \vert X_i) \sim \mathcal{N}(\mu_i, \sigma_i^2) \\),
where \\( \mu_i \equiv \alpha + \beta x_i \\).
Given data \\( (x_1, y_1), \ldots, (x_n, y_n) \\), we have
$$
L_n(\alpha, \beta, \sigma_i^2 \vert X=x, Y=y)
= \prod\_{i=1}^n f(x_i, y_i)
= \prod\_{i=1}^n f(y_i \vert x_i) \cdot f(x_i).
$$
Since \\( f(x_i) \\) does not depend on the parameters, we have
$$
L_n \propto \prod\_{i=1}^n f(y_i \vert x_i)
= \prod\_{i=1}^n \frac{1}{\sqrt{2\pi\sigma_i^2}} \exp\left( - \frac{(y_i - \mu_i)^2}{2\sigma_i^2} \right)
\propto \prod\_{i=1}^n \frac{1}{\sqrt{\sigma_i^2}} \exp\left( - \frac{(y_i - \mu_i)^2}{2\sigma_i^2} \right).
$$
Therefore, the log-likelihood is
$$
l_n(\alpha, \beta, \sigma_i^2 \vert x, y) = - \sum\_{i=1}^n \frac{\log \sigma_i^2}{2}
        + \frac{(y_i - \mu_i)^2}{2\sigma_i^2}.
$$
We would like to maximize \\( l_n \\), but to do so, \\( \sigma_i^2 \\) have to be expressed in terms of the data \\( (x,y) \\) and a few parameters.
Suppose, for example, that \\( \mathbb{V}(\epsilon_i) = \sigma_i^2 = \gamma^2 x^2 \\).
Then,
$$
l_n(\alpha, \beta, \gamma \vert x, y) = - \sum\_{i=1}^n \log \lvert {\gamma x_i}\rvert
        + \frac{(y_i - \mu_i)^2}{2\gamma^2 x_i^2},
$$
which is amenable to maximization.

### Summary

- If \\( \mathbb{V}(\epsilon_i) \\) are known and \\( \epsilon_i \\) are uncorrelated, then WLSEs are BLUE.
- If \\( \epsilon_i \\) are independent and their distributions are known, one can define the likelihood function.
To maximize the likelihood function, \\( \mathbb{V}(\epsilon_i) \\) needs to be expressed in terms of \\( (x,y) \\) and some parameters.

Question: do MLEs equal WLSEs in simple linear regression with heteroskedastic noise?

### Reference
- [CB] Casella and Berger, *Statistical Inference*, Chapter 11
- [S] Shalizi, C, *Advanced Data Analysis from an Elementary Point of View*, Chapter 6
