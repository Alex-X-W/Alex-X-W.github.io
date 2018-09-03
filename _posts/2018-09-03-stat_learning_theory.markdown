---
layout:     post
title:      "ML notes: Statistical learning theory"
date:       2018-09-03
author:     "Xuan"
header-img: "img/in-post/7/header.jpeg"
header-mask: 0.3
catalog:    true
mathjax:    true
multilingual: false
tags:
    - machine learning
---

$\newcommand{\mc}{\mathcal} \newcommand{\mb}{\mathbb}$

## Learning theory

### Three spaces

- $\mathcal{X}$ : Input space
- $\mathcal{A}$ : Action space
- $ \mathcal{Y}$ : Outcome space

**Note**

1. Outcome $y$ is often independent of action $a$
2. But this is not always the case, e.g.
   - search result ranking
   - automated driving
   - other reinforcement learning situations



### Bayes risk

**Prediction function**:

$f: \mathcal{X} \rightarrow \mathcal{A}, \; x \mapsto f(x)$

**Loss function**:

$l: \mathcal A \times \mathcal Y \rightarrow \mathbf R, \;(a, y) \mapsto l(a, y)$

**Data generating distribution**

We assume there is distribution $P_{\mc{X} \times \mc{Y}}$ from which all data points are drawn **i.i.d.**

**Risk as the expected loss**

$R(f) = \mathbb E_{P_{\mc{X} \times \mc{Y}}} l(f(x), y)$

**Bayes Risk**

Bayes risk is the minimal risk over all possible prediction functions $\mathcal F = \\{ \mc X \rightarrow \mc Y \\}$.

The function achieves the optimal is called the **Bayes Prediction Functoin**, often aka the **target function**
$$
f^\ast \in \arg\min_{f\in \mathcal F} R(f) \tag{1}
$$

#### Examples

**Least Squares Regression**

- Spaces: $\mc{A} = \mc{Y} = \mc{R}$
- Loss: $l(a, y) = (a-y)^2$

Risk  
$$
\begin{align*}
R(f) &= \mathbb E [(f(x) - y)^2] \\ &= \mb{E} [(f(x) - \mb{E}[y\vert x])^2] + \mb{E} [(y - \mb{E}[y\vert x])^2]
\end{align*}
$$

So the Bayes prediction function is:  
$$
f^\ast(x) = \mb{E}[y\vert x]
$$


### Empirical Risk

Since we don't know $P_{\mc{X} \times \mc{Y}}$, we can not compute the risk, neither the bayes risk. But we can use empircal risk to approximate.

**Empirical risk**

$\hat {R}_n (f) = \frac{1}{n} \sum_i l(f(x_i), y_i)$

By the Strong law of large numbers, $\lim_{n \rightarrow \infty} \hat R_n(f) = R(f)$ almost surely (i.e. with probability 1).

**Empirical risk minimizer**

A function $\hat f$ is an empirical risk minimizer if $\hat f \in \arg \min_{f\in \mc{F}} \hat R_n(f)$.

#### Remark

Directly applying the ERM will lead to a function $f$ which just memorizes the data. A usual approach is to add constraints to ERM, e.g. constrain the minimization over a particular subset, called a **hypothesis space** $\mc{H}$, rather than the whole $\mc{F}$.

And the ERM becomes
$$
\hat f \in \arg \min_{f\in \mc{H}} \hat R_n(f)
$$


## Bias and Variance

### Basic notions

A **parameter** is some characteristic of a probability distribution $P$, formalized as $\mu = \mu(P)$. Examples include expected value, variance, median, etc...

We usually assume our data $\mc{D}_n = (x_1, ..., x_n)$ is an i.i.d. sample from $P$.

A **statistic** $s = s(\mc{D}_n)$ is any function of the data.

A statistic $\hat \mu$ is a **point estimator** of a parameter $\mu$ if $\hat \mu \approx \mu$.

The distribution of a statistic is called a **sampling distribution**.

The standard deviation of the sampling distribution is called the **standard error**.

### Bias and Variance for real valued estimators

Let $\mu : P \mapsto \mathbf R$ be a real valued parameter, and

let $\hat \mu : \mc{D}_n \mapsto \mathbf R$ be an estimator of $\mu$, we define

**bias** to be $\text {Bias}(\hat \mu) = \mb{E} \hat \mu - \mu$, and

**variance** to be $\text{Var}(\hat \mu) = \mb{E} \hat \mu^2 - (\mb{E} \hat \mu)^2$.

Note that the above two expectations are w.r.t. the distribution of $\mc{D}_n$.

#### Remark

We often use **bootstrap** to estimate the parameters of an estimator, e.g. the bias and variance.

