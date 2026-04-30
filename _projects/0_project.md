---
layout: page
title: Direction-Preserving Number Representations
description: A geometric look at how low-precision number formats preserve vector directions, and why FP4-E2M1 performs surprisingly well.
importance: 0
img: /assets/img/fp4_e2m1_unit_vectors_2d.png
category: work
---

# Direction-Preserving Number Representations

Low-precision number formats are now central to modern machine learning systems. They reduce memory use, improve bandwidth efficiency, and make large-scale inference and training more practical.

Most discussions of low-precision representation focus on value error: how close a quantised number or vector is to the original in terms of absolute error, mean-squared error, or norm distortion.

This project looks at a different question:

> When a real-valued vector is quantised element by element, how well does the resulting representation preserve the vector’s direction?

This matters because many machine learning operations are highly sensitive to direction. Similarity, gradients, and inner products all depend strongly on angular structure. If a low-precision format preserves vector directions well, it can retain useful geometric information even when the individual scalar values are coarse.

The central idea of this work is to study number formats geometrically: not only as sets of scalar values, but as generators of representable directions in vector space.

## Interactive demo

Try the interactive demo here: [Directional Coverage Explorer](https://bardia01.github.io/directional_coverage_explorer/){:target="_blank"}.

---

## From Scalar Alphabets to Vector Directions

A scalar number format defines a finite alphabet of representable values. For example, a 4-bit format gives 16 possible scalar values.

When vectors are quantised element by element, each coordinate is selected from this scalar alphabet. This produces a product-structured code: the vector is built by taking one scalar value per coordinate.

For a scalar alphabet $A$, the representable directions in dimension $n$ are

$$
P_n(A) := \left\{ \frac{x}{\|x\|_2} : x \in A^n \setminus \{0\} \right\}.
$$

This set lives on the unit sphere. Each point corresponds to a direction that can be represented by the quantised vector.

The quality of a scalar alphabet can therefore be measured by how well these points cover the sphere. If every possible direction is close to some representable direction, the format has good directional fidelity.

---

## FP4-E2M1 as a Directional Code

The figure below gives a simple two-dimensional view of the 4-bit E2M1 floating-point format. Each grid intersection corresponds to a possible quantised vector. When these vectors are projected onto the unit circle, they become representable directions.

![Representable directions for the FP4-E2M1 product code](/assets/img/fp4_e2m1_unit_vectors_2d.png){: width="520" }

**Figure 1.** Directional coverage for a product code using two FP4-E2M1 elements. Each point on the circle is a direction that can be represented by the product structure. The directions are not equally spaced, which creates angular gaps.

This picture shows the key geometric issue. Even if a scalar alphabet is well designed numerically, the directions created by its product structure may leave uneven gaps on the sphere.

Those gaps determine the worst-case angular error.

---

## Measuring Directional Coverage

For a scalar alphabet $A$, define the worst-case angular error as the largest angle between any direction on the unit sphere and the nearest representable direction:

$$
F_n(A) :=
\sup_{u \in S^{n-1}}
\min_{c \in P_n(A)}
\angle(u, c).
$$

Smaller values of $F_n(A)$ mean better directional coverage.

This project uses this metric to compare different 4-bit scalar formats, including:

- FP4-E2M1
- FP4-E1M2
- FP4-E3M0
- INT4
- numerically optimised 4-bit alphabets

The goal is not only to compare existing formats, but also to understand what makes a scalar alphabet good at preserving direction.

---

## Why Product Codes Have Fundamental Limits

A scalar alphabet used elementwise creates a product code. This structure is efficient and hardware-friendly, but it is also restrictive.

An unrestricted spherical code can place representable directions directly on the sphere. A product code cannot: its directions must arise from coordinatewise combinations of scalar values.

This work shows that this difference creates a fundamental gap. Product-structured codes cannot cover the sphere as well as optimal spherical codes with the same number of directions.

In two dimensions, almost every product code is strictly worse than an optimal spherical code with the same number of directions. In higher dimensions, the separation persists asymptotically: for any fixed scalar alphabet size, product codes become increasingly limited in their ability to cover all directions.

This does not mean product codes are bad. Rather, it shows that scalar number formats carry an unavoidable geometric constraint when used to represent vectors.

---

## Standard Number Formats Are Also Suboptimal Within Product Codes

The next question is more practical:

> Among product codes, are standard scalar formats such as fixed-point, two’s complement, and floating point optimal for preserving direction?

The results suggest that they are not.

Even within the class of product-structured codes, the choice of scalar alphabet matters. Standard number formats are designed around arithmetic convenience, dynamic range, and value approximation. They are not designed directly for directional coverage.

This work shows that for bit-widths $b \geq 3$, there exist scalar alphabets with better asymptotic directional coverage than the best standard $b$-bit alphabets drawn from common two’s complement, fixed-point, and floating-point families.

This motivates a different design principle for low-precision arithmetic:

> Choose scalar alphabets according to the geometry of the vector directions they induce.

---

## Optimising 4-Bit Alphabets for Direction Preservation

To explore this idea experimentally, 4-bit scalar alphabets were numerically optimised for directional coverage.

The optimisation procedure searches for alphabets that minimise the sampled worst-case angular error in a given dimension. The search combines a global optimisation stage with local refinement.

The resulting optimised alphabets were compared against common 4-bit formats across dimensions up to 64.

![Sampled worst-case angular error for 4-bit alphabets](/assets/img/worst_case_angular_error.png){: width="680" }

**Figure 2.** Sampled worst-case angular error as dimension increases. The optimised 4-bit alphabet gives the lowest angular error. FP4-E2M1 is very close to the optimised alphabet and performs much better than the other standard formats tested.

The sampled worst-case angular errors, in degrees, are:

| Datatype | d = 4 | d = 8 | d = 16 | d = 32 | d = 64 |
|---|---:|---:|---:|---:|---:|
| Optimised alphabet | 4.05 | 4.93 | 5.80 | 6.36 | 6.77 |
| E2M1 | 5.26 | 5.90 | 6.30 | 6.63 | 7.12 |
| INT4 | 6.03 | 7.39 | 9.51 | 10.3 | 11.4 |
| E3M0 | 10.4 | 10.8 | 11.0 | 11.1 | 11.1 |
| E1M2 | 15.6 | 18.5 | 21.6 | 24.6 | 24.7 |

A notable result is that FP4-E2M1, the format used in NVIDIA’s NVFP4, almost matches the optimised alphabet. This gives a geometric explanation for why E2M1 can perform so well in low-precision machine learning workloads.

It is not merely a good value approximation format. It also induces strong directional coverage.

---

## Log-Space Alignment with the Optimised Alphabet

To understand why E2M1 performs so well, the standard alphabets were compared against the optimised alphabet in log-space.

The following plots show log-log regressions in 16 dimensions. A slope of one corresponds to a form of directional isometry: the standard alphabet aligns closely with the optimised directional structure.

![Log-log alignment of E3M0 with the optimised alphabet](/assets/img/log-log-e3m0.png){: width="500" }

**Figure 3.** Log-log alignment between E3M0 and the optimised alphabet.

![Log-log alignment of E1M2 with the optimised alphabet](/assets/img/log-log-e1m2.png){: width="500" }

**Figure 4.** Log-log alignment between E1M2 and the optimised alphabet.

![Log-log alignment of E2M1 with the optimised alphabet](/assets/img/log-log-e2m1.png){: width="500" }

**Figure 5.** Log-log alignment between E2M1 and the optimised alphabet. E2M1 aligns especially closely with the optimised alphabet, suggesting that it acts like an efficient floating-point rounding of the direction-preserving code.

![Log-log alignment of INT4 with the optimised alphabet](/assets/img/log-log-int4.png){: width="500" }

**Figure 6.** Log-log alignment between INT4 and the optimised alphabet.

Among the tested formats, E2M1 shows the strongest alignment with the optimised alphabet. This helps explain its low angular error in the worst-case coverage experiment.

E2M1 can be viewed as a computationally efficient approximation to a more geometrically optimised 4-bit scalar alphabet.

---

## Implications for Low-Precision Machine Learning

This work points towards a geometric way of thinking about number-system design.

Instead of asking only whether a scalar format approximates real numbers well, we can ask:

- What directions does this scalar alphabet generate?
- How evenly do those directions cover the sphere?
- How large is the worst-case angular gap?
- Can the scalar alphabet be redesigned to improve directional fidelity?
- Are existing formats already close to good direction-preserving alphabets?

From this perspective, FP4-E2M1 is especially interesting. It is not the exact optimum found by numerical search, but it is very close. It appears to offer a strong trade-off between geometric fidelity and efficient floating-point implementation.

---

## Summary

This project develops a geometric framework for understanding low-precision scalar number formats as direction-generating systems.

The main conclusions are:

1. Elementwise scalar quantisation creates product-structured direction codes.
2. Product codes have fundamental limitations compared with unrestricted spherical codes.
3. Standard number formats are suboptimal for direction preservation, even within the product-code class.
4. Numerically optimised 4-bit alphabets can improve worst-case angular coverage.
5. FP4-E2M1 is surprisingly close to the optimised 4-bit alphabet, providing a geometric explanation for its strong empirical performance.

The broader lesson is that future low-precision formats for machine learning should be designed not only around scalar value approximation, but also around the vector directions they preserve.