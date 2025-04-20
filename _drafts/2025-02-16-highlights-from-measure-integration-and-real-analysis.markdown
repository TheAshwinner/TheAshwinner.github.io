---
layout: post
title:  "Highlights from Measure, Integration, and Real Analysis"
date:   2025-02-17 14:13:17 -0800
last_modified_at: 2025-04-13 14:13:17 -0800
categories: math
---

I am currently working through the textbook *Measure, Integration, and Real Analysis* by Sheldon Axler. Think of this post as a collection of highlights of key ideas, lemmas, theorems, etc. that I have found interesting so far. These notes are not comprehensive: there will likely be many important intermediate results that I have not included. This post will be relatively technical, but I will try to explain the intuition and motivation for the results.

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

{% include custom-box.html
   type="theorem"
   header="Weaknesses of Riemann integration"
   content="

1. Riemann integration does not handle functions with many discontinuities well.

2. Riemann integration does not handle unbounded functions well.

3. Riemann integration does not play well with limits of sequences of functions.

   "
%}

For (1), consider the function $$f(x) = \begin{cases} 1 & x \in \mathbb{Q} \\ 0 & x \in \mathbb{R} \setminus \mathbb{Q} \end{cases}$$

This function is discontinuous at every $$x \in \mathbb{R}$$. We can see that there are countably many points $$x$$ at which $$f(x)$$ is 1 (as $$\mathbb{Q}$$ is countable) while it is 0 at uncountably many points. This observation might intuitively suggest that the integral $$ \int f(x)$$ should be 0 since $$f(x)$$ is 0 at "most" points. Such a function is not Riemann integrable however: $$\inf_{x \in [x_{i-1}, x_i]} f(x) = 0$$ and $$\sup_{x \in [x_{i-1}, x_i]} f(x) = 1$$ for all $$i$$, so the upper Riemann sum $$U(f, P, C)$$ and lower Riemann sum $$L(f, P, C)$$ are always bounded away from each other, regardless of the partition $$P$$. This means that the Riemann integral is not defined. The Lebesgue integral, which we will introduce later, is able to handle such a function.

For (2), consider the function $$f(x) = \begin{cases}\frac{1}{\sqrt{x}} & 0 < x \leq 1 \\ 0 & x = 0 \end{cases}$$ on the interval $$[0, 1]$$. This integral can be computed as $$\int_0^1 \frac{1}{\sqrt{x}} \, dx = 2$$. But notice that for any partition $$P = \{x_0, x_1, \cdots, x_n : 0 = x_0 \leq x_1 \leq \cdots \leq x_n = 1\}$$ of $$[0,1]$$, we have that $$\sup_{x \in [x_0, x_1]} f(x) = \infty$$. So the upper Riemann sum $$U(f, P, C)$$ is unbounded, and the Riemann integral is not defined.

For (3), we can construct a sequence of Riemann integrable functions that converges pointwise to a function that is not Riemann integrable. For example, let $$q_1, q_2, \ldots \in \mathbb{Q}$$ be an enumeration of all rational numbers in the interval $$[0,1]$$. Define a sequence of functions $$ f_n : [0,1] \to \mathbb{R}$$ by the following:

$$f_n(x) = \begin{cases} 1 & x = q_1, q_2, \ldots, q_n \\ 0 & \text{otherwise} \end{cases} $$

Each $$f_n$$ is Riemann integrable, and $$\lim_{n \to \infty} f_n(x) = 0$$ for all $$x \in [0,1]$$. However, $$f_n \to f$$ pointwise, where $$f(x) = \begin{cases} 1 & x \in \mathbb{Q} \\ 0 & x \in \mathbb{R} \setminus \mathbb{Q} \end{cases}$$, which we showed above is not Riemann integrable.

These three weaknesses motivate a different approach to integration.

## Chapter 2: Measures

This chapter introduces the notion of a measure, which generalizes the notion of length (area, volume, etc) to a larger class of sets in $$\mathbb{R}^n$$. One motivation for this construction is that it allows us to formalize integration with respect to a measure. We use this to define the Lebesgue integral, an improvement over the Riemann integral.

### Formalizing the notion of measure

We define $$\ell(I)$$ to be the length of an interval $$I$$. So $$\ell((a,b)) = b-a$$.

There are a few desirable properties that we might like a measure $$\mu$$ on $$\mathbb{R}$$ to have:

{% include custom-box.html
   type="theorem"
   header="Desiderata for a measure"
   content="

1. $$\mu$$ is a function from the set of all subsets of $$\mathbb{R}$$ to $$[0, \infty]$$
   - Intuitively, if we measure the length of a set, we should get a non-negative real number. We want this property to be true for measures as well.
2. $$\mu(I) = \ell(I)$$ for every open interval $$I$$ of $$\mathbb{R}$$.
   - We want the measure to agree with length for any set that such a length is defined for.
3. $$\mu\left(\bigcup_{k=1}^\infty A_k\right) = \sum_{k=1}^\infty \mu(A_k)$$ for every disjoint sequence $$A_1, A_2, \ldots$$ of subsets of $$\mathbb{R}$$.
   - If we partition a set into smaller pieces, the measure of the whole set should be the sum of the measures of the pieces.
4. $$\mu(t + A) = \mu(A)$$ for every $$A \subseteq \mathbb{R}$$ and every $$t \in \mathbb{R}$$.
   - Moving a set should not change its measure.

   "
%}

Surprisingly, there is no measure on $$\mathbb{R}$$ that satisfies all of these properties. (For a full proof, see theorem 2.22 in the textbook. The standard counterexample is the Vitali set.)

It seems like we must relax some of these properties. In practice, we usually relax property (1): instead of requiring that $$\mu$$ is defined for **all** subsets of $$\mathbb{R}$$, we only require that $$\mu$$ is defined for **some** collection of subsets of $$\mathbb{R}$$ (the measurable sets with respect to $$\mu$$).
In practice, we can choose this collection of subsets (i.e. a $$\sigma$$-algebra) such that nearly all "important" sets that we care about are measurable.

As it turns out, the way we construct such a measure is to first define the "outer measure" of a set, which is a function that satisfies properties (1), (2), and (4) above. We then restrict the domain of this function to the collection of measurable sets and show that this restricted function (which no longer satisfies property (1)) does satisfy property (2), (3), and (4).

### Cantor set and Cantor function

The Cantor set is a standard counterexample in measure theory as an uncountable set with measure 0. The Cantor set is constructed by starting with the interval $$[0,1]$$. At each step, we remove the middle third of each interval in the previous step. After infinitely many steps, we are left with the Cantor set.

For example, let $$C_k$$ be the $$k$$th step in the construction of the Cantor set with $$C_0 = [0,1]$$. Then $$C_1 = [0, 1/3] \cup [2/3, 1]$$, $$C_2 = [0, 1/9] \cup [2/9, 1/3] \cup [2/3, 7/9] \cup [8/9, 1]$$, and so on.

<!-- TODO: add an image of the Cantor set -->

{% include custom-box.html
   type="theorem"
   header="Definition of the Cantor set"
   content="
Let $$C_0 := [0,1]$$, and $$G_k :=\Big( \bigcup_{i=0}^{3^{k-1}-1} (\frac{3i + 1}{3^k},\frac{3i + 2}{3^k}) \Big)$$.

Then the Cantor set is given by $$ C := C_0 \setminus \bigcup_{k=1}^\infty G_k$$
   "
%}

Note that $$G_k$$ is the union of the middle thirds of the intervals of length $$\frac{1}{3^k}$$. Each $$G_k$$ should be removed from $$C_0$$ to produce the Cantor set.

Notice that $$ \lvert \bigcup_{k=1}^{n} G_k \rvert = \frac{1}{3} + \frac{2}{3^2} + \cdots + \frac{2^{n-1}}{3^n} = 1 - \frac{2^n}{3^n}$$. As $$n \to \infty$$, this converges to 1. We know that $$C_0 = [0,1]$$ has measure 1, so $$\lvert C \rvert = \lvert C_0 \setminus \bigcup_{k=1}^{\infty} G_k \rvert = \lim_{n \to \infty} \lvert C_0 \setminus \bigcup_{k=1}^{n} G_k \rvert  = \lim_{n \to \infty} (1 - (1- \frac{2^{n}}{3^n})) = 0$$. So the Cantor set has measure 0.

The Cantor set is also an uncountable set. One way to see this is to consider base 3 representations of numbers. Every number in $$[0,1]$$ can be represented as a base 3 decimal. The numbers in the Cantor set are those numbers that do not have a 1 in their base 3 decimal expansion. Such a set can be shown to be uncountable using a Cantor diagonalization argument.
