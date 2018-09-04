---
layout:     post
title:      "ML notes: Kernel methods and SVM"
date:       2018-09-04
author:     "Xuan"
header-img: "img/in-post/8/header.jpeg"
header-mask: 0.3
catalog:    true
mathjax:    true
multilingual: false
tags:
    - machine learning
---

$\newcommand{\ip}[2]{\langle #1, #2 \rangle}​$

## The Representer Theorum

For a general problem
$$
\min_w (f(\langle w, \psi(x_1) \rangle, …, \langle w, \psi(x_m) \rangle) + R(||w||)) \tag{1}
$$
where $f: \mathbb{R}^m \rightarrow \mathbb{R}​$ is usually the cost, and $R: \mathbb{R}_+ \rightarrow \mathbb{R}​$ is usually a regularization which is monotonically nondecreasing. Assume $\psi​$ is a mapping from $\mathcal{X}​$ to a Hilbert space, then there exists a vector $\alpha \in \mathbb{R}^m​$ such that $w = \sum_i \alpha_i \psi(x_i)​$ is an optimal solution.

### Corallary

Subsituting $w = \sum_i \alpha_i \psi(x_i)$ into (1), we can rewrite

$\langle w, \psi(x_i) \rangle = \langle\sum_i \alpha_i \psi(x_i), \psi(x_j) \rangle = \sum_i \alpha_i \langle \psi(x_i), \psi(x_j) \rangle$

and

$$||w||^2 =\langle \sum_i \alpha_i \psi(x_i), \sum_i \alpha_i \psi(x_i) \rangle = \sum_i \alpha_i \alpha_j \langle \psi(x_i), \psi(x_j) \rangle$$

Let $K(x, x') = \ip{\psi(x)}{\psi(x')}$, we can rewrite (1) into
$$
\min_{\alpha \in \mathbb{R}^m} f(\sum_i \alpha_i K(x_i, x_1), …, \sum_i \alpha_i K(x_i, x_m)) + R(\sqrt{\sum_{i, j} K(x_i, x_j)}) \tag{2}
$$
Thus we only need to compute the inner product in the embedding space $\psi(\mathcal X)$.



## SVM

### Hard SVM

Assume the dataset (in the embedding space) is linearly separable, which is a rather strong assumption (it depends on which embedding we are using but we did not have any gaurantee in the first place).
$$
\arg \max_{w, b} \frac{1}{||w||} \min_i (y_i (w^T \psi(x_i) + b)) \tag{3}
$$
By leveraging the freedom of rescaling we can constrain $y_i(w^T \psi(x_i)+b) \ge 1, \forall i = 1, … N$, thus turning (3) into

**(Primal)**
$$
\begin{align*}
\arg \min_{w, b} & \ \ \frac{1}{2}||w||^2 \tag{4}\\
\text{subject to} & \ \ y_i (w^T \psi(x_i) + b) \ge 1, \forall i  =1, ..., N
\end{align*}
$$
**(Dual)**
$$
\begin{align*}
\arg \max_{a} & \ \ \sum_i a_i - \frac{1}{2} \sum_i \sum_j a_i a_j y_i y_j K(x_i, x_j)\tag{5}\\
\text{subject to} & \ \ a_i \ge 0, \forall i  =1, ..., N \\
			  & \ \ \sum_i a_i y_i = 0
\end{align*}
$$

#### Remark

1)

The decision boundary, or, the seperating hyperplane $y(x) = w^T \psi(x) + b$ can be reformulated, by using $w = \sum_i a_i y_i \psi(x_i)$, into
$$
y(x) = \sum_i a_i y_i K(x_i, x) + b \tag{6}
$$
2)

Because the problem (3) achieves zero duality gap, by complementarity we have

$a_i (1 - y_i (w^T \psi(x_i) + b)) = 0, \forall i$

thus only a small portion (denoted as a set $\mathcal S$) of the data points will have $a_i \neq 0$ and $y_i (w^T \psi(x_i) + b) = 1$, these data points are called the **support vectors**.

1) + 2) =>

So in inference, we can just retain those support vectors in (6) as other data points whose $a_i = 0$ play no role in (6).

### Soft SVM

A relaxation to the hard SVM that it does not require a linear separable dataset, by using the **hinge loss**. An alternative motivation is from allowing the constraints in (4) to be violated by a slack variable $\xi_i$.

**(Primal)**
$$
\begin{align}
\arg \min_{w, b, \xi} &\ \  (\frac{1}{2}||w||^2 + \lambda \sum_i \xi_i) \tag{7} \\
\text{subject to} &\ \ y_i(w^T\psi(x_i) + b) \ge 1- \xi_i \\
			  & \ \ \xi_i \ge 0
\end{align}
$$
**(Dual)**
$$
\begin{align}
\arg \max_{a} & \ \ \sum_i a_i - \frac{1}{2} \sum_i \sum_j a_i a_j y_i y_j K(x_i, x_j)\tag{5}\\
\text{subject to} & \ \ 0 \le a_i \le \lambda, \forall i  =1, ..., N \\
			  & \ \ \sum_i a_i y_i = 0
\end{align}
$$
