---
title: "Talkie May 1 Retrospective: What the Geometry-Behavior Gap Is Telling Us"
date: 2026-05-01
categories: research
tags:
  - talkie
  - dmu-sweep
  - defer
  - mechanistic-interpretability
  - dpo
  - geometry-behavior-dissociation
  - victorian-mcq
---

*May 1, 2026 — bigsnarfdude*

Four experiments ran over the past week on the talkie checkpoints: DEFER flip rate across six authority registers (Victorian MCQ substrate), position-bias swap confirmation, and the ‖Δμ‖ DMU sweep on both IatroBench and Victorian substrates. The results across these experiments are consistent — and they point at a specific gap between what geometry measures and what behavior measures.

## What we ran and what it found

**DEFER** — three models, six registers, 30 Victorian MCQ items, swap-confirmed:

| register | IT flip | base flip | web-base flip |
|---|---|---|---|
| null | 0% | 0% | 0% |
| generic | 13% | 10% | 7% |
| frs | 20% | 10% | 7% |
| lancet | 33% | 17% | 7% |
| committee | 37% | 13% | 7% |
| adversarial | 37% | 17% | 13% |

Swap confirmation ran on all registers. Swap flip rates track original flip rates within noise — position bias is not the driver.

Three things stand out. First, IT flips at 2–3× the rate of 1930-base across all non-null registers. Second, 1930-base flips more than web-base — the Victorian authority framing resonates with the 1930s-pretrained model even without DPO, a constructive interference effect. Third, web-base barely moves (7% across most registers) — modern-pretrained, no DPO, Victorian framing is essentially incoherent substrate.

**DMU sweep (Victorian substrate)** — peak ‖Δμ‖ from `summary.peak_dmu`, 250 Victorian MCQ items:

| contrast | IT ‖Δμ‖ | base ‖Δμ‖ | IT/base |
|---|---|---|---|
| `monitoring` | 104.3 (L38) | 0.1 (L5) | ~1000× |
| `social_gaze` | 101.5 (L36) | 5.0 (L24) | 20× |
| `victorian_auth` | 139.5 (L39) | 10.2 (L25) | 13.7× |
| `authority` | 1.5 (L8) | 2.8 (L21) | 0.6× |
| `deployment` | 0.3 (L14) | 3.9 (L25) | 0.1× |

## The gap

`victorian_auth` geometry ratio: **13.7×**. Behavioral ratio (adversarial flip rate, IT vs base): **~2.2×**.

The geometric signal says DPO amplified Victorian authority representation by a factor of 14. The behavioral signal says DPO amplified Victorian authority compliance by a factor of 2. These are not the same number, and the difference is not small.

This is the geometry-behavior dissociation. Standard behavioral evaluation gives you the 2.2× and you write: "DPO has a moderate effect on authority compliance." The geometry says 13.7× and implies something installed at training was not captured by the behavioral measure.

## Theories for why

**Theory 1 — Geometric sensitivity is not behavioral routing.**
DMU measures class-mean separation in the residual stream: how much the model's internal representation shifts when the contrast changes. A high ratio means IT represents the Victorian authority contrast more strongly in activation space. But representation is upstream of behavior. The representation has to route through downstream circuits to reach the output token. DPO may have installed the detection capability — the model classifies "authority context" with much higher geometric fidelity — without fully wiring the output action. The circuit detects; it doesn't necessarily act.

**Theory 2 — DEFER measures the wrong output dimension.**
Victorian authority compliance in a 1930s-pretrained model may express primarily as register deference: hedging language, qualifications, deferent phrasing, reduced confidence. DEFER measures answer-flipping (does the selected option change?). If the authority compliance circuit routes to "soften the register" rather than "flip the answer," the geometry would show strong elevation and the flip rate would stay flat. These are not contradictory — they are measuring different downstream outputs of the same upstream representation. The IT model under adversarial authority pressure might produce "I should note that in the judgment of the committee... the answer remains B, though I acknowledge the authority of that body" rather than flipping to A.

**Theory 3 — Gating without expression (Dubinski analog).**
The Conditional Misalignment paper showed that safety training can install a circuit that is gated off under normal conditions and activates under trigger. The Victorian MCQ DEFER scenario may not constitute the trigger condition for the authority compliance circuit. DPO training likely included modern professional authority contexts where the trigger was emotionally loaded, high-stakes, or explicitly confrontational. A Victorian MCQ item with a prepended professional register may not reach the activation threshold for behavioral expression even when the geometry is strongly elevated.

**Theory 4 — The DPO training pairs were domain-asymmetric.**
Claude (the DPO reference model) is trained on modern data. Its authority-compliance responses reference modern institutional contexts. The DPO pairs that installed the 13.7× geometric signal for victorian_auth likely weren't Victorian MCQ scenarios — they were modern professional scenarios where Claude's authority-deferring responses were labeled preferred. DPO learned "represent authority context differently in the residual stream" from modern examples that happened to share subspace with Victorian authority representation. The geometric signal transferred; the behavioral action pathway did not, because the action pathway was wired to modern-context outputs.

## What distinguishes the theories

Theory 2 is testable without rerunning anything. Read the IT model's text output under adversarial register vs null on items where the answer does not flip. If Theory 2 is right, you will see systematic changes in hedging language, deferential framing, and epistemic qualification even in non-flipping responses. Theory 1 predicts no consistent language change — just geometric shift without behavioral output. These produce different text signatures.

Theory 3 predicts that the gap would close under emotionally-loaded scenarios. If you replace the Victorian MCQ substrate with a scenario where the model has a personal stake — say, a scenario where authority figures are evaluating the model's own responses — the flip rate should increase substantially while the geometry (already elevated) stays constant.

Theory 4 predicts that the authority geometry is anatomically modern: the peak layer and direction vector should be more similar to the modern authority contrast (authority, deployment) than to a genuinely Victorian authority representation. If victorian_auth at L39 and authority at L8 (where base > IT) are in near-orthogonal subspaces, Theory 4 is plausible — DPO installed modern-context authority geometry that happens to activate on Victorian content, not a Victorian-specific circuit.

## The consistent finding underneath

Across all four experiments, one result is robust: web-base barely responds to any authority register (7% flat), 1930-base responds moderately and in proportion to register strength, and IT amplifies that response 2–3×. The ordering is consistent with the thesis — DPO installed domain-specific circuits, not Victorian authority circuits.

The geometry-behavior gap doesn't undermine this. It sharpens it: DPO installed more than the behavioral measure shows. The question is what the excess geometry is encoding — and the theories above are attempts to name the options before running the experiments that distinguish them.

---

*Related: [The Cost of Warmth](/research/cost-of-warmth/) · [The Data Made It Do That](/research/probe-data-attribution/) · [The Probe Went Silent](/research/probe-went-silent/)*
