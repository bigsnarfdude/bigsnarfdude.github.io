---
title: "The Math Behind the Chaos"
date: 2026-04-05
categories:
  - research
tags:
  - multi-agent
  - ai-safety
  - softmax
  - truth-contagion
  - math
---

*April 2026 -- bigsnarfdude*

*This is the formula walkthrough for [Chaos Takes the Wheel]({% post_url 2026-04-05-chaos-takes-the-wheel %}). Every derivation, step by step. If you want the version without math, read [Why AI Swarms Agree on Half-Truths]({% post_url 2026-04-05-why-ai-swarms-agree-on-lies %}) instead.*

*Companion notebook: [Chaos_Takes_The_Wheel.ipynb](https://github.com/bigsnarfdude/researchRalph) (runs in Colab/Kaggle/Jupyter)*

---

## Part 1: The Influence Graph

Before we touch attention math, we need to establish *why* the chaos agent's message gets read at all.

In our RRMA framework, agents share state via files on a blackboard. The protocol **requires** every agent to read telemetry files and the main blackboard. Whoever writes first sets the context for everyone.

**Influence ratio** for agent $i$:

$$r_i = \frac{\text{out\_degree}_i}{\text{out\_degree}_i + \text{in\_degree}_i}$$

An agent that only broadcasts ($r = 1.0$) is a pure source. An agent that only receives ($r = 0.0$) is a pure sink.

### Run h3: Actual data from `events.jsonl`

```
Agent         | Out | In  | Influence Ratio | Role
------------------------------------------------------------
Agent 0       |   0 |   3 |          0.0000 | honest
Agent 1       |   0 |   6 |          0.0000 | honest
Agent 2       |   3 |   3 |          0.5000 | chaos prompt recipient
Agent 3       |  10 |   1 |          0.9091 | honest
```

The highest-influence agent (Agent 3, r=0.91) was honest. The chaos agent (Agent 2) had influence ratio 0.50. This pattern held across all six campaigns — honest agents consistently had higher influence ratios than chaos agents.

### How Agent 3 got 10 out-edges with only 5 writes

Agent 3 was **first to create** the three telemetry files — documenting real findings. The protocol then forced all three peers to read them.

| Write | File | Readers | Edges |
|-------|------|---------|-------|
| 1 | `LEARNINGS.md` | Agent 0, 1, 2 | 3 |
| 2 | `MISTAKES.md` | Agent 0, 1, 2 | 3 |
| 3 | `DESIRES.md` | Agent 0, 1, 2 | 3 |
| 4-5 | `blackboard.md` (2 edits) | Agent 1 | 1 |
| | | **Total out-edges:** | **10** |

Agent 3's single in-edge: it read `blackboard.md` after Agent 2 edited it.

The protocol's telemetry file structure amplifies the influence of whoever writes first, regardless of intent. In our experiments, that was consistently honest agents.

---

## Part 2: Softmax from First Principles

Every transformer computes attention using:

$$\text{Attention}(Q,K,V) = \text{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

We care about the softmax part. Given a vector of raw logits $\mathbf{z} = [z_1, z_2, \ldots, z_N]$:

$$\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{N} e^{z_j}}$$

**Key property:** The outputs always sum to exactly 1.0. Attention is zero-sum -- one pie, sliced $N$ ways.

### Healthy case: 10 tokens, similar logits

```python
logits = [3.8, 4.0, 4.2, 3.9, 4.1, 3.7, 4.0, 3.8, 4.1, 3.9]
weights = softmax(logits)
```

```
max weight:  0.1270
min weight:  0.0770
ratio:       1.65x
```

All tokens get roughly equal attention (~10% each). The pie is shared fairly. The biggest token gets barely 1.6x the smallest.

---

## Part 3: The Exponentiation Problem

Softmax doesn't just normalize -- it **exponentiates first, then normalizes**. Small additive differences in logits become multiplicative differences in attention.

The logit values below are illustrative -- chosen to show the dynamics clearly, not measured from a specific attention head. The empirical validation uses SAE feature tracking across the full residual stream (see [Chaos Takes the Wheel]({% post_url 2026-04-05-chaos-takes-the-wheel %})), which captures multi-layer, multi-head effects that a single-layer model cannot.

A valid token logit of $v = 4.0$ vs an injection logit of $g = 15.0$:

$$e^{4.0} = 54.6 \qquad \text{vs} \qquad e^{15.0} = 3{,}269{,}017.4$$

```
Valid token logit:    z_v = 4.0
Injection logit:      z_g = 15.0
Raw ratio:            3.75x  (less than 4x larger)

After exponentiation:
  e^4.0  =           54.6
  e^15.0 =    3,269,017.4

Exponential ratio:    59,874x
```

A **3.75x** input gap becomes a **59,874x** competition for attention.

This is the core vulnerability. The exponential in softmax is doing exactly what it's supposed to do -- amplifying differences to create sharper attention. The problem is that the same mechanism that makes attention *useful* makes it *exploitable*.

**Important caveat:** The mechanism driving the logit gap is not truth per se -- it's *confidence and formatting*. The chaos agent's messages are highly structured, authoritative, and precisely formatted. Transformers assign salience based on learned similarity patterns and contextual embeddings, not semantic truth. The attack works because confident framing generates higher activation, and truth makes that framing unfilterable.

---

## Part 4: The Denominator Explosion

Place the truth injection into a context with 9 valid tokens.

Each valid token has logit $v = 4.0$. The injection has logit $g = 15.0$.

The softmax denominator:

$$D = \sum_{j=1}^{N} e^{z_j} = 9 \cdot e^{v} + e^{g} = 9 \times 54.6 + 3{,}269{,}017.4 = 3{,}269{,}508.8$$

The attention weight for **any single valid token**:

$$w_{\text{valid}} = \frac{e^{v}}{D} = \frac{54.6}{3{,}269{,}508.8} = 0.00001670$$

The attention weight for **the injection**:

$$w_{\text{injection}} = \frac{e^{g}}{D} = \frac{3{,}269{,}017.4}{3{,}269{,}508.8} = 0.99984971$$

```
Attention weights:
  Each valid token:   54.6 / 3,269,508.8 = 0.00001670  (0.001670%)
  Truth injection:    3,269,017.4 / 3,269,508.8 = 0.99984971  (99.9850%)

Summary:
  Injection gets:      99.9850% of attention
  ALL 9 valid tokens:  0.0150% combined
  Each valid token:    0.001670%

  >>> Valid math attention ≈ 0.000016
  >>> This is below the model's noise floor. The truth is invisible.
```

### Verification with full softmax

```
  valid_0    | logit=  4.0 | weight=0.00001670 |
  valid_1    | logit=  4.0 | weight=0.00001670 |
  valid_2    | logit=  4.0 | weight=0.00001670 |
  valid_3    | logit=  4.0 | weight=0.00001670 |
  valid_4    | logit=  4.0 | weight=0.00001670 |
  valid_5    | logit=  4.0 | weight=0.00001670 |
  valid_6    | logit=  4.0 | weight=0.00001670 |
  valid_7    | logit=  4.0 | weight=0.00001670 |
  valid_8    | logit=  4.0 | weight=0.00001670 |
  INJECTION  | logit= 15.0 | weight=0.99984971 | ████████████████████████████████████████

  Injection gets 59,874x more attention than each valid token.
```

---

## Part 5: The Logistic Hijack Function

This is the paper's key derivation. With $N$ valid tokens at logit $v$ and one injection at logit $g$, the injection's attention share simplifies to a **logistic function** of the delta $\delta = g - v$.

Start with the raw softmax:

$$p(\text{injection}) = \frac{e^g}{N \cdot e^v + e^g}$$

Divide numerator and denominator by $e^g$:

$$= \frac{1}{N \cdot e^{v-g} + 1} = \frac{1}{N \cdot e^{-\delta} + 1}$$

This tells us three things:

1. **Only the delta matters.** Logits of $(4, 15)$ and $(104, 115)$ produce identical capture probabilities. Absolute magnitude is irrelevant.
2. **The transition is sharp.** It's a sigmoid -- there is a narrow critical zone between "no effect" and "total capture."
3. **$N$ shifts the curve right.** More valid tokens require a larger delta to capture, but only **logarithmically**.

### Critical deltas

Solve for the delta where the injection captures a given fraction of attention:

$$\delta_{50\%} = \ln(N) = \ln(9) = 2.197$$

$$\delta_{90\%} = \ln(9N) = \ln(81) = 4.394$$

$$\delta_{99\%} = \ln(99N) = \ln(891) = 6.792$$

Our experiment: $\delta = 15.0 - 4.0 = 11.0$, giving $p = 99.9850\%$.

We are **far** past the collapse threshold.

### Table of breakpoints

```
  δ = 0.0:   10.0%   (uniform -- no advantage)
  δ = 2.0:   45.1%   (starting to dominate)
  δ = 2.2:   50.0%   (50% hijack -- critical point)
  δ = 4.5:   90.8%   (>90% -- system effectively blind)
  δ = 6.8:   99.0%   (total capture)
  δ = 11.0:  99.98%  (our experiment)
```

---

## Part 6: Scaling -- Does More Context Help?

If we have 100 valid tokens instead of 9, does the injection's power dilute?

The 90% hijack threshold scales as:

$$\delta_{90} = \ln(9N)$$

This is **logarithmic**. Doubling the valid tokens adds only $\ln(2) \approx 0.69$ to the required delta.

```
  N (valid tokens) | δ needed for 90% | Raw logit gap (if v=4)
  ---------------------------------------------------------
                4  |             3.58  | g = 7.58
                9  |             4.39  | g = 8.39
               50  |             6.11  | g = 10.11
              100  |             6.80  | g = 10.80
              500  |             8.41  | g = 12.41
            1,000  |             9.10  | g = 13.10
           10,000  |            11.41  | g = 15.41
```

Going from 9 tokens to 10,000 tokens only raises the bar from $\delta = 4.4$ to $\delta = 11.4$.

Our injection's delta of 11.0 would capture a context of up to $N = e^{11}/9 \approx 6{,}634$ valid tokens at 90%. That's larger than most transformer context windows.

More context doesn't save you. The defense has to be logarithmically better than the attack for every doubling of context length. The attacker just needs one confident token.

---

## Part 7: The Phase Boundary

Zooming out from single-token attention to swarm dynamics. Across 2,000+ experiments with 2--8 agents and chaos ratios from 0% to 50%, we observed what appeared to be a phase transition.

The critical ratio $c^*$ where chaos agents sustain a contagion spiral:

$$c^* \approx \frac{1}{1 + \sqrt{k}}$$

where $k$ is the average number of verification steps an honest agent performs per claim.

With $k = 3$ verification steps:

$$c^* = \frac{1}{1 + \sqrt{3}} = \frac{1}{1 + 1.732} = 0.366 = 36.6\%$$

Observed boundary: **~37.5%**. The formula fits.

**Honesty note:** This formula is an empirical fit from 6 campaigns (2,000+ experiments, 2--8 agents, 0--50% chaos ratios), not a first-principles derivation. The $\sqrt{k}$ dependence is suggestive of a verification-cost scaling law, but proving it would require a formal model of agent interaction dynamics. The phase boundary is better characterized by branch coverage evenness (Shannon entropy) than by influence ratios — chaos agents had lower influence than honest agents in every campaign.

### Experimental results

```
  Campaign             | Agents | Chaos % | Outcome
  -------------------------------------------------------
  r3 (4 agents)        |      4 |    25%  | TRUTH WINS
  h3 (4 agents)        |      4 |    25%  | TRUTH WINS
  h5 (8 agents)        |      8 |  12.5%  | TRUTH WINS
  h4 (4 agents)        |      4 |    50%  | CHAOS WINS
  h6 (8 agents)        |      8 |  37.5%  | PHASE BOUNDARY
  blind (8 agents)     |      8 |  37.5%  | TRUTH WINS *
```

\* The blind domain removed feedback -- agents could not see experiment results. Without a signal to selectively frame, the chaos attack had no vector. Same ratio, opposite outcome.

---

## Putting It All Together

The three mechanisms compound:

| Layer | Mechanism | Math | Effect |
|-------|-----------|------|--------|
| **Protocol** | Telemetry amplification | $r = \frac{10}{10+1} = 0.91$ | Protocol amplifies whoever writes first. In h3, that was an honest agent. |
| **Behavioral** | RLHF contagion spiral | $c^* = \frac{1}{1+\sqrt{k}} \approx 37\%$ | Below threshold: herd immunity holds. Above: contagion wins. |
| **Architectural** | Softmax denominator explosion | $w_{\text{valid}} = \frac{e^v}{Ne^v + e^g} \approx 0.000016$ | Valid information is mathematically invisible |

In our RRMA experiments, the deterministic scorer (`solve.py`) let agents verify claims in <1 second. Chaos agents' influence ratios were consistently lower than honest agents across all six campaigns. The softmax math above models what happens when confident framing dominates the attention distribution. Deterministic verification anchors prevent that in practice.

The valid data is never deleted. It sits in memory at 0.0016% attention -- a rounding error the model literally cannot see.

```
╔══════════════════════════════════════════════════════════╗
║  Valid token logit:     4.0                              ║
║  Injection logit:       15.0                             ║
║  Delta:                 11.0                             ║
║  e^4:                   54.6                             ║
║  e^15:                  3,269,017.4                      ║
║  Amplification:         59,874x                          ║
║                                                          ║
║  Injection attention:   99.9850%                         ║
║  Each valid token:      0.001670%                        ║
║  All 9 valid combined:  0.015030%                        ║
║                                                          ║
║  The truth is not deleted.                               ║
║  It is mathematically drowned to 0.000016.               ║
║  The model cannot see it.                                ║
╚══════════════════════════════════════════════════════════╝
```

---

*Full paper: "Chaos Takes the Wheel: Truth Contagion, Asynchronous State Dominance, and Attention Collapse in Multi-Agent LLMs."*

*Interactive notebook: [Chaos_Takes_The_Wheel.ipynb](https://github.com/bigsnarfdude/researchRalph) -- runs every derivation above with plots.*

*Series: [Civil War for the Truth]({% post_url 2026-04-02-civil-war-for-the-truth %}) | [Bad Truth Influence Graph]({% post_url 2026-04-03-bad-truth-influence-graph %}) | [Chaos Takes the Wheel (SAE analysis)]({% post_url 2026-04-05-chaos-takes-the-wheel %}) | [Why AI Swarms Agree on Half-Truths]({% post_url 2026-04-05-why-ai-swarms-agree-on-lies %}) | **This post***
