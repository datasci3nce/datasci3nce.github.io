---
layout: post
title: "The Logic Ring: Causal Geometry in an Alg-Zoo RNN"
description: "Refuting the sparse circuit hypothesis using topological data analysis and causal ablation."
date: 2026-04-20
tags:[Mechanistic Interpretability, Dynamical Systems, Topology]
comments: false
share: false
---

**Refuting the Sparse Circuit Hypothesis**  
In mechanistic interpretability, we often search for discrete, sparse subgraphs. However, when auditing dense, continuous tasks—such as the 432-parameter Alg-Zoo RNN trained on a continuous `2nd-argmax` task—sparse circuit discovery hits a wall. The logic appears smeared across all neurons.

Through the **Topological Circuit Audit (TCA)**, I demonstrated that gradient descent on continuous analog tasks does not favor sparse graphs; it favors distributed geometric attractors. 

By analyzing the network as a piecewise-linear dynamical system, I identified the "Logic Ring"—a 1-dimensional persistent homology (an invariant center manifold). I causally verified its load-bearing nature using a controlled "Topological Pinch Test," showing that orthogonal perturbations collapse the model's accuracy. 

Most crucially, I derived a mechanistic error bound of 3.44% derived entirely from the manifold's geometric resolution limit, perfectly matching empirical performance. For continuous reasoning, the fundamental unit of computation is the manifold, not the neuron.
