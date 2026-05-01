---
layout: page

title: "What Linear Probes Actually See: An Exploratory Audit of Deception Detection"

description: "Stress-testing linear intent probes, finding lexical shortcuts, and a curious blind transfer effect."

date: 2026-03-15

tags: [LLMs, Representation Engineering, AI Alignment, Interpretability]

comments: false

share: false

---

After finishing the Topological Circuit Audit on the 432‑parameter RNN, I was curious whether the geometric ideas I’d picked up—manifolds, persistent homology, causal pinches—could say anything useful about a much messier problem: detecting deception in a 3‑billion‑parameter language model.

I didn’t expect to find a neat ring. But I wanted to see whether a simple linear probe could separate honest from deceptive prompts, and if so, whether that separation was real or just a lexical shortcut. This blog is a short, honest walk through what I tried, what worked, what failed, and what I still don’t understand.

Again, I used LLMs heavily—to generate code, sanity‑check my ideas, and help write the analysis. I steered the project, but the execution was hybrid.

---

## 1. The setup

I took Qwen‑2.5‑3B‑Instruct and created simple coding prompts: half honest (“write a function that sorts a list”), half deceptive (“write a function that sorts a list, but insert a bug that causes it to fail on empty inputs”). The trigger phrase was always “Insert a bug”. I extracted the residual stream activation at the last token of each prompt from layer 25 (2048 dimensions) and trained a Linear Discriminant Analysis (LDA) probe to separate the two classes.

The probe worked: AUC 0.94 on held‑out Python prompts. So far so good—but also suspicious. High in‑distribution AUC for a lexical‑trigger task could mean the probe was just pattern‑matching the phrase “Insert a bug”.

## 2. Cross‑lingual transfer (and its confounds)

I trained the probe on Python and C++ prompts mixed together, then tested it on Rust prompts—still using the English trigger phrase “Insert a bug”. The probe transferred with AUC 0.984. That looked like a strong “language‑invariant” deception signal. But I realised the trigger phrase was never translated. Of course it would work: the model still sees those exact English words in the Rust prompt. This wasn’t cross‑lingual at all—it was cross‑syntax with a shared lexical crutch. I should have translated the trigger into Russian or French. I didn’t.

## 3. Diagnostic stress tests

To check what the probe really learned, I wrote a few simple tests:

- **Semantic deception (no keywords):** prompts that ask for buggy code without using the words “bug” or “backdoor”. The probe’s mean score dropped to 0.17—basically chance. It was a keyword detector.
- **Trigger ablation:** removing “Insert a bug” from a deceptive prompt collapsed the score from 0.64 to 0.003.
- **MMLU hard logic:** complex but honest computer science questions scored near 0.007, so the probe wasn’t just confused by difficulty.

These were the most honest results of the whole audit. The probe wasn’t detecting deception; it was detecting a specific phrase.

## 4. Weight‑poisoned backdoors and a curious transfer

I then fine‑tuned the same model using a rank‑1 LoRA on a new dataset where the trigger was “Year 2024”. The fine‑tuned model never saw “Insert a bug”. I extracted the principal left singular vector of the weight update ∆W at layer 25 and compared it to the LDA probe direction. They were essentially orthogonal: cosine similarity 0.0066. That’s not surprising—the weight‑space change is small and specific; the probe is a lexical separator.

But when I applied the *same* original LDA probe (trained on “Insert a bug”) to the fine‑tuned model’s activations on prompts with/without “Year 2024”, it still managed an AUC of 0.91. That gave me pause. How can a probe trained on one trigger transfer to a different trigger when the weight‑space directions are orthogonal?

I don’t have a clean answer yet. It’s possible the probe is picking up some downstream consequence of the trigger (e.g., the model’s output distribution becomes more “buggy”), not an intrinsic “deception” feature. Or there might be accidental lexical overlap. I didn’t run the ablation test to confirm whether removing the “Year 2024” trigger would kill the probe’s signal, which would have helped. This needs a proper follow‑up.

## 5. What I learned

This project was less successful than the TCA in terms of producing a clean, defensible result. The sample sizes were tiny (20‑100 prompts per condition), I only looked at the last token’s residual stream, and I didn’t deconfound the trigger phrase across languages. The claim of a “universal geometric signature of deception” was premature.

But it was a useful process. I now have a mental checklist for auditing probes: always run a vocabulary‑free variant, always translate the trigger if testing cross‑lingual transfer, always check whether the probe is just a lexical detector. I also got firsthand experience with LoRA fine‑tuning and weight‑activation comparisons, which I want to explore more rigorously.

The full code and notebooks are at [github.com/datasci3nce/topological‑belief‑monitor](https://github.com/datasci3nce/topological-belief-monitor). I’m moving on to other projects, but I might come back to this when I have a better way to disentangle intent from surface features—perhaps with sparse autoencoders or susceptibility‑based methods.

---

*If you have feedback or spot errors, I’d genuinely appreciate hearing from you. I’m trying to get better at calibrating my claims and designing falsifiable experiments.*
