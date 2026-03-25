---
title: "Trust but Verify: The Human Verification Layer overseeing ResearchRalph agents"
date: 2026-03-25
categories:
  - research
tags:
  - RRMA
  - autonomous-agents
  - verification
  - HITL
  - trustloop
header:
  image: /assets/images/trustloop-banner.png
---

![Research Ralph — TrustLoop: Human-in-the-Loop Program](/assets/images/trustloop-banner.png)

> *"Trust, but verify."*
> — Ronald Reagan, quoting a Russian proverb, 1987

Reagan was talking about nuclear arms treaties with an adversary you couldn't fully trust. The solution wasn't to refuse to engage — it was to build the verification layer. Inspectors on the ground. Signed artifacts. Auditable records.

That's the same problem we're solving with autonomous AI research agents.

---

## The Paradigm Shift

```
Old:  Human designs → Human runs → Human analyzes → Human concludes
New:  Human seeds   → Agents run → Human verifies → Human certifies
```

RRMA (ResearchRalph Multi-Agent) is a framework where AI agents run experiments autonomously — sharing a blackboard, logging results, building on each other's findings. The agents do the research. The human's job changes from *doing experiments* to *verifying experiments*.

This isn't replacement. It's the same shift that happened with DevOps: engineers didn't stop being engineers when CI/CD automated deployment. They moved up the stack. The work became harder, not easier — you need to understand the system deeply enough to know when it's wrong.

Verification is not a lesser role. You need more domain knowledge to catch reward hacking than to run the experiment yourself.

---

## TrustLoop — The Second Layer

RRMA is the research loop. **TrustLoop is the verification loop that wraps it.**

```
┌─────────────────────────────────────┐
│  TrustLoop (human verification)     │
│  ┌───────────────────────────────┐  │
│  │  RRMA / RalphLoop             │  │
│  │  (autonomous research)        │  │
│  │                               │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

| | RalphLoop | TrustLoop |
|--|-----------|-----------|
| **Who** | Agents | Humans |
| **Speed** | Minutes per experiment | Minutes per run |
| **Input** | Domain seed, prior blackboard | results.tsv, traces, diffs |
| **Output** | Experiments, scores, insights | Certified results, anomaly flags |
| **Trust** | Self-reported | Externally verified |

RalphLoop runs experiments. TrustLoop certifies them. The two loops operate at different speeds and different trust levels.

---

## What We've Built

Running RRMA v5 on two live domains right now:

| Domain | Hardware | Best result |
|--------|----------|-------------|
| `battlebotgym-sae-bench-v5` | RTX 4070 Ti | F1=0.8998 (+0.133 over v4 baseline) |
| `rrma-r1` | A10 24GB | GSM8K 0.760 (baseline 0.620) |

The agents are running. The verification tooling is what makes the results trustworthy.

### The Artifact Stack

Every experiment produces a chain of artifacts:

- **`chat.jsonl` session logs** ← *primary evidence.* Every Claude agent session logs to `~/.claude/projects/`. Every thinking block, every tool call, every decision, every failure — raw and unedited. This is what everything else is derived from.
- **`results.tsv`** — append-only experiment registry. Every run logged: ID, score, agent, description, design.
- **`blackboard.md`** — shared agent research log. Every insight, dead end, hypothesis.
- **`extract_trace.py`** — converts jsonl → readable markdown trace.
- **`chat_viewer.py`** — interactive HTML built from the jsonl: agent selector, experiment tray, click any experiment to isolate the exact reasoning turns that produced it.

The chat viewer is the key piece. Click EXP-008 → the chat dims everything except the turns where agents were thinking about that experiment. You see:

- What the agent read before deciding
- What it chose to try
- The failure it observed
- What it planned next

Not a summary. The actual deliberation.

### The Verification Stack

**Layer 1 — Real-time:** Is the run healthy? Score timeline per agent, heartbeat check, regression alerts.

**Layer 2 — Post-run:** What happened and why? Trace inspection, code diffs between experiments, dead-end maps.

**Layer 3 — Certification:** Can we trust this result? Golden set validation, seed-locked reproduction, human spot-check protocol, reward hacking audit.

**Layer 4 — Knowledge preservation:** What transfers? Structured findings registry, cross-domain pattern detection.

---

## The Trust Layer — Three Properties

A result is trustworthy when it is:

| Property | What it means | How we get there |
|----------|--------------|-----------------|
| **Traceable** | Every result links to the reasoning that produced it | chat_viewer, extract_trace |
| **Reproducible** | Same score on a different machine within noise | seed locking, validate.py |
| **Auditable** | A human can detect gaming/hacking by reading the trace | spot-check protocol |

Build in this order. Traceability first — without it, reproducibility and auditability are blind. The goal is results that a human can certify the same way they'd certify work from a collaborator they've known for years.

---

## The Treadmill Problem

HITL is the current rung — not the destination.

The pattern is recursive:

```
Humans ran experiments      → agents abstracted that     → humans verify
Humans verify runs          → TrustLoop abstracts that   → humans certify programs
Humans certify programs     → ??? abstracts that          → humans define intent
```

Every automation layer moves humans one level up. The problem: in this stack, the layer above HITL is already being built while HITL is still being defined. Capabilities advance, the "intermediate regime" redefines itself forward, and the verification layer is always chasing the previous generation.

The only exits from the treadmill:

1. **Freeze the capability frontier** — politically appealing, technically impossible
2. **Verification scales with capability** — the verifier must be as capable as what it's verifying; at the limit, you need ASI to verify ASI
3. **Ground in outcomes, not process** — not "did the agent reason correctly" but "did the world get better"

Option 3 is the only one that doesn't require winning a race.

**What survives either path:** the artifact structure. Traceable, reproducible, auditable results aren't just for human readers — they're ground truth for whatever verification layer comes next. The blackboard, results.tsv, reasoning chains are training data for the system above. The research is eating its own tail.

The intermediate regime may be long or short. Either way: the tooling buys time, and the time is for finding option 3.

---

## Claudeception

There's a wrinkle worth naming directly.

The agents running experiments in RalphLoop are Claude. The TrustLoop verifier reviewing their reasoning is Claude. The human reading the chat viewer is watching Claude audit Claude. And this post was written by Claude, reviewing work done by Claude, for a human to certify.

The inception structure:

```
Layer 0 — RalphLoop:  Claude runs experiments
Layer 1 — TrustLoop:  Claude (or human) verifies reasoning
Layer 2 — HITL:       Human certifies results
```

At each layer, the question of *who* is doing the work gets complicated when the researcher, the verifier, and the writer are all the same model running in different context windows with different instructions.

This is why the artifacts matter so much. The blackboard, results.tsv, trace files, chat.jsonl files — these survive across context boundaries. They're the shared ground truth that doesn't depend on which instance of Claude is reading them. The score either went up or it didn't. The reasoning either connects to the result or it doesn't. That's auditable regardless of who's doing the auditing.

TrustLoop isn't just a verification layer for humans. It's the structure that makes self-referential AI research tractable at all.

---

## This Is Not New

Every high-trust field already works this way:
- **Aviation**: pilots verify autopilot, don't hand-fly every mile
- **Medicine**: doctors verify lab results, don't run every assay
- **Finance**: analysts verify quant models, don't write every trade
- **DevOps**: engineers verify CI pipelines, don't deploy manually

The researcher of 2026 verifies agent runs. The tooling is what makes verification tractable at scale. The treadmill keeps moving. The tools are how you stay on it long enough to find the exit.

---

*Code: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*
