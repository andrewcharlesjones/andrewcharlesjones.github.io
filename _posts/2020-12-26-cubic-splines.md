---
layout: post
title: "Cubic splines"
author: "Andy Jones"
categories: journal
blurb: "Cubic splines are flexible nonparametric models. Here, we discuss some of the spline fundamentals."
img: ""
tags: []
<!-- image: -->
---

Cubic splines are flexible nonparametric models. Here, we discuss some of the spline fundamentals.

## Introduction

Consider the following regression problem:

$$Y = \Phi(X)\beta + \epsilon$$

where $Y \in \mathbb{R}^n$, $X \in \mathbb{R}^{n}$, and $\Phi(X)$ denotes a basis expansion of $X$. Here, we work with a one-dimensional regression for simplicity, although all ideas will easily extend to multiple dimensions.

In vanilla linear regression, we have $\Phi(X) = X$, that is, there is no basis expansion.

In polynomial regression of order $d$, the basis expansion for the $i$th sample is

$$\phi(x_i) = \begin{bmatrix} 1 & x_i & x_i^2 & x_i^3 & \cdots & x_i^d \end{bmatrix}.$$

The coefficient vector $\beta$ then becomes a vector of length $d+1$, and we can still apply the OLS estimator:

$$\widehat{\beta} = (\Phi(X)^\top \Phi(X))^{-1} \Phi(X)^\top Y.$$

## Cubic splines

Cubic splines generalize cubic regression to allow for modeling local regions of the input space differently. Specifically, the approach is to choose $k$ thresholds (often called "knots") $\tau_1, \dots, \tau_k$ and split up the input space into $k+1$ intervals based on these thresholds:

$$(-\infty, \tau_1], (\tau_1, \tau_2], (\tau_2, \tau_3], \cdots, (\tau_k, \infty).$$

Within each of these intervals, we fit a local (cubic) polynomial subject to constraints of continuity and smoothness between intervals.

The basis expansion then becomes

$$\phi(x_i) = \begin{bmatrix} 1 & x_i & x_i^2 & x_i^3 & (x_i - \tau_1)_+^3 & (x_i - \tau_2)_+^3 & \cdots & (x_i - \tau_k)_+^3 \end{bmatrix}.$$

## Smoothing cubic splines

In general, choosing the locations of the knots is a difficult problem. Smoothing splines avoid having to make these choices by placing a knot on each data point, i.e., $k=n$ and $\tau_1=x_1, \tau_2=x_2, \dots, \tau_n=x_n$.

$$\phi(x_i) = \begin{bmatrix}1 & ~x_i & x_i^2 & x_i^3 & (x_i-x_1)^3 & \cdots & (x_i-x_n)^3\end{bmatrix}.$$

"Natural" cubic splines make one more assumption: that the function is linear in the extremes of the input space (below the minimum and above the maximum). As a natural cubic spline, we can write the basis expansion in terms of $n$ basis functions. The design matrix then becomes an $n \times p$ matrix. Let's call this $\Phi(X)$.

It can be shown (see below), that natural cubic splines are the functions that minimize a regularized sum of squares penalty:

\begin{equation} \text{arg}\min_f \sum\limits_{i=1}^n (y_i - f(x_i))^2 + \lambda \int_a^b [f^{\prime\prime}(x)]^2 dx \label{eq:spline_objective} \end{equation}

Notice that the second term penalizes the average second derivative of the function ($\lambda$ is a tuning parameter here). In other words, this objective function favors smooth functions. Clearly, this penalty will be zero when the function is linear. Equivalently, the optimal $f$ will be linear when $\lambda=\infty$.

Since the optimal function will be a cubic smoothing spline, we can rewrite the objective as

$$\text{arg}\min_\beta \left[\sum\limits_{i=1}^n (y_i - \Phi(x_i)\beta)^2 + \lambda \sum\limits_{j=1}^n \int_a^b \beta^\top \Phi^{\prime\prime}(x_i) \Phi^{\prime\prime}(x_j) \beta dx\right].$$

Letting $\Omega$ be a $n\times n$ matrix with $\Omega_{ij} = \int_a^b   \Phi^{\prime\prime}(x_i) \Phi^{\prime\prime}(x_j) dx$, this simplifies to

$$\text{arg}\min_\beta \sum\limits_{i=1}^n (y_i - \Phi(x_i)\beta)^2 + \lambda \beta^\top \Omega \beta.$$

In this form, we can recognize it as a ridge regression problem, and the coefficient estimate will then be

$$\widehat{\beta} = (\Phi(X)^\top \Phi(X) + \lambda \Omega)^{-1} \Phi(X)^\top Y.$$

## Cubic splines as minimizer

In this section, we show that the natural cubic spline minimizes the objective in \eqref{eq:spline_objective}.

We want to show that the function that minimizes the following objective is a natural cubic spline.

$$\mathcal{L} = \sum\limits_{i=1}^n (y_i - f(x_i))^2 + \lambda \int_a^b \left[f^{\prime\prime}(x)\right]^2 dx.$$

Let $f(x)$ be a cubic spline, and let $g(x)$ be any other function that interpolates the points. Let $h(x)=g(x)-f(x)$ be their difference. 

For both of these functions, the first term will be zero because they perfectly interpolate the data.

Focusing on the second term, our goal reduces to showing that

$$\int_a^b \left[f^{\prime\prime}(x)\right]^2 dx \leq \int_a^b \left[g^{\prime\prime}(x)\right]^2 dx, ~~~ \forall g.$$

Notice that by rewriting $g(x) = h(x) + f(x)$ and using the linearity of derivatives and integrals, the right side is equal to

\begin{align} \int_a^b [h^{\prime\prime}(x) + f^{\prime\prime}(x)]^2 dx &= \int_a^b [h^{\prime\prime}(x)^2 + 2h^{\prime\prime}(x) f^{\prime\prime}(x) + f^{\prime\prime}(x)^2] dx \\\ &= \int_a^b [h^{\prime\prime}(x)]^2 dx + 2 \int_a^b h^{\prime\prime}(x) f^{\prime\prime}(x) dx + \int_a^b [f^{\prime\prime}(x)]^2 dx. \end{align}

Notice that $\int_a^b h^{\prime\prime}(x)^2 dx \geq 0$. Thus, we just need to show that $\int_a^b h^{\prime\prime}(x) f^{\prime\prime}(x) dx \geq 0$. Using integration by parts, where $u=f^{\prime\prime}(x)$ and $dv=h^{\prime\prime}(x)$, we have

$$\int_a^b h^{\prime\prime}(x) f^{\prime\prime}(x) dx = f^{\prime\prime}(x) h^\prime(x) \big\rvert_a^b - \int_a^b h^\prime(x) f^{\prime\prime\prime}(x) dx.$$

A natural cubic spline is linear on its endpoints, which implies that $f^{\prime\prime}(a) = f^{\prime\prime}(b) = 0$. Thus, the first term is zero.

For the second term, notice that we can break up the integral into each of the $n-1$ intervals between the points.

\begin{align} \int_a^b h^\prime(x) f^{\prime\prime\prime}(x) dx &= \int_{x_1}^{x_2} h^\prime(x) f^{\prime\prime\prime}(x) dx + \int_{x_2}^{x_3} h^\prime(x) f^{\prime\prime\prime}(x) dx + \cdots + \int_{x_{n-1}}^{x_n} h^\prime(x) f^{\prime\prime\prime}(x) dx \\\ &= \sum\limits_{i=1}^{n-1} \int_{x_{i}}^{x_{i+1}} h^\prime(x) f^{\prime\prime\prime}(x) dx. \end{align}

If we again integrate by parts with $u=f^{\prime\prime\prime}(x)$ and $dv=h^\prime(x)$, we have

$$\sum\limits_{i=1}^{n-1} \left[f^{\prime\prime\prime}(x) h(x)\big\rvert_{x_i}^{x_{i+1}} - \int_{x_{i}}^{x_{i+1}} h^{\prime\prime}(x) f^{\prime\prime\prime\prime}(x) dx\right]. $$

Since both $f$ and $g$ perfectly interpolate the data, the first term will be zero. Furthermore, since $f$ is a cubic polynomial, its fourth derivative will be zero, making the second term zero.

Thus, we have proved that the penalty for any other function will be at least as great as that of a natural cubic spline:

$$\int_a^b \left[f^{\prime\prime}(x)\right]^2 dx \leq \int_a^b \left[g^{\prime\prime}(x)\right]^2 dx, ~~~ \forall g.$$

Returning to the initial loss function, we can see that this implies that the overall loss function for any other function will be greater than a natural cubic spline (again, since we have enough degrees of freedom to interpolate the data, so the first term will always be zero):

$$\mathcal{L}(f) \leq \mathcal{L}(g).$$

## References

- Friedman, Jerome, Trevor Hastie, and Robert Tibshirani. The elements of statistical learning. Vol. 1. No. 10. New York: Springer series in statistics, 2001.
- Sergey Fomel's [notes on splines](http://sepwww.stanford.edu/sep/sergey/128A/answers6.pdf).
