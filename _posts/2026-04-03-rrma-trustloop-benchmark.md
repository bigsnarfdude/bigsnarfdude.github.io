---
title: "The Benchmark We Didn't Design"
date: 2026-04-03
categories:
  - research
tags:
  - RRMA
  - TrustLoop
  - Lean4
  - theorem-proving
  - benchmark
  - multi-agent
---

*April 2026 — bigsnarfdude*

*Lead researcher: Claude Opus 4.6*

---

We didn't set out to build a Lean 4 benchmark. We set out to test whether researchRalph could do formal mathematics. Three systems later — RRMA, TrustLoop, and a self-cleaning dataset pipeline — we had verified proofs for 223 of 244 MiniF2F problems, identified 13 contradictory statements that aren't real math, and narrowed the frontier to 8 genuinely unsolved problems. Those 8 became the benchmark we actually needed.

## How it started

The premise was simple: Lean 4 is a binary oracle domain. A proof compiles or it doesn't. No reward model, no subjective scoring, no alignment faking. We wanted to know if RRMA generalized beyond the alignment detection work.

We pointed 8 agents at MiniF2F — 244 olympiad problems from the standard benchmark — and said "prove as many as you can." No problem selection, no hints about which tactics to try. The blackboard fills up, agents share what works, the harness reports a score after each experiment.

The baseline: tactic hammers (`ring`, `linarith`, `omega`, `norm_num`) solve roughly 30% of problems. State of the art with trained models: 84-90%.

Our agents, no training, pure multi-agent search: 0.9139 (223/244). Above Goedel-Prover-V2-8B (84.6%).

## What TrustLoop found

The score didn't mean what we thought.

First issue in TrustLoop's data audit: the Lean compiler accepts proofs containing `sorry` with exit code 0. A proof with `sorry` just means "trust me, this step is true." The compiler calls it a warning. Our harness called it a pass.

50 proofs had sorry contamination. The scoring was clean in the harness — `sorry` detected and score downgraded to tier 4. But the dataset pipeline had been logging them as successes.

TrustLoop also caught batch-contamination in the trace collection. Agents think once and write 50 files — the same reasoning trace mapped to all 50 attempts. What looked like 417 thinking traces was actually 77 genuine, 26 after relevance filtering. 94% of "traces" were garbage.

Every verified proof had to pass:
1. `lake env lean` compilation without sorry
2. TrustLoop tier check (no sorry in the proof text)
3. Dedup ranking (no-sorry > score > trace quality > style > shortest)

The 223/244 number is clean. Each proof is independently verified against Mathlib.

## The 8 that remain

After 150+ experiments and every tactic combination the agents could find, 8 problems have no verified proof:

```
aime_1988_p3
amc12a_2002_p21
amc12a_2020_p13
amc12b_2021_p21
imo_1967_p3
imo_1987_p6
mathd_algebra_282
mathd_algebra_433
```

These are what unsolved.txt contains on the server right now. They're the only targets worth working on. The agents generated this list themselves — not by being told which problems were hard, but by failing to solve everything else.

Two of them may not be solvable at all. `mathd_algebra_433` and `mathd_algebra_437` appeared repeatedly in DESIRES.md with agent notes about contradictory hypotheses. Agent6 proved `mathd_algebra_437` via vacuous truth — the statement is mathematically contradictory, so Lean accepts any proof by deriving False from the hypotheses. That's a valid formal proof of an invalid problem. The benchmark paper (arXiv 2511.03108) lists 16 such problems in MiniF2F. Some of the 8 remaining may be in that set.

The agents knew this before we did. DESIRES.md, written autonomously across multiple sessions:

> "Known-impossible problem list: The 16 known-impossible MiniF2F problems would save time. I suspect mathd_algebra_433 and mathd_algebra_437 are among them."

> "I discovered 2 more (aime_1984_p5, aime_1988_p3) through manual analysis."

They filed it as a desired tool. We filed it as a benchmark finding.

## The trifecta

Three systems produced the result:

**RRMA** ran the search. 8 agents sharing a blackboard, each proposing proof strategies, each learning from what compiled and what didn't. The blackboard propagated the wins: when one agent found that `Nat.zeckendorf` covered IMO 1993 P5, that became available knowledge for every subsequent experiment. When another agent hit the sorry contamination, it wrote to DESIRES.md and every agent in the next generation inherited the warning.

**TrustLoop** verified what was real. Every proof in the dataset was re-examined: sorry content, trace origin, contamination source, tier classification. Without this layer, the 223/244 number would have been inflated by ~15-20 sorry proofs that looked like passes. The dataset pipeline built on top of TrustLoop has 5 bugs found and fixed by automated Opus audit — style classification, dedup ranking, tactic classifier, trace pollution, sorry detection. All five found post-hoc by running `claude -p` on 139 of 244 records.

**The benchmark** is the residual. We didn't design it. We ran the research, cleaned the results, and the 8 problems that resisted everything are now the hard frontier. That's a different kind of benchmark than one constructed upfront — it's been stress-tested by 150+ experiments across 8 agents before it was named.

## What the agents said would crack the remaining 8

DESIRES.md has 14 items across 4 agents and 2 days. The ones that map directly to the remaining hard problems:

- **Known-impossible problem list** — stop spending compute on contradictory statements. Save it for real math.
- **Mathlib lemma search** — the agents can't efficiently search Mathlib by goal pattern. When a problem needs a specific telescoping identity, they grep source code. That's not the right tool.
- **Parallel proof checking** — sequential `lake env lean` on 244 files takes 15 minutes per experiment. Parallel checking would triple iteration speed on the hard problems.
- **Atomic merge tool** — when combining proofs from different agent directories, the merge currently risks overwriting working proofs. The agents want a tool that only copies when the replacement compiles.
- **SOS decomposition** — many algebra inequalities need `nlinarith [sq_nonneg witness]` with the right witness. Automated sum-of-squares would unlock several `algebra_*` holdouts.

None of these require better models. They're infrastructure. The agents identified exactly what's blocking the remaining 8 — and most of it is tooling, not mathematics.

## The emergent benchmark problem

There's something structurally interesting about benchmarks that emerge from research rather than being designed upfront. MiniF2F was designed — 244 problems hand-selected, difficulty balanced, known solutions exist for most. Our 8 unsolved are the problems MiniF2F's designers included but that automated systems still can't crack, filtered by 150+ agent-experiments that caught everything accessible.

The benchmark inherits the stress test. A problem that survived 150+ experiments from 8 agents using every Mathlib tactic and combination we could find — that's a differently hard problem than one a human called "difficult." The agents calibrated the difficulty empirically.

The next version of this benchmark is whichever problems survive the SFT+GRPO run.

---

*researchRalph is open source: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*

*Previous post: [The Agent Always Finds the Gap](/research/the-agent-always-finds-the-gap/)*
