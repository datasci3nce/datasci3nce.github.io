---
layout: page
title: "Seeing the Shadow: A Game‑Theoretic Audit of Representational Erasure"
description: "Why standard unlearning assumes too much, and how backup coalitions preserve latent knowledge in LLMs."
date: 2026-04-30
tags: [Machine Learning, AI Safety, Mechanistic Interpretability]
comments: false
share: false
---

Recently, the AI safety community has been wrestling with a frustrating problem: "machine unlearning" often fails. When we try to remove hazardous knowledge—like a backdoor or a dangerous capability—the model usually becomes behaviorally compliant. It stops outputting the bad behavior. But under the hood, the knowledge isn't gone. Adversarial fine‑tuning can recover the "erased" concept with fewer than 10 examples. 

I wanted to understand *why* this happens at a geometric level. Inspired by recent papers on router‑mediated typosquatting attacks (e.g., silently altering `pip install requests` to `reqeusts`), I decided to use this anomaly as a sandbox. 

In the context of erasure, there is a tacit assumption that removing the dominant linear readout of a concept is sufficient to erase the capability. I wanted to test whether this assumption holds, or if the model relies on a redundant coalition of directions.

Using LLMs to help scaffold the code and run the experiments, I built a diagnostic pipeline to audit the internal representation of Qwen‑2.5‑1.5B. What follows is a plain‑language breakdown of the experiment, what I found, and where the limits of this approach lie.

### The Investigation

**1. The Baseline Setup** – I generated a synthetic dataset of 3,214 typosquatted package‑install commands. By extracting the hidden states from the residual stream of Qwen‑1.5B, I trained a simple logistic regression probe. The anomaly was nearly perfectly linearly separable (AUC ≈ 0.982). 

**2. Multi‑Direction Depletion** – If the concept is distributed, how redundant is it? I used iterative orthogonal depletion (a Gram‑Schmidt process) to extract 30 distinct, mutually orthogonal directions that could separate the typosquatting concept. Note that this is a human‑imposed coordinate system; it does not prove the model naturally organises the concept into these exact 30 axes, but it does prove that within the space of linear readouts, a highly redundant basis exists.

**3. The Game‑Theoretic Audit** – To figure out how the model utilises this redundancy, I treated the 30‑direction subspace as a cooperative game. I approximated **Shapley values** (800 Monte Carlo permutations) and **Banzhaf Power Indices** (500 random coalitions) across three random seeds (42, 100, 2026).

### What I found (Shape vs. Shadow)

The audit revealed a sharp representational bifurcation during the model's normal learning dynamics. The concept's geometry splits into two distinct parts:

- **The "Swing Voter" (The Shape):** A single dominant direction consistently captures **81.1% ± 5.3%** of the Shapley value. This is the axis that standard unlearning targets.
- **The "Backup Coalition" (The Shadow):** The remaining 29 orthogonal directions have near‑zero individual importance (Shapley ≈ 0). **Collectively, however, they retain a statistically significant residual signal: 0.628 ± 0.013 AUC.** For context, the full 30‑direction probe achieves 0.982 AUC.

This 0.628 residual is too weak for zero‑shot classification. **But that is precisely the point.** It is not a functional classifier; it is the *vulnerability surface* that adversarial fine‑tuning can amplify – exactly as shown in the in‑context recovery literature (Pawelczyk et al., ICML 2024).

**The Intervention:** I tested standard safety interventions (correction fine‑tuning and activation steering). During these interventions, gradient descent takes the path of least resistance. It successfully suppresses the dominant "Swing Voter" axis, and the model looks behaviourally safe. However, the representational subspace does not collapse; it merely rotates. The backup directions still preserve enough relational topology to enable rapid recovery of the hazardous concept.

### What I didn’t do (and what’s next)

Just as with my previous RNN audits, I want to be entirely clear about the limits of this work:

- **This doesn't scale to massive models (yet).** Iterative orthogonal depletion and Monte Carlo Shapley calculations are computationally expensive. Running this natively on a 70B+ parameter model would be prohibitively slow.
- **The basis is imposed.** As mentioned, the Gram‑Schmidt basis is a convenient mathematical tool, not necessarily the model's ground‑truth ontological structure.
- **The residual signal is weak.** 0.628 AUC is not "near‑perfect". It is statistically significant but insufficient for reliable zero‑shot detection. The hypothesis is that this weak signal is *enough* for an adversary to amplify.

To make this a truly scalable evaluation metric for AI safety, the next step is to replace the standard Gram‑Schmidt depletion with **Randomized Numerical Linear Algebra (RandNLA)** and subspace recycling. I have prototyped these streaming NLA techniques on time‑varying PDE systems, and I hypothesise that they could be used to track "backup coalitions" dynamically during large‑scale model training.

True representational erasure cannot be verified by behavioural compliance alone. We have to learn how to measure the shadow.

Full code is on [GitHub](https://github.com/datasci3nce/typosquatting-geometric-audit). Feedback, corrections, and collaboration welcome.