---
layout: post
title: "Weight-Activation Duality in Linear Intent Probes"
description: "Auditing the Topological Belief Monitor across prompted and weight-poisoned backdoors."
date: 2026-03-15
tags: [LLMs, Representation Engineering, AI Alignment]
comments: false
share: false
---

**Auditing the Topological Belief Monitor (TBSM)**  
Can a linear probe trained to detect an explicitly prompted deception ("Insert a bug") generalize to detect a deeply embedded weight-poisoned backdoor ("Year 2024")? 

In auditing linear intent probes on Qwen-2.5-3B, I discovered a fascinating transferability phenomenon. An activation-space linear discriminant analysis (LDA) probe trained on pure prompt-based deception successfully detected a completely unseen LoRA-injected backdoor with an AUC of 0.91. 

However, a geometric analysis revealed a paradox: the weight-update direction of the LoRA backdoor ($\Delta W$) was nearly entirely *orthogonal* to the activation-space direction of prompted deception (cosine similarity 0.0066). 

This indicates that while the universal activation-space signature of deception remains invariant, the underlying weight-space mechanics of prompted vs. poisoned deception operate through fundamentally different internal pathways.
