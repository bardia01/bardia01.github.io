---
layout: page
title: Direction-Preserving Number Representations
description: The geometry of low-precision number formats.
importance: 0
img: /assets/img/fp4_e2m1_unit_vectors_2d.png
category: work
---

**Bardia Zadeh and George A. Constantinides** · Imperial College London

**Accepted to Asilomar 2026**

[Paper](https://arxiv.org/abs/2605.07662) · [Code and Lean proofs](https://github.com/bardia01/Direction-Preserving-Number-Representations) · [Interactive demo](https://bardia01.github.io/directional_coverage_explorer/) · [George's blog post](https://constantinides.net/2026/06/02/directionality-in-low-precision/)

As part of my PhD research, I worked with my supervisor George Constantinides to investigate how low-precision number formats preserve vector directions. In block-scaled arithmetic, a shared scale factor handles magnitude, leaving the values inside a block to encode its direction. This led us to ask: how should we choose those values to represent directions well?

We treat a number format as a finite set of scalar values, or an alphabet. Choosing each vector component from this alphabet creates a product code. Normalising its nonzero vectors gives a set of points on the unit sphere, whose largest angular gaps measure the format's worst-case directional error.

<div class="row justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/fp4_e2m1_unit_vectors_2d.png" alt="E2M1 scalar grid projected onto the unit circle, showing uneven gaps between representable directions" class="img-fluid rounded z-depth-1" caption="Directions represented by two E2M1 values. The red points are unevenly spaced around the circle, leaving gaps in angular coverage." %}
    </div>
</div>

We prove that, for any fixed alphabet size, product codes have worse worst-case directional coverage in sufficiently high dimensions than unrestricted spherical codes with the same number of raw codewords. We also establish that standard floating-point, fixed-point and two's complement alphabets are asymptotically suboptimal within the product-code class. Our theorems are formally verified in Lean, with the proofs openly available alongside the experimental code.

To explore practical block sizes, we numerically optimised 4-bit alphabets and compared them with standard formats.

<div class="row justify-content-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.html path="assets/img/max_worst_case_angular_error_deg.png" alt="Sampled worst-case angular error for optimised, E2M1, INT/E1M2 and E3M0 alphabets across dimensions 4 to 64, with E2M1 close to the optimised alphabet" class="img-fluid rounded z-depth-1" caption="Sampled worst-case angular error for 4-bit alphabets as dimension increases. E2M1, used in NVIDIA's NVFP4, closely follows the optimised alphabet." %}
    </div>
</div>

For 16-element blocks, the size used in NVIDIA's NVFP4, the sampled worst-case angular errors were:

| Format | Angular error (degrees) |
|:-------|-----------------------:|
| Optimised alphabet | 6.07 |
| E2M1 | 6.67 |
| INT/E1M2 | 9.58 |
| E3M0 | 11.0 |

These results, from [Table I of the paper](https://arxiv.org/html/2605.07662v1#S9), are estimates over one million sampled directions. E2M1 comes close to the optimised alphabet, offering a geometric explanation for its strong performance in block-scaled machine learning. The experiments also show how directional coverage can guide the design of future number formats.

I also built the [Directional Coverage Explorer](https://bardia01.github.io/directional_coverage_explorer/), where you can change a scalar alphabet and see how its representable directions cover the circle.
