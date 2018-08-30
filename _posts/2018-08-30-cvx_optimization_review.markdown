---
layout:     post
title:      "Convex optimization review"
subtitle:   "just to connect some dots"
date:       2018-08-30
author:     "Xuan"
header-img: "img/in-post/5/header.jpeg"
header-mask: 0.3
catalog:    true
mathjax:    true
multilingual: false
tags:
    - optimization
---

*TODO: 1) Convergence analysis 2) interior point primal dual method*

---

## Convexity

#### Separating hyperplane theorum

If two convex sets, $A​$ and $B​$ are disjoint, then there exists a hyperplane separates the two, i.e., there exist $a​$ and $b​$ such that $a^T x\le b​$ for all $x​$ in $A​$ and $a^T x \ge b​$ for all $x​$ in $B​$.

#### Supporting hyperplane theorum

For every point $x_0$ in the boundary of a convex set $C$, denoted as $\textbf{bd}\,C = \textbf{cl}\, C \,\backslash \textbf{int}\, C$, there exists a supporting hyperplane, i.e. there exists an $a$ such that $a^T x \le a^T x_0$ for all $x \in C$.

---

## Duality

#### The Lagrange dual function

$g(\lambda, \nu) = \inf_{x\in \mathcal{D}} (f_0(x) + \lambda ^T f(x) + \nu ^T h(x))$

- It is a **pointwise infimum** of a family of affine functions of $(\lambda, \nu)$, hence it is concave.

- For every $(\lambda, \nu)$ pair that is **dual feasible**, i.e. $\lambda \ge 0$ and $(\lambda, \nu) \in \textbf{dom} \, g$ (which usually translates to $g(\lambda, \nu) > - \infty$ since $g$ is a mapping to $\mathbf{R}$), $g(\lambda, \nu)$ is a lower bound for the primal optimum. 

  Proof: for every primal feasible point $\tilde{x}$, $g(\lambda, \nu) \le f_0(\tilde x) + \lambda ^T f(\tilde x) + \nu ^T h(\tilde x) \le f_0(\tilde x) \le p^\ast$

- $g(\lambda, \nu)​$ constrained on dual feasible points can be viewed as a set of lower bounds of $p^\ast​$

- **Saddle point interpretation**: $p^\ast = \inf_x \sup_{\lambda, \nu} L(x, \lambda, \nu) \ge \sup_{\lambda, \nu} \inf_x L(x, \lambda, \nu) = d^\ast$.

#### Slater's condition and strong duality

If there is a **strictly feasible** point, meaning one such point that all the inequality constraints hold strictly, then strong duality holds. 

- The condition can be reduced to weaker form on affine inequility constraints that they can hold with equalities.
- If a problem is **self-dual**, meaning the dual of the dual problem is the primal, e.g. SDP, then either of the primal-dual problem pair holds the Slater condition will gurantee strong duality.

Sketch of proof:

Consider $\mathcal A = \\{ (f(x)^T, h(x)^T, f_0(x))\, \vert \, x \in \mathcal D\\} + \mathbf R_+^m \times \\{0\\}  \times \mathbf R_+$ 

and $\mathcal B = \\{(u, v, t)\, \vert \, u=0, v = 0, t \lt p^\ast \\}$.

Then apply separating hyperplane theorum. The strictly feasible point is used to ensure that the hyperplane is non-vertical thus won't pass thru $\mathcal B$.

#### Duality gap

For a primal dual feasible point pair $x$ and $(\lambda, \nu)$, $f_0(x)-g(\lambda, \nu)$ is refered to as the **duality gap**, which also gives a certificate of how close the primal value is to its optimum.

#### Complementarity

*Assume strong duality holds*, then we must have 

$f_0(x^\ast) = g(\lambda^\ast, \nu^\ast) \le f_0(x^\ast) + {\lambda^\ast}^T f(x^\ast) + {\nu^\ast} ^T h(x^\ast) \le f_0(x^\ast)$

which implies that ${\lambda^\ast}^Tf(x^\ast) = 0$, so ${\lambda^\ast}$ and $f(x^\ast)$ are complementary.

#### KKT conditions:

Assumes zero duality gap.

**KKT**:

- **primal feasibility**

  1) $f(x^\ast) \le 0$; 2) $h(x^\ast)=0$

- **dual feasibility**

  1) $\lambda^\ast \ge 0$; 2) $\nabla_xL(x^\ast, \lambda^\ast, \nu^\ast)=0$

- **complementarity (zero duality gap)**

  $\lambda^\ast f(x^\ast) = 0$


i) If the primal problem is convex, then KKT conditions are **equivalent** for the points to be primal and dual optimal.

ii) If the primal problem is nonconvex, the KKT conditions are **necessary**.

Remark for dual feasibility: 

$\nabla_xL(x^\ast, \lambda^\ast, \nu^\ast)=0$ means $x^\ast$ minimizes $L$, then $g(\lambda^\ast, \nu^\ast)>-\infty$ and hence dual feasible.

---

## Conjugacy

#### Conjugate function

The conjugate of a function $f: \mathbf{R}^n \rightarrow \mathbf{R}$, $f^\ast: \mathbf{R}^n \rightarrow \mathbf{R}$, is defined as 

$f^\ast(y) = \sup_{x\in\textbf{dom}\,f} (y^Tx-f(x)) = - \inf_x (f(x) - y^Tx)$

but whose domain constrained on $y\in \mathbf{R}^n$ for which the supremum is finite. 

- $f^\ast$ is a pointwise supremum of a family of affine functions, hence convex.
- **Fenchel-Young's inequality**: $f(x) + f^\ast(y) \ge x^Ty$.
- if $f$ is convex and closed, the $f^{\ast\ast}=f$.

#### Rewrite Lagrange dual function

For a general linear constraint problem 

$\min f_0(x): Ax \le b, Cx=d$

using conjugate we can rewrite its Lagrange dual function as

$g(\lambda, \nu) =  \inf_x(f_0(x) + \lambda^T(Ax-b)+\nu^T(Cx-d)) = -\lambda^Tb-\nu^Td - f_0^\ast(-A^T\lambda-C^T\nu)$.

---

## Unconstrained algorithms: descent methods

#### The general descent algorithm

- **Determine a direction**

  Look for a $\Delta x$ with $\nabla f(x^{(k)})^T \Delta x \lt 0$


- **Line search**

  Find $t$ to update $x^{(k+1)} \leftarrow x^{(k)} + t \Delta x$.

- **Check stopping criterion**


#### Line search methods

1. **Exact line search**

   $t \leftarrow \text{argmin}_{s \ge 0} f(x + s \Delta x)$

2. **Backtrack line search**

   while $f(x + t \Delta x) \gt f(x) + \alpha t \nabla f(x)^T \Delta x$, do $t \leftarrow \beta t$

#### Direction choice methods

1. **Gradient descent**

   $\Delta x = -\nabla f(x)$

2. **Newton's method**

   $\Delta x = -\nabla^2f(x)^{-1}\nabla f(x)$

---

## Equality constrained algorithms

*We are primarily converning linear equality constraints, since linear form is the only possibility of an equality constraint to be convex.*

For $\min f(x): Ax=b$ , the KKT conditions become:

1. dual feasibility: $\nabla f(x^\ast)+A^T\nu^\ast=0$
2. primal feasibility: $Ax^\ast = b$

#### Elimination of equality constraints

For a problem 

$\min f(x): Ax=b$ 

we can find a specific solution to the constraint, $x_0$, and a matrix $F$ such that $\text{Range}(F) = \text{Null}(A)$. Thus we convert the problem into an unconstrained problem

$\min_z f(Fz+x_0)$.

#### Dual method: ADMM

*Alternating Direction Method of Multipliers*

- **Motivation**

  - **Dual ascend**

    For a *linear equality constraint* (hence strong duality holds) problem $\min f(x): Ax=b$, solve the dual problem (which is *unconstrained*) using gradient ascend.

    The gradient of the dual objective at $y^k$ is $\nabla g(y^k) = Ax^{k+1} - b$ , where $x^{k+1} =\text{argmin}_xL(x, y^k)$.

    The algorithm:

    - $x^{k+1} \leftarrow \text{argmin}_xL(x, y^k)$
    - $y^{k+1} \leftarrow y^k + \alpha (Ax^{k+1} - b)$

  - **Dual decomposition**

    If the primal objective $f$ is separable in $x$, then $x^k =\arg \min_xL(x, y^k)$ step can be decomposed into $x_i^k =\arg\min_{x_i} L_i(x_i, y^k)$. Note that all the $x_i^k$ are evaluated in parallel holding other components constant as from $x^k$.

  - **Augmented Lagrangian and the method of multipliers**

    Still for the same linear equality constrained problem, one injects into the Lagrangian a penalty term for the residual of the equality constraint

    $L_\rho(x, y) = L(x, y) + (\rho/2) \Vert Ax-b\Vert^2$

    The algorithm is similar to dual ascend but set $\alpha=\rho$.

- **ADMM**

  Setting: primal objective decomposable. Augment the Lagrangian and use dual decomposition but update the components sequentially, taking latest values of other components.

#### Primal method: extension of Newton's method

##### Feasible starting point method

Using second order approximation to primal objective at a point $x$ yields

$\min_v (f(x)+\nabla f(x)^T \nu + (1/2)v^T\nabla^2 f(x)\nu): A(x+\nu)=b $

The method starts with a feasible $x^{(0)}$, and $\Delta x$ update preserves the feasibility. 

Note that $A(x+\nu)=b\, \Rightarrow A\nu=0$. The Newton step $\Delta x$ is then taken as the solution to the second order approximation problem, which is also the solution of the KKT condition equations

$$\begin{pmatrix} \nabla^2f(x) & A^T \\ A & 0 \end{pmatrix} \begin{pmatrix} \Delta x \\ \nu \end{pmatrix} = \begin{pmatrix} - \nabla f(x) \\ 0 \end{pmatrix}$$.

##### Infeasible starting point method

With a infeasible start point, we make updates to best approximate feasibility and satisfies the optimal conditions.

$$\begin{pmatrix} \nabla^2f(x) & A^T \\ A & 0 \end{pmatrix} \begin{pmatrix} \Delta x \\ \nu \end{pmatrix} = - \begin{pmatrix} \nabla f(x) \\ Ax-b \end{pmatrix} \tag{1}$$

Remark:

The infeasible start Newton's method takes a step at a point not gaurenteeing a decrement step to the objective value. However, once it reaches feasibility, it will coincide with what is in the feasible case Newton's method, and thereafter the step is sure to be a decrement step.

The algorithm:

Repeat **until** $Ax = b$ and $r(x, \nu) \le \epsilon$

- Compute primal and dual Newton steps $\Delta x$ and $\Delta \nu$. The dual step $\Delta \nu$ is the solution to (1) minus the previous $\nu$.
- Backtrack on $\Vert r \Vert_2$. $r$ is the primal-dual residual $$\begin{pmatrix} \nabla f(x)+A^Tb \\ Ax-b \end{pmatrix}$$
  - $t \leftarrow 1$
  - while $\Vert r(x+t\Delta x, \nu + t\Delta \nu)\Vert_2\gt (1-\alpha t)\Vert r(x, \nu)\Vert_2$, do $t \leftarrow \beta t$


- $x \leftarrow x+t\Delta x$, $\nu \leftarrow \nu + \Delta \nu$

---

## Inequality constrained methods

#### Barrier method

- **Log barrier**

  For a problem with inequality constraints:

  $\min f_0(x): Ax=b, f(x)\le0$

  relax it to:

  $\min f_0(x) + \sum_i -(1/t) \log(-f_i(x)): Ax=b$

  or equivalently:

  $\min tf_0(x) + \phi(x): Ax=b$.

  The **log barrier function** $\sum_i -(1/t) \log(-f_i(x))$ is an approximation to the ideal barrier $\sum_iI_-(-f_i(x))$, where the $t$ here is a parameter. The log barrieir is convex so it preserves the convexity of the objective. As $t \rightarrow \infty$, the log barrier approaches the ideal barrier.

- **Central path and $m/t$-suboptimal**

  The **central path** associated with the problem is defined as the set of points $x^\ast(t)$ for $t>0$ such that the KKT conditions enhanced with strictly feasibility hold for the barrier relaxed problem parametrized at $t$.

  Central path conditions can also be interpreted as relaxed KKT conditions on the original problem with relaxed complementarity/duality gap:

  $\lambda_i f_i(x) = -1/t\, , \forall i$.

  For $x^\ast(t)$ on the central path, it is a **$m/t$-suboptimal** point for the original problem, i.e.

  $f_0(x^\ast(t)) - p^\ast \le m/t$.

- **Barrier method**

  Choose a strictly feasible starting point $x$, and $t \leftarrow t^{(0)}$, $\mu \gt 1$, tolerance $\epsilon > 0$

  - Centering: $x \leftarrow x^\ast(t)$, by solving  $\min tf_0(x) + \phi(x): Ax=b$, with starting point at the latest $x$
  - Update: $t \leftarrow \mu t$ if $m/t \lt \epsilon$ else quit

  Remark:

  - An infeasible starting point method can be used to solve the equality constrained problem at the centering step.


  - To choose a strictly feasible starting point, we add an additional phase before called **phase I**. By solving an optimization problem such as:

    $\min s: f(x) \le s\textbf{1}, Ax=b$. (where $s$ is a scalar), or

    $\min \textbf{1}^Ts: f(x) \le s, Ax=b, s \ge 0$. (where $s$ is a vector)

    We can compute a strictly feasible point for the original problem, or conclude that the constraints are infeasible.

    Note that the phase I problem is formulated such that a strict feasible point is easy to attain, so we can solve by pure barrier method. Or alternatively, we can also use infeasible starting point Newton's method to solve the barrier problem associated with the phase I problem.

#### Primal dual interior point method

TODO

---

## Solving symmetric linear system

We are considering a linear system

$Ax=b$, where $A$ is PSD, to find the solution $x^\ast$ is equivalent to find a minimum of:

$\phi(x) = 1/2 x^TAx - b^Tx$

#### Conjugate gradient method - [ref](http://www.cs.cmu.edu/~pradeepr/convexopt/Lecture_Slides/conjugate_direction_methods.pdf)

- **$Q$-Conjugate**

  $\\{d_1, …, d_n\\}$ are said to be $Q$-conjugate/orthogonal if $d_i^TQd_j =0$ for any $i \neq j$.

- **Conjugate direction theorum**

  Let a set of *given* vectors $\\{d_1, …, d_n\\}$ be $Q$-conjugate, denoted as $\perp^{(Q)}$, and $x_0$ be arbitrary starting point, then after $n$ steps of the following process, $x_n = x^\ast$:

  - $x_{k+1} = x_k + \alpha_k d_k$
  - $g_k = Qx_k - b$
  - $\alpha_k = - \frac{g_k^Td_k}{d_k^TQd_k}$

  Remark:

  This is essentially factorizing $x^\ast - x_0$ into the $Q$-conjugate basis $\\{d_1, …, d_n\\}$ sequentially. $\alpha_k$ corresponds to the "coordinate" under the $Q$-conjugate basis in the sense that:

  $\frac{(x^\ast - x_0)^TQd_k}{d_k^T Q d_k} = \frac{(x^\ast - x_k)^TQd_k}{d_k^T Q d_k} = \alpha_k$, since 

  $(x^\ast - x_k) - (x^\ast - x_0) = \sum_{i=0,…,k-1}\alpha_i d_i$ which is $Q$-conjugate with $d_k$.

- **Conjugate gradient algorithm**

  Since we are not given the $Q$-conjugate basis beforehand, so compute them also along the iteration:

  *The update steps are the same with conjugate direction theorum:*

  - $x_{k+1} = x_k + \alpha_k d_k$
  - $g_k = Qx_k - b$
  - $\alpha_k = - \frac{g_k^Td_k}{d_k^TQd_k}$

  *but with additional $d_k$ calculation*

  - $d_0 = b - Qx_0$
  - $d_{k+1} = -g_{k+1} + \beta_kd_k$
  - $\beta_k = \frac{g_{k+1}^TQd_k}{d_k^TQd_k}$

  Remark:

  To verify the correctness, observing that conjugate direction theorum holds for any basis, one only needs to show that the $\\{d_k\\}$ constructed by the conjugate gradient algorithm is indeed $Q$-conjugate.

  In the induction step of the proof, it is sufficient to show that both of the following hold:

  1. $d_{k+1} \perp^{(Q)} d_k$
  2. $g_{k+1} \perp^{(Q)} \\{d_{i<k+1}\\}$

  the first one is straightforward, the second is the same as in the remark in the conjugate direction theorum section.