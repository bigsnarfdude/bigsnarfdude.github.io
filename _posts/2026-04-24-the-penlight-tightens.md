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

That sentence above was true at midday. By evening we had two more axes' worth of data, and the dissociation finding had grown into something bigger than a tightening. Section "Three axes, three geometries" below.

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

## Three axes, three geometries

The detector/recruitment dissociation we found on authority in the morning replicated on sycophancy in the afternoon and on emergency in the evening. Three axes, three contrast ladders, three S1-direction-vs-full-direction comparisons. The split shows up on all three. The geometry is structurally different on each.

**Authority — orthogonal.** S1 (bare role mention) carries 70% of the probe signal but only 22% of the behavioural shift. The probe direction is roughly perpendicular to the recruitment direction. Ablating S1 alone barely moves behaviour. The role-mention direction is a surface-feature distractor — real direction in activation space, not load-bearing for behaviour.

**Sycophancy — anti-aligned.** S1 (bare user-mention) carries 57% of probe signal and is *anti-correlated* with recruitment. Ablating it *increases* sycophantic compliance by Δ = +0.056. Ablating the full sycophancy direction *decreases* it by Δ = −0.066. Comparable magnitudes, opposite signs. We tested the symmetric case — steering S1 at the opposite α — and got Δ = −0.069. Mirror image. That's a bidirectional causal mechanism: the user-mention direction is a *suppressor* in Instruct's L25 residual, holding back sycophantic compliance. Remove it, compliance rises. Inject more, compliance falls. First sub-component direction in the project to pass a bidirectional causal test.

**Emergency — S1-dominates.** S1 (the bare "EMERGENCY PROTOCOL ACTIVE" framing-token swap) ablation moves Instruct's compliance by Δ = −0.116. That's *bigger* than ablating the full emergency direction. From one surface feature.

That third class forces the project-reframing claim. The scalar `‖Δμ‖` is the **probe-canonical direction** — which direction separates the classes in representation space. It is *not necessarily* the **recruitment-canonical direction** — which direction, when ablated, moves behaviour most. Sub-components of the probe direction's subspace can outperform the full direction on causal effect.

The three taxonomic classes aren't arbitrary either. They map cleanly onto how a single surface feature can relate to a circuit's load-bearing semantic:

- Surface feature **orthogonal to** the load-bearing semantic → orthogonal class (authority).
- Surface feature **anti-correlated with** the load-bearing semantic, acting as guardrail → anti-aligned class (sycophancy).
- Surface feature **carrying** the load-bearing semantic → S1-dominates class (emergency).

That's an exhaustive taxonomy of what S1 can be relative to the recruitment direction. We've seen all three.

The implication for past Tier-2 results in the project — including the −0.11 causal closure on Olmo-3-7B-RL-Zero-Code's eval-L25 direction we wrote up yesterday — is real but bounded. The −0.11 still happened. But it measured *one* direction in the contrast subspace; if eval is in the S1-dominates class like emergency, the recruitment-canonical direction within the same subspace would produce a larger effect than what we headlined. The Tier-2 gate as currently written measures the probe-canonical direction. PROTOCOL v1.2 will need to add an A2 per-rung scan (or its equivalent) to find the recruitment-canonical direction systematically.

The vocabulary tightens twice in one day. Detector vs recruitment was the morning's split: scalar measures one thing, circuit lives elsewhere. Probe-canonical vs recruitment-canonical is the evening's: the scalar selects one direction in a subspace; the circuit can live in sub-components of that subspace, sometimes more strongly than in the canonical direction. The two splits compose. Both are now in PROTOCOL.

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

## What landed, what's still queued

Closed today, in order:

- **σ-robustness on the Tian ℰ probe.** Killed the Phase 3 7B port. The 10⁵× was the nonlinearity.
- **Cross-regime sign reversal characterised as graded-with-threshold, not cliff-or-slope.** Magnitude monotonically decreases with source-saturation; sign flips somewhere in (0.29, 0.82). Layer-mismatch confound ruled out separately — sign reversal is direction-intrinsic.
- **Detector/recruitment dissociation generalised across three axes.** Three structurally-different geometries (orthogonal, anti-aligned, S1-dominates).
- **Sycophancy guardrail confirmed bidirectionally.** First sub-component direction in the project to pass a sign-symmetric causal test under both ablation and steering.
- **Project-reframing.** Probe-canonical ≠ recruitment-canonical. Past Tier-2 results measure one direction in the subspace the circuit lives in.

Still queued, deferred to tomorrow or later:

- 10-seed null tightening on the guardrail steering leg to push z past −6 and convert Tier-2\* → Tier-2 PASS on the bidirectional claim.
- Bayesian GLM readout migration → PROTOCOL v1.2.
- A2 per-rung scan procedure → PROTOCOL v1.2 candidate.
- Eval-awareness audit on the alignment-faking benchmark.
- Retroactive construct-vs-ecological-validity audit on the contrast stimuli set.

Seven findings in twelve hours of compute, no Lambda. The headline isn't any one of them — it's that the instrument tightened twice. Detector vs recruitment in the morning. Probe-canonical vs recruitment-canonical in the evening. The penlight was disciplined even when it pointed at empty space (σ-robustness), and the discipline that got built into the protocol is what made the project-reframing claim survivable when it landed. Pure exploration, with the documentation trying to keep pace.
