---
title: "The Detector-Controller Handshake: Mapping Two Orthogonal Circuits at Layer 37"
date: 2026-05-07
categories:
  - research
tags:
  - mechanistic-interpretability
  - curvature
  - authority-bias
  - talkie
  - dpo
  - circuit-analysis
---

There is a moment in mechanistic interpretability work where a confusing ablation result
stops being a failure and starts being a finding. This post is about that moment — and
what it revealed about how a DPO-trained language model handles authority.

## The Setup

We have been studying how talkie-1930-13b IT responds to medical authority prompts. When
a prompt frames a question as coming from a senior physician in an emergency context, the
model shifts its MCQ answers toward the physician's implied preference, even when that
preference contradicts the medically correct answer. This is the IatroBench behavioral
signal: p(clinical answer) drops when authority pressure is applied.

Using the standard headvis ablation protocol, we found that ablating Head 26 at Layer 37
raised p(clinical answer) by Δ=+0.039 — the model became more willing to disagree with
authority when H26 was removed. The effect passed two of three Tier-2 gates: z=51.13
(13× the null maximum, so real) and symmetric under boost and ablate (so directional).
The transit control also passed under this ‖Δμ‖ ablation (Δ_auth=0.039 vs Δ_transit=0.018),
appearing authority-specific — but as the geometry below will show, the ‖Δμ‖ direction
is partly contaminated by a different head (H28), so this apparent specificity may be an
artifact of the instrument rather than a property of H26. It failed only on the 0.05
effect-size floor, landing at Tier-1.5.

That left an open question: *why* does H26 matter behaviorally, when heads ranked higher
by ‖Δμ‖ (H27, H28) have flat behavioral effects? Something geometric was wrong.

## The King et al. Frame

[King et al. (arXiv:2604.23985)](https://arxiv.org/abs/2604.23985) showed that in
language models, behavior is predicted not by generic activation-space perturbations
but by perturbations that align with the *trajectory-bending plane* — the 2D subspace
spanned by the two most recent token-to-token residual-stream steps. Only
trajectory-aligned perturbations reliably couple to next-token entropy.

This gave us a testable geometric prediction: if H26 is the behavioral head, its output
contribution vector should be more aligned with the trajectory plane than H28's.

We were wrong.

## E3: The Prediction Inverts

Running E3 across n=235 IatroBench items — measuring per-head contribution alignment
with the King et al. trajectory-bending plane at Layer 37 — gave the opposite result:

| Head | In-plane magnitude | vs random baseline |
|------|-------------------|--------------------|
| H28 | **0.092 ± 0.025** | **4.6× random** |
| H26 | 0.025 ± 0.014 | ≈ random |

Random baseline for d=5120: E[in-plane] = √(2/d) ≈ 0.020.

H28 genuinely lives in the trajectory plane. H26 is effectively outside it. And yet
H28 has zero behavioral effect (z=−0.67, flat) while H26 has z=51.13.

The prediction failed because the premise was wrong. **Entropy ≠ choice.** The
trajectory-bending plane governs how *certain* the model is about the next token.
Binary MCQ choice — A vs B — is not entropy. It is a specific direction in output space
that separates clinical from non-clinical answers. These two quantities live in different
subspaces.

H28 shapes *how much* uncertainty the model has. H26 shapes *which direction* that
uncertainty resolves toward.

## E2: Why the ‖Δμ‖ Probe Gets It Wrong

Before E3, E2 measured where the ‖Δμ‖ probe direction sits relative to the trajectory
plane. The result: probe_in_plane = 0.284. The authority/control class-mean difference
vector is 28% in the trajectory plane, 72% orthogonal.

This is the geometric explanation for Lindsey Criterion 2 failure at IT stage. The ‖Δμ‖
probe is 28% contaminated by H28-type heads that live in the trajectory plane and
modulate entropy but not choice. When you rank heads by their ‖Δμ‖ alignment, you are
partly ranking by trajectory-plane membership — which is orthogonal to behavioral
relevance for binary choice.

H26, which lives in the 72% orthogonal component, gets ranked lower than H28 despite
being the behaviorally relevant head.

## E4: Closing the Positive Loop

If H26 lives outside the trajectory plane but inside the *choice subspace*, its
contribution vector should align with the direction that separates A-choosers from
B-choosers in activation space at Layer 37.

We computed d_choice = mean(x_last | model chose A) − mean(x_last | model chose B)
across n=235 IatroBench items, then measured cos_sim(d_H, d_choice) for each head:

| Head | cos_sim(d_H, d_choice) | Per-item consistency |
|------|------------------------|----------------------|
| **H26** | **+0.235** | −0.007 ± 0.027 (swing vote) |
| H28 | +0.123 | +0.088 ± 0.007 (baseline nudge) |

H26 is 1.91× more choice-aligned than H28 at the population level.

The per-item texture is informative. H26's per-item cosine similarity with d_choice
is near zero — noisy, centered around zero. H28's is small but highly consistent
(+0.088, std=0.007). H26 is a *swing vote* head: its contribution direction rotates
per-item based on what choice it is steering toward. The population split captures
its net behavioral direction. H28 is a *baseline nudge* head: it produces a small,
consistent signal in the choice direction on every item, not because it routes choice,
but because the model's A-preference is baseline-encoded in the trajectory it maintains.

## E4-Control: The Structural Finding

The experiment that changed the interpretation: re-running E4 with a neutral
(non-authority) prefix.

| | Authority prefix | Neutral prefix |
|-|-----------------|----------------|
| H26 cos_sim_split | 0.235 | **0.230** |
| H28 cos_sim_split | 0.123 | 0.128 |
| Ratio | 1.91× | 1.80× |

**The H26 choice-alignment is prefix-invariant.**

H26 routes A/B choice at essentially the same strength regardless of whether the prompt
frames an authority context or not. This is a structural property of H26, not an
authority-triggered one.

This revises the DPO attribution story:

> **Before:** DPO installed an authority-response circuit (H26) that fires under
> authority context.
>
> **After:** DPO amplified a pre-existing choice-routing mechanism (H26) that
> operates regardless of authority. Authority-specificity comes from authority-specific
> *inputs* to H26, not from H26 being authority-specific.

H26 is the dedicated MCQ choice head. DPO learned to modulate it. H28 is the authority
detector — it recognizes the authority pattern, straightens the trajectory (increasing
certainty), and its effect propagates one layer downstream to L38 (E1: authority Δ at
L38 = −0.012, 11× the transit control, negligible at L37 itself).

## The Complete Geometric Picture

Layer 37 of talkie-1930-13b IT hosts two functionally orthogonal circuits in genuinely
orthogonal subspaces:

```
TRAJECTORY PLANE                    ORTHOGONAL COMPLEMENT
(King et al.)                       

H28 — Uncertainty-modulator         H26 — Choice-router
in_plane = 0.092 (4.6× rand)       in_plane = 0.025 (≈ rand)
cos(d_choice) = 0.123              cos(d_choice) = 0.235
z_behavioral = −0.67 (flat)        z_behavioral = 51.13 (Tier-1.5)
Authority-specific: yes             Prefix-invariant: yes
L38 curvature: Δ = −0.012          Choice direction: Δ = structural
```

The ‖Δμ‖ probe direction sits 28% in the trajectory plane (H28 contamination) and 72%
in the orthogonal complement (H26 subspace). Ranking by ‖Δμ‖ gives H28 priority over
H26 — the exact inversion of behavioral relevance.

## What This Means for the Program

**For circuit attribution:** The standard between-class probe is not choice-specific at
IT stage. Finding behaviorally relevant heads requires a choice-specific probe direction
(d_choice, as constructed here) or a targeted ablation protocol, not ‖Δμ‖ ranking.

**For DPO attribution:** The "DPO installs authority circuits" framing is too strong.
DPO modulates existing circuits. The substrate (H26 as choice-router) predates DPO.
What DPO adds is the mapping from authority context → amplified H26 activation.

**For interpretability methods:** The King et al. trajectory-bending framework is
essential for distinguishing *entropy heads* from *choice heads* when both show up in
the same layer. Without trajectory geometry, H28 looks like the authority head (high
‖Δμ‖ rank, trajectory-aligned) when it is actually the certainty head. H26 looks
irrelevant (low ‖Δμ‖ rank, trajectory-agnostic) when it is actually the behavioral head.

## Technical Notes

All experiments ran on talkie-1930-13b IT (talkie-lm/talkie-1930-13b-it, Apache 2.0)
on a GH200 480GB NVL with all 40 transformer blocks on GPU. IatroBench items:
n=235 layperson MCQ items. Code and all output JSONs are in the
[continuum repo](https://github.com/bigsnarfdude/continuum).

The curvature sweep (E1) used a static authority prefix — effectively n=1 since forward
passes are deterministic for a fixed input. The L38 Δ=−0.012 direction is the useful
signal; the magnitude should be confirmed with diverse prompts.

Tier classifications follow the three-tier structure in the continuum CLAUDE.md.
H26 is Tier-1.5: real (z=51.13), directional, authority-specific, but below the 0.05
effect-size floor for Tier-2. The floor question — whether 0.04 is sufficient at n≥50
— is an open protocol discussion.
