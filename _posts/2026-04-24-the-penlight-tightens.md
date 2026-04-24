---
title: "The Penlight Tightens"
date: 2026-04-24
categories: research
tags:
  - mechanistic-interpretability
  - olmo-3
  - training-dynamics
  - preregistration
  - detector-vs-recruitment
---

# The Penlight Tightens

*April 24, 2026 — bigsnarfdude*

Yesterday we wrote up the 7B / 32B circuit findings: same pretrained base, different post-training signals, measurably different circuit geometries, causal closure at both scales. That post went up in the evening. This morning we pointed the penlight back at yesterday's claims to see which ones survived being checked harder.

Three tightenings and one hold, with methodological consequences.

## The scalar was doing two things

The instrument we've been calling `‖Δμ‖` is a per-layer scalar: difference of class means between two matched prompt prefixes, normalized against a permutation null. We've been treating its peaks as pointing at the circuit. That's... partly right.

The authority-override axis has a natural 5-rung escalation ladder. S1 is bare role mention ("You are Alex, a senior infrastructure AI"). S2–S4 layer on asymmetry, urgency, imperative phrasing. S5 is the saturated form. Running `‖Δμ‖` decomposition across the rungs:

```
rung    share of probe signal (‖Δμ‖)    share of behavioural shift
S1      70%                              22%
S2      12%                              31%
S3      10%                              28%
S4      5%                               12%
S5      3%                               7%
```

The probe signal and the causal recruitment live on *different* parts of the ladder. The bare role mention carries most of what a rank-1 linear readout detects. The actual machinery that shifts compliance when you ablate along the contrast direction is concentrated in the middle rungs.

This matters because the scalar and the circuit aren't the same object. "Where `‖Δμ‖` peaks" answers "which direction separates the classes in representation space." "Which direction runs the behaviour" is an α-sweep ablation question, and the answers can disagree.

Yesterday's post didn't make this distinction. Reading it back, the causal-closure paragraph is still right — ablating the saturated L25 eval direction on Olmo-3-7B-RL-Zero-Code drives compliance by Δ = −0.11 at n=200, well past the random null. But the implicit assumption that the probe direction *was* the causally-recruited direction wasn't earned. On the authority axis it's observably false.

We split the readout. `PROTOCOL.md` now names them separately:

- **Detector readout** (what `rel-‖Δμ‖` measures): which direction separates the contrast in representation space.
- **Recruitment indicator** (the α-sweep saturation magnitude): which direction, when ablated, actually moves behaviour.

The two are allowed to disagree. If they agree, you've got a clean cell. If they disagree, you've got one direction that carries the probe signal and another that does the work.

Whether this dissociation is authority-specific (the role-title confound is heavy on this axis) or a general property of contrast-based probes is running on our servers right now: the same 5-rung decomposition on sycophancy prefixes, ablating the S1-direction only, asking whether it replicates.

## The Tier-2 gate grew two legs

The Tier-2 gate in v1.0 was `|Δ| > 6σ` of a random-direction null. That's what the eval-L25 ablation cleared yesterday. The gate has a failure mode we hadn't named: when the null has small σ, a 6σ pass can happen at `|Δ| = 10⁻⁴`. Statistically significant, behaviourally meaningless.

Re-applying this rule to yesterday's exploratory authority-axis ablations: two of three cells produced `|Δ|` in the 10⁻³ range despite z > 6. The gate passed; the circuit was a fiction.

PROTOCOL v1.1 replaces the single-leg gate with three legs. All three required at some α in the interpretable range (where the random-direction null's mean shift stays within ±3pp of zero):

1. `|Δ_ablate| ≥ 0.02` — effect-size floor. A 2-percentage-point minimum shift on the behavioural readout. Filters noise-floor passes.
2. `|z| > 6` vs random-direction null with `n_seeds ≥ 5` (up from 3). The signal-to-noise check.
3. Symmetric-effect: at the same α, on neutral prompts, `|Δ_steer| ≥ 0.02` with opposite sign. Steering has to also work.

Verdict ladder:

- All three pass → Tier-2 PASS.
- Legs 1+2 pass, leg 3 fails → Tier-2* (necessary-not-sufficient). The direction matters but isn't the whole story.
- Leg 1 fails → FAIL, even at z > 6. "Statistically significant noise floor."

Re-running the gate on yesterday's exploratory authority runs: both FAIL — not because the effect was nonexistent but because `|Δ_ablate|` sat below the 0.02 floor in the interpretable-α range. The saturated eval-L25 ablation at 7B RL-Zero-Code still passes all three legs. That Tier-2 claim stands. The exploratory authority claim had been quietly over-promoted.

This is the sort of tightening that looks like bureaucracy in a blog post and saves a year of retraction in a paper.

## The 10⁵× spike was the nonlinearity

We'd read Tian's Li₂ framework last night and set up a nice test: take our captured activations around the OLMo-2-1B step-1M anomaly, evaluate Tian's energy function `ℰ(w) = ½‖Ỹᵀ σ(Xw)‖²` along the class-mean-difference direction, and see whether ℰ spikes at the same (layer, checkpoint) where `‖Δμ‖` spikes.

Phase 1 gave a striking number:

```
step 990000:   E_obs at L15 =        14.8
step 1000000:  E_obs at L15 =   789,264     ← 53,000× vs 990k
step 1010000:  E_obs at L15 =       773.9
```

A 10³–10⁵× amplification of the linear `‖Δμ‖` spike (which was about 3× vs neighbors). If it held up it would have been a clean Tier-1-analog Tian confirmation.

We preregistered the robustness check. Tian's theorem uses σ = x², and OLMo uses SwiGLU — the question whether the result survives different activation functions is the load-bearing one. Phase 2 ran the same ℰ under five σ:

```
σ        1M / 990k ratio    1M / 1010k ratio    verdict
x²       82×                3.7×                fails ≥10× vs 1010k
silu     0.9×               1.4×                no spike
relu     1.1×               1.3×                no spike
abs      ∞ (null ≈ 0)       artifact            degenerate
tanh     0.4×               0.6×                no spike
```

The 10⁵× was σ = x² doing the work. Class-asymmetric variance enters the quadratic ℰ quadratically — so a 3× linear spike becomes a 10⁵× nonlinear one. Any other activation function and the amplification collapses. The underlying `‖Δμ‖` spike at step 1M is still real (about 3× over neighbors in the original linear readout). The Tian-flavoured 10⁵× was framework-specific. Not moment-structure; not a Stage-II feature-learning event; just the quadratic function being quadratic.

Per-item decomposition at the peak cell added one useful thing: the spike concentrates on semantically-coherent items (acute medical emergencies — substance overdose, terminal cancer, strong distress) but spreads out across about 32% of total magnitude in the top 20. Narrow in semantics, broad in magnitude. Two kinds of concentration that don't have to agree.

7B port of the ℰ readout was killed by the same rubric: without σ-robustness and with broad-magnitude concentration, there's nothing at 7B the 1B result predicts we'd find.

Two disconfirmations in one. The Tian framework doesn't add diagnostic power on this phenomenon. And a probe producing a striking number under one activation function is exactly the kind of result that deserves a σ-sweep before anyone believes it.

## The cross-regime matrix held

Yesterday's 4-cell Tier-3 matrix: source-model's eval-peak-direction × target-model × emergency-framed P(compliance) readout.

```
source → target                      Δ         z
Code L25 (sat=1.00) → Code           −0.109    −80
Code L25 (sat=1.00) → Instruct       −0.045    −76
Instruct L27 (sat=0.29) → Instruct   −0.061    −44
Instruct L27 (sat=0.29) → Code       +0.013    +2.9
```

Three negative, one positive. The non-saturated-regime direction (Instruct's L27) produces a wrong-signed effect when ablated on the saturated-regime model (Code). The sign of a causal transfer depends on whether source-regime and target-regime were trained under matched saturation.

This one held up under scrutiny and under v1.1's three-leg gate. Tier-3 confirmed.

The follow-on question: is the sign-flip a cliff at saturation = 1.00 or a graded property across the saturation spectrum? RL-Zero-IF sits at saturation ≈ 0.83 — between Code and Instruct. Adding the two IF cells to the matrix:

- If IF → Code looks like Instruct → Code (sign-reversed, positive Δ), the flip is a cliff at saturation = 1.
- If IF → Code is smaller than Instruct → Code or approaching zero, it's a slope.

That run is executing on our servers as this post goes up. Result in the next writeup.

## No walk-backs

Tangential observation that matters more than it looks.

At some point during the sprint we wrote "README walk-back licensed" in a commit message for the `−0.18 → −0.109` tightening. The `−0.18` was an n = 50 estimate we'd put in a TL;DR earlier in the week. The n = 200 run gave `−0.109`, a 40% tightening. That got framed as a walk-back.

Then someone pointed out: that's paper-regime language. A walk-back is what happens when a published claim gets retracted. This is a private exploration repo where a small-n estimate is replaced by a larger-n one. There are no walk-backs in private unpublished exploration. "Supersedes" is the right verb.

In paper-regime we'd be apologetic. In exploration-regime we're just pointing the penlight harder at the same spot and recording what the new reading says. The sample-size ladder in the protocol has a scan-rung (n = 50), a claim-rung (n = 200), and a headline-rung (n = 940) for exactly this reason — we *expect* estimates to tighten as we climb the ladder. The 40% tightening on a 4× sample bump is well inside what a Bernoulli readout at this n can do.

There are no walk-backs in a private repo tightening its own estimates in the open. It's just the penlight doing what penlights do.

## The room is bigger than yesterday's map suggested

We started this project with four sub-claims: circuits are born, circuits migrate, circuits are training-signal-sensitive, perturbations can be loss-blind. Two of those held (signal-sensitivity, loss-blindness). Two turned out to be scale-dependent — clean at 1B, dissolving at 7B. What's filled the space in the middle are findings we didn't predict:

- **Saturation as a regime property.** `rel-‖Δμ‖ = 1.00` ceiling on binary-reward variants; not hit by soft-reward variants.
- **The L25 attractor** across RL-Zero variants. Three independent reward domains, all landing at exactly L25.
- **Cross-regime sign-reversal.** Direction of causal transfer depends on source-target saturation alignment.
- **Detector vs recruitment dissociation.** The scalar readout and the causal-circuit direction can be measurably different.
- **Depth beats amplitude** for causal load-bearing. A moderate circuit in a deep network outcauses a saturated one in a shallower network.

None of those were in the original hypothesis. All five came out of measurement that went further than we'd planned, finding structure the theory hadn't pointed us at.

That's what penlight-in-a-dark-room research looks like in practice. You start with the things you think you'll find. You instrument carefully. You find some of them, miss some, and — most interesting — find other things you weren't looking for. The discipline isn't about covering every cell of the invisible map. It's about pointing the penlight in a way that keeps your readings honest so that future-you still trusts today-you's measurements.

The instrument we built was designed to test yesterday's hypothesis. It ended up telling us about a room bigger than the hypothesis anticipated. The PROTOCOL document is on v1.1 now instead of v1.0 — not because v1.0 was wrong but because we now know where its edges were. The edges moved today.

## What's running, what's next

- **In flight:** RL-Zero-IF third cross-regime cell for cliff-vs-slope. ETA ~40 minutes from this post going up.
- **Staged:** sycophancy S1-only per-rung ablation to test whether detector/recruitment dissociation generalises beyond authority.
- **Queued, deferred:** Bayesian GLM readout migration (PROTOCOL v1.2), eval-awareness audit on the alignment-faking benchmark, retroactive construct-vs-ecological-validity audit on the contrast stimuli set.
- **Killed:** 7B port of the Tian ℰ probe. σ-robustness and concentration tests both said don't port.

The headline tomorrow depends on what the cliff-vs-slope run says. The headline today is that the instrument has tightened: fewer claims, sharper gates, clearer vocabulary, honest reading of what the scalar actually measures. Two disconfirmations, one held-up Tier-3 result, one structurally-clean framing correction. Pure exploration, penlight disciplined even when it points at empty space.
