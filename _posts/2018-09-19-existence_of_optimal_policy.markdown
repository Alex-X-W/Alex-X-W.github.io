---
layout:     post
title:      "On the existence of optimal policy"
date:       2018-09-19
author:     "Xuan"
header-img: "img/in-post/10/header.jpeg"
header-mask: 0.3
catalog:    true
mathjax:    true
multilingual: false
tags:
    - reinforcement learning
---
$\newcommand{\mc}{\mathcal} \newcommand{\mb}{\mathbb}$

## Setting

We are considering in the setting of:

- Discrete actions
- Discrete states
- Bounded rewards
- Stationary policy
- Infinite horizon

The **optimal policy** is defined as:
$$
\pi^\ast \in \arg \max_\pi V^\pi(s), \forall s \in \mc{S} \tag{1}
$$
and the **optimal value function** is:
$$
V^\ast = \max_\pi V^\pi (s), \forall s \in \mc S \tag{2}
$$
There can be a set of policies which achieve the maximum. But there is only one optimal value function:
$$
V^\ast = V^{\pi^\ast} \tag{3}
$$


## The question

**How to prove that there exists at least one $\pi^\ast$ which satisfies (1) simultaneously for all $s \in \mc{S}$ ?**



## Outline of proof

1. Construct the **optimal equation** to be used as a temporary surrogate definition of optimal value function, which we will prove in step 2 that it is equivalent to the definition via Eq.(2).
   $$
   V^\ast(s) = \max_{a \in \mc A} [ R(s, a) + \gamma \, \sum_{s^\prime \in \mc S} T(s, a, s^\prime) V^\ast(s^\prime)] \tag{4}
   $$

2. Derive the equivalency of defining optimal value function via Eq.(4) and via Eq.(2).

   (Note in fact we only need the necessity direction in the proof, but the sufficiency is obvious since we construct Eq.(4) from Eq.(2).)

3. Prove that there is a unique solution to Eq.(4).

4. By step 2, we know that the solution obtained in step 3 is also a solution to Eq.(2), so it is an optimal value function.

5. From an optimal value function, we can recover an optimal policy by choosing the maximizer action in Eq.(4) for each state.



## Details of the steps

#### 1

Since $V^\ast(s) = V^{\pi^\ast}(s) = \mb E_a [Q^{\pi^\ast}(s, a)]$, we have $V^{\pi^\ast}(s) \le \max_{a \in \mc A} Q^{\pi^\ast} (s, a)$. And if there is any $\tilde{s}$ such that $V^{\pi^\ast} \neq \max_{a \in \mc A} Q^{\pi^\ast} (s, a)$, we can choose a better policy by maximizing $Q^{\ast} (s, a) = Q^{\pi^\ast} (s, a)$ over $a$.

#### 2

##### (=>)

Follows by step 1.

##### (<=)

i.e. If $\tilde V$ satisfies $\tilde V(s) = \max_{a \in \mc A} [ R(s, a) + \gamma \, \sum_{s^\prime \in \mc S} T(s, a, s^\prime) \tilde V(s^\prime)]$, then $\tilde V(s) = V^\ast(s) = \max_\pi V^\pi(s), \forall s \in \mc S$.

Define the **optimal Bellman operator** as
$$
\mc T V(s) = \max_{a \in \mc A} [ R(s, a) + \gamma \, \sum_{s^\prime \in \mc S} T(s, a, s^\prime) V(s^\prime)] \tag{5}
$$
So our goal is to prove that if $\tilde V = \mc T \tilde V$, then $\tilde V = V^\ast$. We show this by combining two results, following *Puterman*[1]:

a) If $\tilde V \ge \mc T \tilde V$, then $\tilde V \ge V^\ast$.

b) If $\tilde V \le \mc T \tilde V$, then $\tilde V \le V^\ast$.

> Proof:
>
> a)
>
> For any $\pi = (d_1, d_2, ...)$,
> 
> $$
> \begin{align*}
> \tilde V &\ge \mc T \tilde V = \max_{d} [ R_d + \gamma \, P_d \tilde V] \\
> &\ge R_{d_1} + \gamma \, P_{d_1} \tilde V \\
> \end{align*}
> $$
> 
> Here $d$ is the decision rule(action profile at specific time), $R_d$ is the vector representation of immediate reward induced from $d$ and $P_d$ is transition matrix induced from $d$. 
>
> By induction, for any $n$,
> 
> $$
> \tilde V \ge R_{d_1} + \sum_{i=1}^{n-1} \gamma^i P_\pi^i R_{d_{i+1}} + \gamma^n P_\pi^n \tilde V
> $$
> 
> where $P_\pi^j$ represents the $j$-step transition matrix under $\pi$.
>
> Since 
> 
> $$
> V^\pi = R_{d_1} + \sum_{i=1}^{\infty}\gamma^i P_\pi^i R_{d_{i+1}}
> $$
> 
> we have
> 
> $$
> \tilde V - V^\pi \ge \underbrace{\gamma^n P_\pi^n \tilde V -\sum_{i=n}^{\infty}\gamma^i P_\pi^i R_{d_{i+1}}}_{\rightarrow 0 \ \text{as}\ n\rightarrow \infty}
> $$
> 
> So we have $\tilde V \ge V^\pi$. And since this holds for any $n$, we conclude that
> 
> $$
> \tilde V \ge \max_\pi V^\pi = V^\ast
> $$
> 
> b)
> 
> Follows from step 1.

#### 3

The optimal Bellman operator is a contraction in $L_\infty$ norm, cf. [2].

> Proof:
>
> For any $s$,
> $$
> \begin{align*}
> \left\vert \mc T V_1(s) - \mc TV_2(s) \right\vert 
> &=
> \left\vert \max_{a \in \mc A} [ R(s, a) + \gamma \, \sum_{s^\prime \in \mc S} T(s, a, s^\prime) V_1(s^\prime)] -\max_{a^\prime \in \mc A} [ R(s, a^\prime) + \gamma \, \sum_{s^\prime \in \mc S} T(s, a^\prime, s^\prime) V(s^\prime)]\right\vert \\
> &\overset{(*)}{\le}
> \left\vert \max_{a \in \mc A} [\gamma \, \sum_{s^\prime \in \mc S} T(s, a, s^\prime) (V_1(s^\prime) - V_2(s^\prime))] \right\vert \\
> &\le
> \gamma \Vert V_1 - V_2 \Vert_\infty
> \end{align*}
> $$
> where in (*) we used the fact that
> $$
> \max_a f(a) - \max_{a^\prime} g(a^\prime) \le \max_a [f(a) - g(a)]
> $$
>

Thus by Banach fixed point theorum it follows that $\mc T$ has a unique fixed point.



## References

[1] Puterman, Martin L.. “Markov Decision Processes : Discrete Stochastic Dynamic Programming.” (2016).

[2] A. Lazaric. http://researchers.lille.inria.fr/~lazaric/Webpage/MVA-RL_Course14_files/slides-lecture-02-handout.pdf