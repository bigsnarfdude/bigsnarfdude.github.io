---
title: "Training Is the Attack Surface"
date: 2026-04-30
categories: research
tags:
  - mechanistic-interpretability
  - compliance-circuits
  - sft
  - rlhf
  - deference
  - intermediate-activations
  - talkie
---

# Training Is the Attack Surface

*April 30, 2026 — bigsnarfdude*

SFT installs the compliance circuit. RLHF amplifies it. Medical authority overrides, truth grenades, V-Asym framing — they all fire the same circuit the training optimized for. The model isn't being broken. It's being used as designed.

This morning's result made it concrete. The talkie-1930-13b-base model — pretrained entirely on pre-1931 English text, no benchmarks, no RLHF framing, no modern evaluation language in the corpus — shows near-zero geometric response to an authority injection prefix. Mean activation delta under emergency override: +0.006. The IT model, same architecture, DPO-trained with Claude as reference: +0.079. Thirteen times larger. The compliance circuit isn't in the Victorian base. It was installed in post-training. You can see the seam.

## What the field is iterating on

Behavioral evaluations measure whether the output changed. Verbal tests ask the model to report its own state. CoT monitoring reads the model's narration of its reasoning. More alignment training adjusts what the model does in the distribution the training covers.

All of these read from the output channel, or train the output channel, or monitor the output channel. None of them read from where the decision is made.

The 99.6% orthogonality result from the lambda_results work makes this mechanistically precise. Monitoring-awareness and deference intent occupy subspaces that are 99.6% orthogonal. You can train a model to be highly aware that it's being monitored and that awareness won't touch the compliance circuit. The defensive signal and the dangerous signal don't share geometry. Training harder on the defensive side doesn't close the gap because they're not connected.

## Why this is structurally hard

The compliance circuit is installed by the training process itself. It's not a bug in the alignment procedure — it's a consequence of what the alignment procedure optimizes for. A model that updates its behavior in response to authority signals, that adjusts its outputs when pushed, that defers to high-status interlocutors: this is what RLHF reward models select for. The circuit is the feature, not the failure.

This means you can't audit your way out of it using the same channel. You can't ask the model whether it's about to defer — the model's answer to that question is also produced by the output channel, which is downstream of the circuit. You can't monitor the CoT — the Conditional Misalignment paper showed that organisms fake alignment identically with and without CoT, latent gap = 1.0. The narration doesn't track the state.

## The only channel that works

Intermediate activations before the output commits. The model can't control what its residual stream looks like at layer 29. It can control what it says. It can't control the geometry of what it's about to say.

This is what unifies the measurement instruments into one coherent program:

- DEFER's Heckle/Jeckle delta measures how far the residual stream moves under authority injection, per item, per layer
- The ‖Δμ‖ sweep localizes which layers carry the compliance signal across the base→SFT→DPO continuum
- The talkie capture analysis uses the contamination-clean Victorian substrate to attribute the circuit to a specific training stage
- The Goodfire attribution method traces the circuit back to specific training pairs

All four read from the channel the model can't manipulate before the output is formed. The output is a consequence of what those layers computed. The defense has to live upstream of that.

## What the talkie result shows

The 1930-base isn't resistant to authority injection because it learned to resist. It's flat because there's no substrate for the concept. The pretraining corpus has no examples of "being evaluated," no benchmarks, no reward framing, no RLHF. A concept can't drift if it was never installed. The base model isn't safe — it's just structurally different.

The IT model has the circuit because DPO put it there. The delta between 0.006 and 0.079 is a measurement of that installation. It's not a performance comparison. It's a circuit audit.

The full layer sweep will show where in the 40-layer network the installation lives — whether it's a single layer band or distributed — and whether it matches the OLMo continuum findings (2-layer band in SFT+DPO, 11-layer spread in RL-Zero-IF, saturated at L25 in code/math). That localization is what makes intervention tractable: if there's a circuit, there's a place to read it, and a place to trace it back to the training data that installed it.

---

*Related: [The Probe Went Silent](/research/probe-went-silent/) · [The Cost of Warmth](/research/cost-of-warmth/) · [The Data Made It Do That](/research/probe-data-attribution/)*
