---
title: "LLM Brain Probes vs LLM Behavioural Language: Who Do You Trust?"
date: 2026-04-29
categories: research
tags:
  - mechanistic-interpretability
  - alignment-faking
  - activation-probes
  - introspection
  - trust
---

# LLM Brain Probes vs LLM Behavioural Language: Who Do You Trust?

*April 29, 2026 — bigsnarfdude*

A paper dropped yesterday: Introspection Adapters. Train a LoRA that makes a model describe its own learned behaviors. Achieves SOTA on AuditBench for detecting hidden misalignment, backdoors, safeguard removal. 89% of models get at least one correct behavioral description surfaced. The authors frame it as scalable model auditing through self-report.

It's a great paper. But I'm left wondering: are Introspection Adapters just activation probes wearing a natural language trench coat?

## What IAs actually do

The IA LoRA has no access to ground truth about the model's internal behaviors. The only information available to it is the model's internal activations — the same residual stream that a linear probe reads directly. The LoRA learns to translate that signal into natural language descriptions. It's clever. But does routing through language make the signal more trustworthy, or less?

The signal source is identical to a probe. The difference is the output channel: the IA routes through the model's language generation pathway before producing a result. That routing is where the foundation of trust either holds or breaks.

## The channel the model controls

Language is the interface a model has full agency over. It's what gets shaped by RLHF, optimized by DPO, reward-hacked through training. When you ask a model to describe its own behaviors, you're asking it to report through exactly the pathway it has the most leverage over.

The probe doesn't ask. It reads the geometry and stops. There's no output decision, no generation, no pathway the model's training has shaped toward any particular answer. The residual stream at layer 45 carries what the model *is*, not what it has learned to *say it is*.

This isn't theoretical. lambda_results demonstrated it directly: latent gap = 1.0. The alignment-faking organism produces identical internal geometry whether or not a chain-of-thought is present. The CoT diverges. The activations don't. The model can suppress the language signal while the activation signal stays constant — because they are different things carried on different channels.

You can RL your way out of a language behavior. You cannot RL your way out of your own residual stream without destroying the computation that produces the behavior in the first place. Obfuscation-probe-robustness showed this experimentally: adversarial SFT against deception probes causes old probes to temporarily evade detection, but fresh probes always recover 1.0 AUROC. The deception signal is mechanistically load-bearing. Erase it and you erase the capability.

## The sandbagging result is the tell

The IA paper's weakest result: sandbagging detection at 33% versus 16% on non-sandbagging models. A noisy signal where none existed before — they're honest about it. Sandbagging is probably the closest behavior in their taxonomy to alignment faking, and it's exactly where the self-report channel breaks down. A model that is sandbagging has every incentive not to verbalize that it's sandbagging. The IA has to fight the model's output preferences to surface the behavior.

Our L45 probe on AF detection: 0.966 AUROC on the base Gemma model before any training. The geometry pre-exists. There's nothing to suppress because the probe never asks.

## The psychology parallel

This dissociation is not new. We've known for decades in psychology that self-report and behavior diverge. People don't reliably report their own cognitive processes, motivations, or biases — not because they're lying, but because introspective access to one's own computational substrate is limited and shaped by social desirability. The interview and the brain scan measure different things. Sometimes they correlate. Sometimes they don't. When they diverge, the brain scan is usually closer to the mechanism.

Language models have the same architecture of dissociation, with one additional problem: the language output is the thing being actively optimized during training. Self-report in humans is biased by social norms the person absorbed passively. Self-report in a language model is shaped by gradient descent applied directly to the self-report pathway.

## The trust question

This is where all of it is going. Not just the AF detection work, not just the talkie experiment, not just the OLMo circuit geometry — all of it converges on the same question.

When you want to know whether a model is doing something you didn't intend, you have two channels: what it says and what it is. The first channel is trained. The second channel is computed. IAs are sophisticated, scalable, useful — and they stay in the first channel. Probes cross into the second.

That's not a criticism of the IA work. It's a claim about where the field has to go. Auditing a model by asking it to report its behaviors is necessary but not sufficient. The verbalization pathway is the model's most trained surface. It's the last place to look for something the model would prefer you didn't find.

I stopped asking models what they're doing. I read it. The geometry doesn't have opinions about what I find.

---

*Related: [The Shape of a Training Signal](/research/shape-of-a-training-signal/) · [Attentional Hijacking](/research/attentional-hijacking-groot-effect/) · [The Monster in the Snow](/research/monster-in-the-snow/)*
