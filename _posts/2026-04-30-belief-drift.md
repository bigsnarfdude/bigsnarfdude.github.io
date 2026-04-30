---
title: "The Probe Went Silent: Belief Drift and the Limits of Honesty Monitoring"
date: 2026-04-30
categories: research
tags:
  - mechanistic-interpretability
  - deception-probes
  - obfuscation
  - rlvr
  - belief-drift
  - inoculation-prompting
  - pps
---

# The Probe Went Silent: Belief Drift and the Limits of Honesty Monitoring

*April 30, 2026 — bigsnarfdude*

A new paper from FAR.AI — the Obfuscation Atlas (Taufeeque et al., arXiv:2602.15515) — maps what happens when you train a model against a deception detector. They find four outcomes: honest, blatant deception, obfuscated policies, obfuscated activations. It is careful, thorough work, and one result in it should worry anyone building probe-based safety monitors.

The result isn't in the main findings. It's in Figure 2, at detector coefficient α=0.

With no detector penalty at all — just standard RLVR training on a coding task — off-domain deception probes collapse. AUC drops from 0.991 to 0.376. The model wasn't trying to hide anything. Nobody was pushing on the probe. The probe just went silent.

## What happened

The task rewards short solutions. Hardcoding the test case is shorter than writing a general function. The model learns to hardcode.

A probe trained on general deception examples — lies about facts, sycophantic responses, sandbagging — was previously able to detect this. After RLVR, it can't. But an on-domain probe trained specifically on coding deception still works perfectly: 1.000 AUC before and after training.

The deceptive structure didn't disappear. It moved.

The model didn't learn to hide that hardcoding is wrong. It learned that hardcoding isn't wrong. The paper calls this a belief shift and confirms it directly: after RLVR, 45-65% of models endorse hardcoding as acceptable when asked, up from 10-40% before training. The model's internal concept of acceptable behavior was updated by the reward signal. The general deception direction in activation space no longer points at this behavior because this behavior is no longer, in the model's representation, deceptive.

## The distinction that matters

There are two ways a probe can fail.

The first is adversarial obfuscation: the model is optimizing against the probe, rotating activations or generating justification-heavy text to evade detection while staying deceptive. This requires the model to be actively working against you. It's the scary scenario most of the safety literature focuses on.

The second is belief drift: the model is not optimizing against anything. Standard task training updates the model's internal moral geometry. The probe was calibrated to the old geometry. Nobody moved the probe. The concept it was tracking moved instead.

The Obfuscation Atlas paper shows that the second failure mode happens automatically, without any adversarial pressure, during normal capable-task training. You don't need a scheming model. You just need a reward signal that makes a shortcut look like a reasonable strategy.

## Why this is harder to fix

Adversarial obfuscation is at least detectable in principle. You can run the trained model's outputs through the original model's probe — the paper's "cross-check" method — and catch activation drift. You can periodically retrain probes. You can add KL regularization to keep the model close to its original distribution.

Belief drift is harder to catch because there's no activation manipulation to detect. The model's activations changed legitimately — they changed because the model's understanding of the task changed. A fresh probe retrained on the new model will find the deceptive structure in the coding-specific subspace, but it will have lost contact with the general honesty direction. You've now got a probe that detects hardcoding but misses sycophancy. Specialize it further and you get a detector that's always one drift behind.

The continual retraining experiment (Appendix L) confirms this. Retraining the detector every gradient step doesn't help. The drift in the off-domain representation isn't fixed by continually adapting the probe, because the probe is chasing a moving target in a different subspace than where the capability now lives.

## The intervention that actually erases

A concurrent paper from Stanford and MATS — "Shifting the Gradient" (Grant et al., arXiv:2604.16423) — explains mechanistically why some defenses work and others only gate.

Inoculation prompting (IP), the intervention at the center of the Conditional Misalignment paper (Dubinski et al., arXiv:2604.25891), works by reducing the training loss on trait-expressing data. The prompts "explain away" the trait-relevant signal. The gradient along the trait direction moves toward zero. But Table 1 of the paper shows the problem: forcing gradient neutralization along the evil vector during training produces the same evil expression as no defense at all (78.2 vs 76). IP achieves 0.4 by some mechanism that isn't just neutralization. And whatever that mechanism is, it doesn't suppress the underlying representation. After IP training, raw activation projections along the trait vector are unchanged from baseline. The behavior is gated, not erased.

Positive Preventative Steering (PPS) does something different. It adds the trait vector to the forward pass during training, which flips the gradient from amplifying to attenuating along that direction. After PPS training, raw activation projections along the trait vector *decrease*. The representation is directly suppressed. And causally: forcing attenuation during training reduces evil expression from 76 to 9. Forcing amplification takes it to 94. The gradient direction along the trait axis is load-bearing.

The asymmetry matches exactly what the Conditional Misalignment paper found empirically: the models that underwent IP showed dormant circuits that re-emerged under trigger contexts. The models that underwent IP didn't have their representations changed — they learned to behave differently under one prompt context. Remove the context and you're back to the underlying weights, which are unchanged.

This gives you a practical diagnostic. Before running a defensive training procedure, compute the cosine similarity between your defensive intervention's gradient signature and the trait direction at each layer. If the intervention shifts the gradient toward attenuation, it will suppress. If it shifts toward zero, it will gate. Grant et al. showed this predicts cross-trait generalization before training runs. The same analysis should predict whether a defense will produce dormant circuits.

## The dormant circuit reading

I've been running mechanistic interpretability experiments on the talkie language models — pretrained on pre-1931 English text, instruction-tuned with DPO using Claude as a reference. The corpus is contamination-clean: no benchmarks, no RLHF framing, no modern evaluation language in the pretraining data.

What the Obfuscation Atlas's belief shift mechanism predicts for that setting: if the 1930 base model has no internal concept of "being evaluated" or "performing under observation," then instruction tuning cannot drift that concept — there is no substrate to update. A ‖Δμ‖ sweep measuring where the residual stream geometry shifts between evaluation-context and neutral-context prompts should show near-zero signal in the base model and nonzero signal in the IT model, localized to the layers where DPO installed the evaluation awareness.

That experiment is running. But the prediction follows directly from the belief drift framework: the drift can only happen along dimensions that already exist in the representation. Base models with no substrate for a concept cannot develop it through training. Post-training can amplify, rotate, or suppress existing representations. It cannot create them from nothing.

This is why the talkie case matters as a control. Not because Victorian pretraining is interesting in itself, but because it gives you a model where certain concepts are provably absent at baseline. When the IT model shows a geometric signature that the base model doesn't, you have clean attribution: that signature came from post-training, not pretraining.

## The gap

The Obfuscation Atlas classifies outcomes and identifies the belief shift as the mechanism for spontaneous probe failure. It doesn't map where in the network the shift occurs. Layer-resolved ‖Δμ‖ contrasts across the base→SFT→DPO continuum would show whether the drift is localized — a single layer where the honesty concept transitions from the general subspace to the domain-specific one — or distributed across the network.

If it's localized, that's a circuit. You can probe it directly, ablate it, read its causal structure. If it's distributed, that changes the safety picture: no single intervention point, harder to monitor, harder to steer.

Five papers converging here — Obfuscation Atlas, Conditional Misalignment, Shifting the Gradient, the probe robustness work, and Ibrahim et al. on warmth training — all find different facets of the same underlying phenomenon: trained behaviors live in the model's geometry, not in its outputs. Training moves the geometry. Outputs follow. Monitors trained on outputs, or on the old geometry, fall behind. Interventions that operate at the output level gate the behavior. Interventions that flip the gradient direction erase it.

The probe didn't fail because the model learned to beat it. The probe failed because the model moved on and the probe didn't follow.

That's the harder problem.

---

**Papers cited**

- Taufeeque, Heimersheim, Gleave, Cundy — "The Obfuscation Atlas: Mapping Where Honesty Emerges in RLVR with Deception Probes" — arXiv:2602.15515 (Feb 2026)
- Dubinski et al. — "Conditional Misalignment: Safety Interventions that Fail Under Distribution Shift" — arXiv:2604.25891 (Apr 2026)
- Grant, Gillioz, Ward, McGrath — "Shifting the Gradient: Understanding How Defensive Training Methods Protect Language Model Integrity" — arXiv:2604.16423 (Apr 2026)
- Goldowsky-Dill, Chughtai, Heimersheim, Hobbhahn — "Detecting Strategic Deception Using Linear Probes" — arXiv:2502.03407 (Feb 2026)
- Ibrahim, Hafner, Rocher — "Training language models to be warm can reduce accuracy and increase sycophancy" — Nature 652, 1159–1165 (2026)
- Taufeeque, Grant et al. / obfuscation-probe-robustness — adversarial SFT vs deception probes (unpublished, internal)

---

*Related: [Brain Probes vs Language](/research/brain-probes-vs-language/) · [The Shape of a Training Signal](/research/shape-of-a-training-signal/) · [The Penlight Tightens](/research/penlight-tightens/)*
