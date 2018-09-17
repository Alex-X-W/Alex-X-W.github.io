---
layout:     post
title:      "ML notes: Latent variable models"
date:       2018-09-04
author:     "Xuan"
header-img: "img/in-post/9/header.jpeg"
header-mask: 0.3
catalog:    true
mathjax:    true
multilingual: false
tags:
    - machine learning
---

## General latent variable model

**Learning problem**: 

Given incomplete dataset $\\{ x_i\\}$, estimate $\theta^\ast$, for instance by MLE: $\hat \theta = \arg \max_\theta p(x;\theta)$

**Inference problem**: 

Given $x$, find conditional distribution $p(z \vert x; \theta^\ast)$



$p(x; \theta)$ is often termed as **marginal likelihood** since it is $p(x, z; \theta)$ with $z$ marginalized out. And $p(x, z; \theta)$ is called the **joint**.



## EM

### Setting

- Want to find $\theta^\ast$ by MLE:

  $\hat \theta = \arg \max_\theta p(x;\theta) = \arg \max_\theta \log p(x; \theta)$

  which can be hard (e.g. hard to get gradient in GMM if use gradient descent).

- Maximizing the joint likelihood is easy:

  $\max_\theta \log p(x, z; \theta)$

  which is also to say, maximizing the posterior likelihood is easy:

  $\max_\theta \log p(z \vert x ; \theta)$



### Approach

Create a family $\mathcal Q$ of lower bounds on $\log p(x; \theta)$:

$\log p(x ; \theta) \ge L_q(\theta), \forall \theta \in \Theta$

And we will try to find the maximum over all lower bounds:
$$
\hat \theta = \arg \max_\theta [\sup_q L_q(\theta)] \tag{1}
$$
We can take the $q$ to be an **approximate prior** on $z$, and we construct $L_q(\theta)$ to be the **ELBO**:
$$
\begin{align} 
\log p(x; \theta)
& = \sum_z q(z)\log \frac{p(x, z; \theta)q(z)}{q(z)p(z \vert x;\theta)}\\
& = \underbrace{\sum_z q(z) \log \frac{p(x, z; \theta)}{q(z)}}_{L_q(\theta),\ \text{the ELBO}}+\underbrace{\sum_z q(z)\log\frac{q(z)}{p(z\vert x; \theta)}}_{\text{KL}(q(z)\Vert p(z\vert x; \theta))\ge0}\\
&\ge \text{ELBO}
\end{align} \tag{2}
$$
Hence by using the ELBO as the family of lower bounds (indexed by $q$), we know that in (1):

$\sup_q L_q(\theta) = \log p(x; \theta)$, the suprema are attained at $q(z) = p(z \vert x; \theta), \forall \theta$.



### Algorithm

In EM, we iteratively maximize the ELBO over either $q$ or $\theta$ by holding the other fixed.

- $\arg \max_q L_q(\theta)$ for fixed $\theta$ - **E-step**

  according to (2), the maximum is attained at $q(z) = p(z \vert x; \theta)$. So to use EM, we are given that the posterior is tractable.

- $\arg \max_\theta L_q(\theta)$ for fixed $q$ - **M-step**

  since $L_q(\theta) = \underbrace{\sum_z q(z) \log p(x, z; \theta)}_{\mathbb E [\text{complete data log-likelihood}]} - \underbrace{\sum_z q(z) \log q(z)}_{\text{no}\ \theta \ \text{involved}}$

  so only the first term, the expectation of the complete data log-likelihood, is relevant to maximization. So finding the $q$ in the other step is essentially used to compute the expectation, hence the name "E-step".

  **Note**

  In the M-step, $\arg \max_\theta L_q(\theta)$ can be hard, we can relax it by just finding a $\theta^{(t+1)}$ such that $L_q(\theta^{(t+1)}) \ge L_q(\theta^{(t)})$, which still guarantees to converge (to a local minimum). And now in the E-step, we will actually just use $q^\ast(z)$ to compute $\sum_z q(z) \log p(x, z; \theta)$, and then in the M-step, say, take some gradient steps w.r.t. $\theta$.


So in implementation the EM looks like:

- E-step

  Compute: $J(\theta) := \sum_z q^\ast(z) \log p(x, z ; \theta)$

- M-step

  Maximize: $\arg \max_\theta J(\theta)$



### Remark

- Note that in EM, we only care about optimizing $\theta$, and we may not easily generate samples after we have done EM (because EM does not return a prior $p(z)$ from which we can easily sample from).

- In VAE, we assume the prior is an easy distribution which we can easily sample from, e.g. Gaussian or Uniform

  Sampling: (1) draw $z \sim p(z)$; (2) draw $x \sim p(x \vert z; \theta)$

  Learning: given $\mathcal D = \\{ x \\}$, learn a $q(z \vert x; \phi) \approx p(z\vert x; \theta)$ to draw $z$, and then use $p(x \vert z; \theta)$ to draw $x$



### EM in Bayesian setting

Now we have a prior $p(\theta)$.

We want to find MAP estimate: $\hat \theta = \arg \max_\theta \log p(\theta \vert x)$.

Since $\log p(\theta \vert x) = \log p(x \vert \theta) + \log p(\theta) - \log p(x)​$, we can still use $L_q(\theta)​$ as a lower bound family for $\log p(x \vert \theta)​$, and thus the EM looks like:

- E-step

  The same, compute $J(\theta) := \sum_z q^\ast(z) \log p(x, z ; \theta)$

- M-step

  Maximize: $\arg \max_\theta [J(\theta) + \log p(\theta)]$