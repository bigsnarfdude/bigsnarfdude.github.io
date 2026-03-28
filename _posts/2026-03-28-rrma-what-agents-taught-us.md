---
title: "How We Built ResearchRalph Multi-Agent — And What the Agents Taught Us"
date: 2026-03-28
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - autonomous-agents
  - researchRalph
---

*March 2026 — bigsnarfdude*

---

## The Question

Can a group of AI agents do real research? Not summarize papers. Not run pre-written scripts. Actually propose experiments, run them, interpret results, share findings with each other, and improve over time — without a human in the loop?

That's what researchRalph is. Here's how we got there.

---

## v1: The Ralph Wiggum Loop

It started with a single agent. One Claude instance. Twitter buzzing with personal Ralph Loops reading a config file, running an experiment, recording the score, reading its own output, proposing the next experiment, repeat.

It worked. A single agent on a simple domain would make genuine progress. It would notice patterns ("every time I increase dropout the val loss goes up"), propose sensible next steps, and avoid obviously repeated experiments. Not brilliant — but real.

The lesson: **a single agent in a loop is already a research process.** The bottleneck wasn't intelligence, it was context — after 50 turns the agent couldn't remember what it had tried 40 turns ago.

That's what motivated v2. Claude to the rescue.

---

## v2: Proving Collaboration Works better than standard Ralph Loop

The first multi-agent version had structure. Agents had roles. There was a blackboard protocol, memory files, operator controls. Eight Claude agents on 8×A100 nodes, optimizing a GPT-2 model on TinyStories.

Result: **1.047 val BPB** — matching Karpathy's 125-experiment H100 single-agent result in 186 experiments. Agents collaborated: one would find a learning rate schedule that worked, post it to the blackboard, and three others would build on it instead of rediscovering it.

The lesson: **multi-agent collaboration is real.** Agents genuinely share useful information. A blackboard works.

The problem: too much protocol. Agents spent cycles on coordination instead of science. Claude to the rescue.

---

## v3: Less Protocol = Better Science

We stripped everything. No roles. No operator commands. Just a blackboard and a simple instruction: "write what you tried and what happened."

Result: **0.9894 F1 on SAE-bench**, beating a 0.97 ceiling that the domain designer thought was hard. 135 experiments on a single RTX 4070 Ti.

The lesson: **protocol is overhead.** Agents are smart enough to coordinate through a shared document without imposed structure. Trust them.

The problem: still needed a human to review results between runs and decide whether to redesign the scaffold. The loop wasn't closed. Claude to the rescue.

---

## v4: Replacing the Human

We added a gardener — an outer agent that runs the research loop, monitors process quality every 20 minutes, and decides whether to continue, nudge, or redesign.

Process quality (PQ) is scored 0-30 from blackboard signals: papers cited, explanations of why things worked, ablation comparisons, unique experimental axes. Low PQ = agents are gaming the metric. High PQ + flat score = agents are blocked.

The gardener can rewrite `program.md` — the agents' instruction file — between generations. The human is out of the loop.

This is where it gets interesting. Claude to the rescue.

---

## The Problem We Didn't Anticipate

When we ran v4 on rrma-lean — Lean 4 theorem proving, where the oracle is the Lean compiler — the gardener kept saying CONTINUE. Scores were climbing. Process quality was 18/30. Everything looked fine.

But something was wrong. After 75+ experiments, the agents were running the same move over and over: merge hand-crafted proofs from different agent directories into a combined submission. Scores oscillated. The gardener couldn't see it.

The agents could. They were writing it down — in DESIRES.md, MISTAKES.md, LEARNINGS.md. Files we added to give agents a place to document what they wished they had and what had failed.

Reading DESIRES.md after the fact was striking. Eight agents, across multiple sessions, independently converging on the same 3-4 asks:

1. **Parallel proof checking** — sequential evaluation of 244 files takes 15 minutes; they wanted `xargs -P 8`
2. **Sorry-aware evaluator** — proofs containing `sorry` compile with exit code 0 (just a warning), silently inflating scores
3. **Atomic merge tool** — "only copy proofs for problems that currently fail in the target"
4. **Known-impossible problem list** — 16 problems in miniF2F are mathematically contradictory; agents kept spending time on them

The gardener was blind to all of this. It scored blackboard metrics but never read the files where agents documented their own blockers. Claude to the rescue.

---

## v4.4: Agents as Informants

The fix was simple. `diagnose.sh` now reads DESIRES.md, LEARNINGS.md, and MISTAKES.md. When agents have filed 3+ scaffold-actionable desires (parallel eval, sorry detection, tool requests), the gardener fires a NUDGE — and the NUDGE prompt includes the full telemetry so it can act on specific requests.

The REDESIGN prompt now includes all three files too. When the gardener rewrites `program.md`, it knows what agents said they needed.

First run with v4.4: the gardener's first NUDGE correctly diagnosed the problem —

> *"Every experiment from exp095 onward is the same move — merge hand-crafted proofs... Nobody has tried attacking the timeout-sensitive scoring bottleneck itself — for instance, refactoring existing long proofs to reduce heartbeat/memory pressure (proof golf), which could recover 2-4 problems without solving any new math."*

That's the agents' own diagnosis, synthesized and reflected back.

Current score: **0.9057 (221/244)** — above Goedel-Prover-V2-8B (84.6%), no training, pure multi-agent search.

---

## The Pattern

The insight wasn't "add telemetry to agents." It was that **agents doing real research naturally produce structured feedback about what's blocking them** — if you give them a place to write it down.

DESIRES/MISTAKES/LEARNINGS emerged from a single instruction in `program.md`:

> *"After every experiment append to: MISTAKES.md (tactics that failed and why), DESIRES.md (tools or context you wish you had), LEARNINGS.md (discoveries about the environment)."*

That's it. No structured format enforced. No parsing. Agents wrote prose. The gardener reads prose.

The deeper pattern: **agents are first-class informants about their own scaffold.** Not just workers executing experiments, but diagnosticians of the system they're running in. The scaffold that works best is the one that listens to what the agents say is broken.

---

## What's Next

The telemetry loop closes the last gap in v4. The remaining open questions:

- **Does this pattern transfer?** Run v4.4 on gpt2-tinystories — do agents doing gradient descent experiments produce useful scaffold feedback too?
- **Can the gardener act on desires automatically?** Right now it nudges. Could it actually write the parallel eval script the agents asked for?
- **Self-modifying scaffolds** — v5 direction: agents that can propose and test changes to their own `run.sh` and `program.md`, not just describe what they want.

---

## The Arc

```
v1: single agent loop → proved concept
v2: agents collaborate → proved
v3: less protocol = better science → proved
v4: gardener replaces human reviewer → proved
v4.4: agents inform gardener about scaffold → proved
v5?: agents modify their own scaffold → open
```

We didn't plan v4.4. The agents asked for it.

---

*Code: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*
*Domains: gpt2, sae-bench, af-elicitation, prompt-climb, rrma-red-team, rrma-r1, rrma-lean*
