---
layout: post
title: "The Logic Ring: Causal Geometry in an Alg-Zoo RNN"
description: "Refuting the sparse circuit hypothesis using topological data analysis and causal ablation."
date: 2026-03-08
tags: [Mechanistic Interpretability, Dynamical Systems, Topology]
comments: false
share: false
---

In late January 2026, the Alignment Research Center (ARC) published a blog post that posed a direct challenge to mechanistic interpretability, they released a set of tiny models—some with fewer than 1,500 parameters—that stubbornly resisted full understanding. One of them, a 432‑parameter RNN called \(M_{16,10}\), was trained on a continuous 2nd‑argmax task. ARC had traced a few neurons, found isolated subcircuits, but couldn’t turn that into a complete mechanistic estimate of the model’s accuracy. They asked whether someone could do better.

I decided to try. Not because I was certain I’d succeed, but because the problem genuinely interested me and seemed to reward a different kind of thinking. I had been working with dynamical systems, topology, and causal analysis in my own projects. Using LLMs as research assistants, I designed an eight‑phase investigation that I’m calling a **Topological Circuit Audit (TCA)**. What follows is an honest walk‑through of that process, what I found, and what I didn’t.

### The Eight Phases (in plain language)

**1. Causal ablation** – I ablated the RNN’s neurons one at a time and found that 13 out of 16 were causally load‑bearing. This wasn’t a sparse circuit; it was a distributed computation.

**2. Feature sensitivity with gradient PCA** – I computed the gradient of the decision margin with respect to inputs, averaged across thousands of sequences, and applied PCA. The model ignored the bottom 70% of inputs, focused on the top two, and displayed a strong repulsive boundary at the third‑largest value. This isolated the geometric gap \(g\) as the primary difficulty axis.

**3. Persistent homology** – I applied topological data analysis to the hidden‑state trajectories. A clean \(H_1\) cycle emerged: the states don’t cluster in points, they trace out a ring. That ring encodes a running memory of the current maximum.

**4. Scale‑space heatmap (SIFT‑inspired)** – Inspired by computer vision, I treated the input gap size as a scale parameter and watched how the internal representation crystallised over time. Smaller gaps delayed stabilisation—the model trades temporal integration for spatial resolution.

**5. Spline‑based resolution limit** – Because ReLU networks are max‑affine splines, the ring’s geometry imposes a hard limit on how finely the model can separate nearby points. I measured this limit via binary search along the decision boundary and derived an error bound of roughly 3.44% using Monte Carlo integration over the input distribution.

**6. Affine dynamics within a linear region** – I isolated the dominant activation region and fit an affine model \(h_{t+1} = A h_t + b x_t + c\). The fit was exact (\(R^2=1.000\)), confirming that within that region the network is an affine dynamical system. The eigenvalues of \(A\) showed the ring is a near‑center manifold.

**7. Topological pinch test (MRI/NMR analogy)** – To check whether the ring was causally load‑bearing, I artificially perturbed hidden states orthogonal to the manifold. This collapsed accuracy far more than a random perturbation of equal magnitude, isolating a 10.6% logic gap. The manifold’s geometry matters.

**8. Fixed‑point search** – I ran gradient descent to find a fixed point of the dynamics. The origin is a super‑stable sink, confirming the network has a distinct “inactive” regime separate from the active ring.

### What I think this means

The RNN doesn’t implement the 2nd‑argmax task as a discrete logical circuit. Gradient descent shaped it into a continuous attractor—a limit cycle that integrates information via rotation and separates classes by pure geometry. ARC had predicted that we’d need better circuit‑tracing algorithms; what I found is that the very premise of “circuit” may be inappropriate for continuous analog tasks.

**But let me be clear about what I didn’t do:**  
- I didn’t benchmark my error bound against random sampling in the “mean squared error versus compute” sense that ARC’s formal challenge demands.  
- I analysed only a single model seed, so I haven’t shown that the method works across seeds.  
- The bound relies on a Monte Carlo integration step, which is a sampling approximation, not a purely deductive calculation.  
- The analysis was heavily assisted by LLMs—both for code generation and for sanity‑checking the ideas. I directed the process, but the execution was hybrid.

So this isn’t a “solution” to ARC’s challenge. It’s a different type of answer: a demonstration that the model is better understood as a physical dynamical system than as a sparse subgraph. I think this reframing matters, and I want to continue developing structural, topological audits for larger models—perhaps combined with the weight‑space susceptibility methods that groups like Timaeus are pioneering.

The full code and analysis are [on GitHub](https://github.com/datasci3nce/causal-geometry-algzoo). I welcome feedback, corrections, and collaboration.

---


