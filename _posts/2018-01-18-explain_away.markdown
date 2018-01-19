---
layout:     post
title:      "About explaining away"
subtitle:   "experiments and thoughts"
date:       2018-01-18
author:     "Xuan"
header-img: "img/in-post/4/header.jpeg"
header-mask: 0.3
catalog:    true
multilingual: false
tags:
    - pgm
---

### Explore the phenomenon of "explaining away"
First, let's consider a simple example, where a guy's happiness (H) can be caused by either a raise in his company (R) or a sunny day (S), and assuming the priors of R and S are independent.  

Now if we set the probabilities to these values:  
- Priors:  
P(S)=0.7  
P(R)=0.01

- CPD table:

Sunny | Raised | Happy | $P$
--- | --- | --- | ---
T | T | T | 1
F | T | T | 0.9
T | F | T | 0.7
F | F | T | 0.1

#### Before knowing about the wheather, P(R|H)
First let's look at P(R|H), which is the probability of the guy getting a raise given he is happy (without knowing if today is a sunny day).  
```python
def P_R_given_H(P_S,
                P_R,
                P_11,
                P_01,
                P_10,
                P_00):
    
    nominator = (P_11*P_S + P_01*(1-P_S)) * P_R
    denominator = P_11*P_S*P_R + P_01*(1-P_S)*P_R + P_10*P_S*(1-P_R) + P_00*(1-P_S)*(1-P_R)
    return nominator / denominator
```
Now if we use the CPD table given above, we get  
```
P(R|H) is 0.018494
```  

#### After checking out it is sunny, P(R|S,H)
Next if we now already know today is a sunny day, let's look at P(R|S,H):  
```python
def P_R_given_S_H(P_S,
                  P_R,
                  P_11,
                  P_01,
                  P_10,
                  P_00):
    
    nominator = P_11 * P_S * P_R
    denominator = P_S * (P_11*P_R + P_10*(1-P_R))
    return nominator / denominator
```
Again if we use the CPD table given above, we get  
```
P(R|S, H) is 0.014225
```

#### Explaining-away just happened!
_so we can see that  
P(R|S,H)<P(R|H)  
which is saying after knowing today is sunny, the probability of the guy getting raised gets smaller so that R got **explained away!**_  

#### Further analysis
By some simple mathematical transformation by applying the bayes rule and full probability formula, we found that P(R|S,H) < P(R|H) satisfies iff our CPD tables satisfies P(H|S,R)⋅P(H|¬S,¬R) > P(H|S,¬R)⋅P(H|¬S,R).
Let's test this by a counterexample concrete CPD table.  

Sunny | Raised | Happy | $P$
--- | --- | --- | ---
T | T | T | 1
F | T | T | 0.6
T | F | T | 0.55
F | F | T | 0.4

And this time when we print the values using `P_R_given_H` and `P_R_given_S_H` we got:  
```
P(R|H) is 0.807207
P(R|S, H) is 0.809249
```

#### Final remarks
In practice, it is usually more natural to encode positively influential events in a intercausal reasoning networks. So we should have P(H|S,R)>0.5, P(H|¬S,R)>0.5 and P(H|S,¬R)>0.5 at the same time. And usually we also have P(H|S,R) being the biggest one. So we have P(H|S,¬R)⋅P(H|¬S,R) > 0.25.  
Also, the prior of the effect (in our example is P(H)) tends to be small, because if it is a big prior, there is less reason we want a model to predict its happening. Note that P(H|¬S,¬R) is comparable with P(H). So if the prior  P(H) < 0.25, then the relation P(H|S,R)⋅P(H|¬S,¬R) > P(H|S,¬R)⋅P(H|¬S,R) satisfies immediately. So in most cases, explaining away happens in a positive direction.