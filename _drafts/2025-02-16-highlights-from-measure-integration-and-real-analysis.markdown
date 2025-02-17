---
layout: post
title:  "Highlights from Measure, Integration, and Real Analysis"
date:   2025-02-17 14:13:17 -0800
categories: math
---

I am currently working through the textbook *Measure, Integration, and Real Analysis* by Sheldon Axler. Think of this post as a collection of highlights of key ideas, lemmas, theorems, etc. that I have found interesting so far. This post will be relatively technical, but I will try to explain the intuition and motivation for the results.

I assume familiarity with basic concepts in real analysis including pointwise versus uniform convergence, uniform continuity, and the Riemann integral.

## Chapter 1: Riemann Integration

Recall the definition of the Riemann integral as the following:

The upper Riemann sum of a function $$f$$ on an interval $$[a, b]$$ is given by:

$$U(f, P, C) = \sum_{i=1}^n (x_i - x_{i-1}) \sup_{x \in [x_{i-1}, x_i]} f(x) $$

while the lower Riemann sum is given by:

$$L(f, P, C) = \sum_{i=1}^n (x_i - x_{i-1}) \inf_{x \in [x_{i-1}, x_i]} f(x) $$

where $$P = \{a = x_0 < x_1 < \cdots < x_n = b \}$$ is a partition of $$[a, b]$$.

The function $$f$$ is Riemann integrable if the supremum of the lower Riemann sums equals the infimum of the upper Riemann sums as the mesh of the partition (the maximum length of the intervals in the partition: i.e. $$\max_i \| x_i - x_{i-1} \|$$) approaches zero:

$$\lim_{\text{mesh}(P) \to 0} U(f, P, C) = \lim_{\text{mesh}(P) \to 0} L(f, P, C)$$

This common value is called the Riemann integral of $$f$$ over $$[a, b]$$ and is denoted by $$\int_a^b f(x) \, dx$$.

#### Limitations of Riemann Integration

