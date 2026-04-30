---
layout: post
title: "Seeing the Shadow: A Game-Theoretic Audit of Representational Erasure in LLMs"
description: "Why standard unlearning fails, and how backup coalitions preserve latent knowledge."
date: 2026-04-30
tags: [Machine Learning, AI Safety, Mechanistic Interpretability]
comments: false
share: false
---

**The Problem with Unlearning**  
A central open problem in AI safety is that "machine unlearning" often fails. When we try to remove hazardous knowledge—like a backdoor or a typosquatting vulnerability—the model appears behaviorally compliant. Yet, adversarial fine-tuning can recover the "erased" concept with fewer than 10 examples. Why? 

My recent research suggests that standard interpretability and unlearning methods suffer from a "sparsity illusion." They assume features are single linear directions. I hypothesized that concepts are actually highly redundant coalitions of directions, and unlearning merely suppresses the dominant axis while leaving a latent "backup coalition" intact.

**The Game-Theoretic Audit**  
To prove this, I conducted a geometric audit of Qwen-2.5-1.5B on a router-mediated typosquatting task. Using iterative orthogonal depletion, I extracted 30 orthogonal probe directions that separate the concepts. 

To measure the true causal weight of these directions, I treated the 30-dimensional subspace as a cooperative game. I computed exact Shapley Values (marginal AUC contribution) and Banzhaf Power Indices (swing probability) across training checkpoints. 

**The Shape vs. The Shadow**  
The game-theoretic audit revealed a sharp representational phase transition at just 50 training samples:
1. **The "Swing Voter" (The Shape):** A single dominant direction crystallized, accounting for 92% of the classification signal. 
2. **The "Backup Coalition" (The Shadow):** The remaining 29 orthogonal directions formed a highly redundant backup coalition. In the base model, they possessed near-zero Shapley value. However, their Banzhaf swing potential remained entirely intact.

**Implications for AI Safety**  
When standard unlearning (like gradient ascent or representation steering) is applied, it successfully collapses the "Swing Voter." Behaviorally, the model looks safe. However, the unlearning process is completely blind to the "Shadow." The backup coalition survives the intervention, preserving the relational topology of the hazardous concept in a latent state. 

True representational erasure cannot be verified by behavioral compliance alone. It requires tracking the collapse of these backup coalitions—a structural invariant that game-theoretic component auditing makes visible.
