---
layout: post
title:  "Highlights from Measure, Integration, and Real Analysis"
date:   2025-02-17 14:13:17 -0800
categories: math
---

I am currently working through the textbook *Measure, Integration, and Real Analysis* by Sheldon Axler. Think of this post as a collection of highlights of key ideas, lemmas, theorems, etc. that I have found interesting so far. This post will be relatively technical, but I will try to explain the intuition and motivation for the results.

I assume familiarity with basic concepts in real analysis including pointwise versus uniform convergence, uniform continuity, and the Riemann integral.

## Chapter 1: Riemann Integration

### Recap of Riemann Integration

The Riemann integral is generally the first integration theory that students encounter. At a high level, if we want to compute the Riemann integral for a function $$f$$ over an interval $$[a, b]$$, we partition the interval into smaller and smaller subintervals and sum up the areas of the rectangles that lie above the curve (if we are computing the upper Riemann sum) or below the curve (if we are computing the lower Riemann sum). For Riemann integrable functions, these two sums should converge to the same value as the subintervals get arbitrarily small.

<!-- TODO: add diagram -->

More formally, the upper Riemann sum of a function $$f$$ on an interval $$[a, b]$$ is given by:

$$U(f, P, C) = \sum_{i=1}^n (x_i - x_{i-1}) \sup_{x \in [x_{i-1}, x_i]} f(x) $$

while the lower Riemann sum is given by:

$$L(f, P, C) = \sum_{i=1}^n (x_i - x_{i-1}) \inf_{x \in [x_{i-1}, x_i]} f(x) $$

where $$P = \{x_0, x_1, \cdots,  x_n : a = x_0 < x_1 < \cdots < x_n = b \}$$ is a partition of $$[a, b]$$.

We define the mesh of a partition to be the maximum length of the intervals in the partition: i.e. $$\text{mesh}(P) = \max_i (x_i - x_{i-1})$$.

Then, the function $$f$$ is Riemann integrable if $$\lim_{\text{mesh}(P) \to 0} U(f, P, C) = \lim_{\text{mesh}(P) \to 0} L(f, P, C)$$

This common value is called the Riemann integral of $$f$$ over $$[a, b]$$ and is denoted by $$\int_a^b f(x) \, dx$$.

### Limitations of Riemann Integration

There are a few weaknesses with the Riemann integral that motivate the need for a more powerful integration theory.

1. Riemann integration does not handle functions with many discontinuities well.

2. Riemann integration does not handle unbounded functions well.

3. Riemann integration does not play well with limits of sequences of functions.

For (1), consider the function $$f(x) = \begin{cases} 1 & x \in \mathbb{Q} \\ 0 & x \in \mathbb{R} \setminus \mathbb{Q} \end{cases}$$

This function is discontinuous at every $$x \in \mathbb{R}$$. We can see that there are countably many points $$x$$ at which $$f(x)$$ is 1 (as $$\mathbb{Q}$$ is countable) while it is 0 at uncountably many points. This observation might intuitively suggest that the integral $$ \int f(x)$$ should be 0 since $$f(x)$$ is 0 at "most" points. Such a function is not Riemann integrable however: $$\inf_{x \in [x_{i-1}, x_i]} f(x) = 0$$ and $$\sup_{x \in [x_{i-1}, x_i]} f(x) = 1$$ for all $$i$$, so the upper Riemann sum $$U(f, P, C)$$ and lower Riemann sum $$L(f, P, C)$$ are always bounded away from each other, regardless of the partition $$P$$. This means that the Riemann integral is not defined. The Lebesgue integral, which we will introduce later, is able to handle such a function.

For (2), consider the function $$f(x) = \begin{cases}\frac{1}{\sqrt{x}} & 0 < x \leq 1 \\ 0 & x = 0 \end{cases}$$ on the interval $$[0, 1]$$. This integral can be computed as $$\int_0^1 \frac{1}{\sqrt{x}} \, dx = 2$$. But notice that for any partition $$P = \{x_0, x_1, \cdots, x_n : 0 = x_0 \leq x_1 \leq \cdots \leq x_n = 1\}$$ of $$[0,1]$$, we have that $$\sup_{x \in [x_0, x_1]} f(x) = \infty$$. So the upper Riemann sum $$U(f, P, C)$$ is unbounded, and the Riemann integral is not defined.

For (3), we can construct a sequence of Riemann integrable functions that converges pointwise to a function that is not Riemann integrable. For example, let $$q_1, q_2, \ldots \in \mathbb{Q}$$ be an enumeration of all rational numbers in the interval $$[0,1]$$. Define a sequence of functions $$ f_n : [0,1] \to \mathbb{R}$$ by the following:

$$f_n(x) = \begin{cases} 1 & x = q_1, q_2, \ldots, q_n \\ 0 & \text{otherwise} \end{cases} $$

Each $$f_n$$ is Riemann integrable, and $$\lim_{n \to \infty} f_n(x) = 0$$ for all $$x \in [0,1]$$. However, $$f_n \to f$$ pointwise, where $$f(x) = \begin{cases} 1 & x \in \mathbb{Q} \\ 0 & x \in \mathbb{R} \setminus \mathbb{Q} \end{cases}$$, which we showed above is not Riemann integrable.

These three weaknesses motivate a different approach to integration.