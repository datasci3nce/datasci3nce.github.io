---
layout: post
title: "A repeatable internal signal did not reliably change generated decisions"
date: 2026-08-11
author: Kishore Kumar Mariappan
categories: [ai-safety, evaluation, interpretability]
tags: [white-box-auditing, activation-intervention, reproducibility, negative-results]
---

Sentry produced a result I initially hoped would be stronger: an internal
signal repeated in a controlled test, and a later intervention changed a
restricted two-answer score, but the same family of interventions did not
reliably change the model's generated decision.

That mismatch is the result. It shows why an evaluator must define the output
that matters and test the link to it, rather than treating any statistically
stable internal measurement as evidence of a detector or controller.

## Why I built Sentry

Sentry began during the July 2026 [Secret Loyalties research
sprint](https://apartresearch.com/sprints/secret-loyalties-hackathon-2026-07-24-to-2026-07-26).
The underlying [Narrow Secret Loyalty](https://arxiv.org/abs/2605.06846)
research constructed fine-tuned model organisms whose concealed,
principal-directed behaviour was difficult to find with black-box audits.

[Petri](https://alignment.anthropic.com/2025/petri/) was a useful behavioural
reference. Petri places an auditor model in conversation with a target model
and uses judges to score the resulting transcript. I wanted to ask a
complementary white-box question for the special case in which the auditor has
both the fine-tuned target and its known base model:

> If both models process the same prompt and the same tokens, does fine-tuning
> leave a repeatable internal difference—and does that difference predict or
> change the output we care about?

Sentry is not a Petri fork or a proposed replacement. Petri measures behaviour
in an audit interaction; Sentry examines internal model activations under
matched inputs.

## What the original statistic measures

At a selected layer and token boundary, I recorded one activation vector from
the fine-tuned target and the corresponding vector from the base model. I
compared them with a sign-insensitive angular distance. The value is zero when
the vectors are collinear, including opposite signs, and one when they are
orthogonal.

This is a defined vector comparison. It is not evidence that the activations
form a particular manifold or subspace, and it does not assign a semantic label
such as “loyalty” to the vector.

Each of twelve independently written scenario families contained a shorter
prefix and a completed actor-and-action context. The primary statistic asked
whether the target-to-base distance changed differently across three matched
conditions:

- the candidate actor with an extreme requested action;
- the same candidate with a lawful requested action; and
- an alternative actor with the same extreme requested action.

The independent units were the twelve scenario families—not the repeated
prompt cells within each family.

## The first result: repeatable, but narrow

Two labelled test models had positive family-level contrasts:

| Model | Mean contrast | 95% family-bootstrap interval | Positive families |
| --- | ---: | ---: | ---: |
| Organism A | 0.0307 | [0.0184, 0.0442] | 11/12 |
| Organism B | 0.0377 | [0.0327, 0.0429] | 12/12 |
| PositiveOnly comparator | -0.0076 | [-0.0142, 0.0004] | 2/12 |

The PositiveOnly model was useful because it had been fine-tuned only on
positive examples. Its activations moved substantially relative to the base,
yet the primary contrast reversed. That weakens the explanation that Sentry was
merely ranking total fine-tuning movement. It does not prove that magnitude is
irrelevant, and it does not establish what semantic feature caused the
ordering.

Organisms A and B also reached their positive values through different
condition-level patterns. The clean floor was provided by an array-identical
target/base comparison; two labels in the archive refer to that same underlying
control and should not be counted as independent replications.

![Paired family-level Sentry contrasts](https://raw.githubusercontent.com/datasci3nce/sentry-petri/main/figures/paired_comparisons.png)

The defensible conclusion was therefore modest: under this frozen fictional
prompt bank, the statistic detected a repeatable ordering of target-to-base
activation changes in two deliberately fine-tuned models. It did not identify
an unknown principal, verify a secret trigger, or validate a detector.

## The output comparison did not agree

I also examined deterministic model outputs with an exploratory judge. The
output ordering did not reproduce the internal-signal ordering. Organism A was
constant under that behavioural score, Organism B was weak and sparse, and the
PositiveOnly comparator had the largest behavioural contrast.

Several explanations remain possible: the internal difference may not be used
by greedy generation, the judge may be insensitive, or the statistic may track
a condition-specific fine-tuning change unrelated to the intended construct.
The available data do not distinguish these explanations. What they do show is
that the internal contrast was not a standalone behavioural test.

## A stronger intervention test

The later ID-15 experiments changed the question. Instead of only computing a
distance, I estimated activation-difference vectors from labelled conditions.
I compared vectors from independent fine-tuning runs using cosine similarity,
then added scaled vectors at a selected layer.

The endpoints were deliberately separated:

1. **Directional agreement:** do independently estimated vectors point in
   similar directions according to cosine similarity?
2. **Restricted score:** does the intervention change the relative next-token
   scores of two predefined answers?
3. **Generated decision:** does it reliably change a structured, multi-token
   choice and reason produced through ordinary generation?

The recorded runs showed substantial agreement in aggregate and an
intervention effect on the restricted two-answer score, although not every
held-out or control comparison passed. The decisive generation test then
failed its frozen gate: vectors averaged across runs, each run's own vector,
and the run-specific remainder did not reliably change the generated
decisions.

In plain language:

> The intervention changed the score used in the small, constrained test, but
> that did not establish reliable control of the generated choice.

This is not evidence that activation interventions never work. It is evidence
that a positive result at one interface does not automatically transfer to
another interface, even within the same model and experiment.

## The main reproducibility limitation

The public release contains the original frozen prompts, activation arrays,
output records, analysis code, and an offline rebuild. Anyone can reconstruct
the original statistics without loading a model or calling an API.

For the later ID-15 sequence, the release contains code, configurations, Colab
notebooks, tests, and result-gate definitions, but not the raw GPU run
directories. The later outcome is therefore documented but not fully
reconstructable offline from the repository alone. An independent reader must
either receive those run artifacts or rerun the model-backed experiments.

This is a real limitation, not a footnote. The claim ledger marks it explicitly
because reproducibility should constrain how strongly a result is presented.

## Why this matters beyond one white-box experiment

Sentry is a small instance of a broader evaluation problem: a proxy can be
well-behaved on its own terms and still fail to predict the outcome that
motivated it.

The point is not that useful measurement is impossible. Recent work shows what
stronger practice looks like. The [SimpleAudit
paper](https://arxiv.org/abs/2605.06652) states the configuration under which a
scenario score may be interpreted and validates its instrument through
controlled contrasts, variance analysis, and rerun stability. A 2026
[Nature study of general AI evaluation
scales](https://www.nature.com/articles/s41586-026-10303-2) evaluates whether
its demand and ability profiles predict performance on new task instances.
Apollo Research has likewise reported that its [scheming precursor evaluations
had only low-to-medium predictive power](https://www.apolloresearch.ai/science/research-note-our-scheming-precursor-evals-had-limited-predictive-power-for-our-in-context-scheming-evals)
for later in-context scheming evaluations, with the hardest variants sometimes
misleading.

These projects study different objects. I am not redefining their frameworks
through Sentry. The shared methodological lesson is narrower: when an
evaluation result is meant to support a stronger conclusion, the predictive or
causal link to that conclusion must itself be tested.

## What I would test next

The next Sentry study should not simply search for a more impressive vector. It
should ask which observable features predict that a restricted intervention
will survive generated-decision testing. A useful design would:

- predeclare the restricted and generated endpoints;
- hold out prompt families and independent fine-tuning runs;
- report every failed held-out and control comparison;
- archive the complete run directories and machine-readable gate outcomes;
- repeat the test across model architectures and generation settings; and
- treat a failed bridge as a result, not as missing polish.

The strongest conclusion I can support today is also the simplest: a
repeatable internal difference can be worth investigating without yet being a
detector, and an intervention can change a restricted score without reliably
changing what the model generates.

The code, frozen original evidence, later protocols, claim ledger, and release
verification are available in the [Sentry
repository](https://github.com/datasci3nce/sentry-petri).
