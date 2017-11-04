---
layout:     post
title:      "Elements of GPU overflow programming:P"
subtitle:   "an exploded Integer whitens all the data"
date:       2017-11-04
author:     "Xuan"
header-img: "img/in-post/3/header.jpeg"
header-mask: 0.3
catalog:    true
multilingual: false
tags:
    - engineering
    - gpu
---

### An exploded Integer whitens all the data
#### 0. Background
I am studying a little on GPU (specifically Nvidia ones) these days. And the recent toy problem I worked on was a simple prime number generating algorithm. There are a lot of algorithms to go but since my goal is around GPU so I just took an unsalted version with little optimization, the Greek one called [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).  
![](/img/in-post/3/Sieve_of_Eratosthenes_animation.gif)  

*source: [wikipedia](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)*

#### 1. The Problem
The algorithm in pseudocode is straightforward:  
![](/img/in-post/3/algorithm.png)  
*source: [wikipedia](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)*  
So far it seemed all right huh? Oh you just spotted a pitfall? You are welcome with my boxes in the below image :)  
For a slightly better performance which turned out to be the trigger of overflow (and this blog), we can replace the `i < sqrt(N)` computation by `i * i > N`. Then...
Overflow time!
```c
/*
bang when `multiplier` reaches ceil(sqrt(2^31)) = 46341
*/
/*
which is when `N` gets to (46341-2-1)*2 + 2 = 92678
*/
__global__ void CUDACross(bool *candidates, int size){
    for (int idx = blockIdx.x*blockDim.x + threadIdx.x; idx < size/2 + 1; idx += blockDim.x * gridDim.x) {
        int multiplier = idx + 2;
        int check = multiplier * multiplier;
        while (check < size + 2) {
            candidates[check - 2] = false;
            check += multiplier;
        }
    }
}
```

#### 2. The Scene
I was originally thinking that even when overflows, those composite numbers got crossed out before overflow should not be affected. But mysteriously when overflow happens, the whole bulk data come out as untouched. Nothing got crossed!  
It turned out oddly there was a clear cleavage of correct and error, precisely at N = 92678. The weird behavior was quite steadily reproducible when I tested with varying input sizes. And it was so reproducible that made me believe it was indeed a bug in my code, not in third party libraries cuz I didn't include any, surely not in system libraries cuz they never betrayed me, and absolutely not in the processor cuz I ran the program not with my machine...

> More experienced programmers know very well that the bug is generally in their code: occasionally in third-party libraries; very rarely in system libraries; exceedingly rarely in the compiler; and never in the processor.

*[quote from Xavier Leroy](http://gallium.inria.fr/blog/intel-skylake-bug/)*

The behavior when such integer overflow happens is weird, all the changes you have made are gone... I suspect it is designed so for some reason I don't know. I think I understand now the words from [Prof. Z.](http://www.mzahran.com) who is teaching us GPU that  
> Nvidia likes to make themselves mysterious.

#### 3. Want to play with it?
##### 3.1 Requirements
I tested on both `GeForce GTX TITAN X` and `GeForce GTX TITAN Z` and got the same behaviors. No guarantee on other machines but I am willing to bet.  
##### 3.2 Codes
I pointed to the [repo](https://github.com/Alex-X-W/gpu_overflow) related if you want to play with. It should be easy to reproduce the results.  
You should see a clear cleavage when N = 92678. When N < 92678, all good; once N >= 92678, nothing got crossed out. (by "crossed out/crossed" I mean deleted from candidate list)  
To test if all numbers are correct, you can use [`evaluate.py`](https://github.com/Alex-X-W/gpu_overflow/blob/master/evaluate.py):  
`$ python evaluate.py --gold 1stmillion.txt --test [YOUR_OUTPUT]`
to see a fraction of the output, lets say from char position 12 through 25, you can use:  
`$ cut -c 12-25 [YOUR_OUTPUT]`
