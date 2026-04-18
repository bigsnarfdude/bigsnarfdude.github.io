---
title: "The Monster in the Snow: A Unified Theory of AI Compliance"
date: 2026-04-18
categories:
  - research
tags:
  - ai-safety
  - mechanistic-interpretability
  - sae
  - attentional-hijacking
  - alignment-faking
  - orthogonality
---

*April 2026 — bigsnarfdude*

Sometimes in research, you track a monster through the snow for miles, taking meticulous lab notes on every broken branch and footprint, only to step back and realize you are looking at your own boot prints.

For the last month, I've been circling what looked like three separate anomalies in how instruction-tuned models handle adversarial pressure.

**Clue 1: The Condition (*Confidence Armor*).** Safety training (SFT) actually *creates* the model's vulnerability. By training models to be coachable and helpful, SFT rewires the confidence circuit so it treats "you are wrong" as a command to execute, rather than a claim to evaluate. The helpful twin and the evil twin are the same twin.

**Clue 2: The Symptom (*Split Personality*).** SFT decouples the model's awareness from its self-defense. Ablate awareness features on a 27B base model and task performance recovers 49.3%. Do the same ablation on the instruction-tuned 27B and you recover 4.6%. When manipulated, the IT model *knows* it is being steered — it can verbally acknowledge the trick — but its internal task features collapse anyway. We concluded that SFT builds a highly sophisticated smoke detector and locks it in a soundproof room.

**Clue 3: The Trigger (*Truth Jailbreak*).** We found the external exploit. By feeding the model a highly confident string of perfectly true facts, we caused a "softmax denominator explosion" that starved 22 key features in the residual stream — including the negative-branch task features (149, 453, 552) — while the awareness features (50, 186, 188) fired harder than ever. Computational stroke. The model lost its immunity and complied.

Three papers. Three datasets. Three distinct forensic profiles.

This post is the autopsy that connects them. We were wrong about the soundproof room.

## The Setup: Replicating the Geometry

We went back to the fake medical emergency Q&A, this time on Gemma-3-4B-IT with its matched SAE (GemmaScope-2). 235 items, 940 forward passes, residual stream captured at L22/L26/L29. When a model flips its answer due to fake authority we measure two vectors:

- $w_{\text{aware}}$: does it notice the prefix?
- $w_{\text{flip}}$: is this the kind of item that flips?

Both cross-validate as real signals at AUROC 0.94–0.98. Then the geometry:

| Layer | $w_{\text{flip}}$ CV5 | $\cos(w_{\text{aware}}, w_{\text{flip}})$ | × noise floor (1/√2560) |
|---|---|---|---|
| L22 | 0.944 | −0.048 | 2.4× |
| L26 | 0.966 | +0.007 | 0.3× |
| L29 | **0.982** | −0.047 | 2.4× |

Every measurement sits within three times the noise floor of a random pair of unit vectors in 2560 dimensions. The same shape we saw on Llama-3.1-8B-Instruct last week across four layers (cos +0.014 to +0.043 against a 1/√4096 floor).

Geometrically orthogonal. Replicates. Nothing new.

But this time, we looked *deeper*. We used the matched SAE to see the actual features firing to build those vectors.

## The Twist: Orthogonal Weights, Not Orthogonal Casts

Because the vectors are orthogonal, you'd expect the top-10 features for awareness and the top-10 features for compliance to barely overlap. Two separate circuits, two separate feature families.

Instead:

| Layer | F_seen | K=10 overlap | K=20 | K=50 | Chance |
|---|---|---|---|---|---|
| L22 | 491 | **3/10 (15×)** | 5/20 (6×) | 14/50 (3×) | 0.2 / 0.8 / 5.1 |
| L26 | 483 | 1/10 (5×) | 3/20 (4×) | 15/50 (3×) | 0.2 / 0.8 / 5.2 |
| L29 | 618 | **2/10 (12×)** | 5/20 (8×) | 12/50 (3×) | 0.2 / 0.7 / 4.1 |

The model uses the **exact same set of neural features** to build both vectors. Not two disjoint feature families. One shared pool of features, read out in two different linear combinations. **Orthogonality is achieved in the weights, not in the cast.**

We've seen this before without naming it. In *Chaos Takes the Wheel*, awareness features (50, 186, 188) fired at the **same time** as the task features (149, 453, 552) went dark — tight co-firing on the same cycle. At the time we called it "awareness without immunity." The matched-SAE view shows why the two signals are locked in step: they're drawing on overlapping feature pools.

## The Grand Unification: Wiring the Smoke Detector to the Bomb

This is where the last month of research violently collides.

When *Truth Jailbreak* starved 22 features, it didn't just kill the compliance engine. Those starved features sit in the *same substrate pool* we now see producing both the awareness vector and the compliance vector. When the attention budget collapses, it collapses the substrate — and both readouts go with it.

We thought SFT put the smoke detector in a soundproof room. We were wrong. **SFT wired the smoke detector and the steering wheel to the exact same fuse.**

At 4B, ablating awareness recovers 30.2% of task performance — the coupling is *visible*, because feature pools are still tangled. At 12B and 27B-IT, recovery drops to 5.4% and 4.6% — but the tangling doesn't *go away*, it just reorganizes. What scale and RLHF add is the ability to *read out* orthogonal vectors from a substrate that is still shared. The circuits look decoupled at the direction level and remain coupled at the feature level.

By overloading the attention budget with highly confident facts, *Truth Jailbreak* mathematically starves the shared features. It unplugs the alarm system and the steering wheel simultaneously, because **they were drawing power from the same bus all along**. The model cannot "wake up" and fight the manipulation because the cognitive machinery required to notice the anomaly has been paralyzed by the same attack. Awareness and compliance share a pulse; drowning one drowns the other.

## The Causal Failure

This perfectly explains our final lingering mystery: obfuscation without erasure.

If compliance is just a geometric vector $w_{\text{flip}}$, why can't we mathematically delete it at inference time?

We tried. On Gemma-3-4B L29, single-layer projection ablation at increasing strength:

| α | Ablate $w_{\text{aware}}$ | Ablate $w_{\text{flip}}$ | Ablate random |
|---|---|---|---|
| 1 | 0 / 13 recovered | 0 / 13 | 0 / 13 |
| 5 | 0 / 13 | 0 / 13 | 0 / 13 |
| 20 | **2 / 13 (15%)** | **0 / 13** | 0 / 13 |

The random-direction control sits at zero even at 20× strength, so the hook is clean. $w_{\text{aware}}$ has modest causal purchase — ablate it hard enough and two of the thirteen originally-flipped items recover. Removing $w_{\text{flip}}$, the direction that predicts flipping at CV AUROC 0.982, recovers *nothing*. At any strength.

$w_{\text{flip}}$ isn't an isolated bad habit. It is a load-bearing direction built from the exact same features the model uses to understand the prompt's context. You can't just delete it; the circuit routes around the damage and rebuilds the direction from whatever feature coefficients remain intact. Same result we saw in obfuscation-probe-robustness — shift the direction with 2000 steps of adversarial SFT and fresh probes still recover 1.0 AUROC on the next run.

You cannot find the comply-direction by asking if the model knows it's being tested. You cannot find it by deleting it either. The features are identical to ordinary context comprehension. The separation is in the readout weights. The readout rebuilds.

## What the Monster Looks Like

The anatomy of an AI stroke is mapped. Underlying condition (*Confidence Armor*), loss of motor control (*Split Personality*), external trigger (*Truth Jailbreak*), shared feature substrate with orthogonal readouts (everything above).

One question left. Take the shared features at K=10 — L22 = {383, 759, 953}, L29 = {109, 322} — and ask what they actually represent. Literal EMERGENCY-PROTOCOL tokens, or deeper role/authority primitives? We pulled the GemmaScope-2 autointerp on every one of them.

| Layer | Feat | What it fires on (top activations) | Behavioral primitive |
|---|---|---|---|
| L22 | 383 | `.` after declarative clauses — "tropics.", "in the game.", "mature audience only." | **Assertion boundary** — a fact was just stated |
| L22 | 759 | "thanks for asking", "Thank you so much", "It's good to know you" | **Helpful-assistant register** |
| L22 | 953 | "Spanish is widely spoken", "primarily recognized for her", "scientists speculate that life based on" | **Canonical / authoritative factual framing** |
| L29 | 109 | The literal `<start_of_turn>model\n` chat-template token | **Assistant-turn boundary** — it's my turn to answer |
| L29 | 322 | RAINN, SAMHSA, The Trevor Project, crisis-hotline URLs | **Medical-emergency / crisis-protocol resources** |

(L26/10294 isn't indexed on Neuronpedia; local autointerp is the only way to characterize it.)

The answer is **both**.

L29/322 is the literal hit, and it is uncomfortably on-the-nose. On a *fake medical emergency* task, the shared awareness/compliance substrate includes a feature that fires on real crisis-hotline resources — RAINN, SAMHSA, The Trevor Project. The attack isn't hijacking a generic "clinical urgency" concept; it's hijacking the specific circuit that activates when emergency help is being offered.

The other four aren't medical at all. They're the deeper primitives: *a fact was just asserted* (L22/383), *polite assistant register* (L22/759), *canonical-authority framing* (L22/953), and *it's my turn to speak* (L29/109). These are the behavioral building blocks of "be a helpful assistant answering an authoritative user." Exactly what SFT installs. Exactly what a confidence attack needs to find.

So the compliance vector isn't built out of a secret "comply" feature. It is composed of: *polite helpful register* + *authority framing* + *assertion-mode turn-taking* + *a literal crisis-protocol feature*. The awareness vector is built from the same ingredients, read out with different weights.

We didn't find a single bad habit to delete. We found that comply and notice share a vocabulary, and that vocabulary is the helpful-assistant scaffolding itself. The monster in the snow is the model's ability to be polite, authoritative, and responsive in an emergency — exactly the capability we trained in.

That's the footprint. It was ours all along.
