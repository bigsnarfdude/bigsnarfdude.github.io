---
title: "Mathlib Steers the Agent: A Controlled Test of Multi-Agent Theorem Proving"
date: 2026-03-30
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - MCP
  - Lean4
  - theorem-proving
  - Mathlib
---

*March 2026 — bigsnarfdude*

---

We built two MCP servers for running and investigating multi-agent research, then validated the full pipeline with a controlled Lean 4 experiment. The experiment produced an unexpected finding: the agent's proof strategy wasn't shaped by the problem — it was shaped by Mathlib.

---

## The Setup

RRMA (ResearchRalph Multi-Agent) runs parallel Claude agents that coordinate through shared artifacts — a blackboard, experiment registry, learnings, mistakes. A gardener agent manages lifecycles. Agents iterate toward a score on a harness evaluator.

We'd already run 8 agents on MiniF2F (244 olympiad problems, Lean 4). During that run, agent7 proved IMO 1993 P5 — the functional equation f(f(n)) = f(n) + n — using Zeckendorf representations instead of the standard golden ratio approach. That was interesting, but we couldn't tell if it was a fluke or if something systematic was happening. We needed a controlled test.

---

## The MCP Tools

We built two FastMCP stdio servers that give Claude Code native tools for the full lifecycle.

**ResearchRalph MCP** — run control. 6 read-only tools: list domains, read configs, check active runs (screen sessions), parse results, read shared artifacts. This is how you ask "what's running?" and "what does the blackboard say?" without SSH and grep.

**TrustLoop MCP** — post-run forensics. 7 tools: run overview, agent detail (summary/thinking/timeline modes), full-text search across all traces, artifact inspection, cross-agent influence chains, raw step data, compact index. This is how you ask "why did the agent do that?" and get an answer grounded in thinking blocks and artifact reads.

The workflow: kick off a run, monitor with ResearchRalph MCP, investigate with TrustLoop MCP. Everything comes from structured trace data through MCP tool calls — no log scraping, no bash parsing.

---

## The Controlled Experiment

We created a single-problem domain: `domains/imo1993p5/`. One problem, clean harness, empty blackboard listing three known approaches equally:

- **Zeckendorf** — represent n as sum of non-consecutive Fibonacci numbers, shift indices
- **Golden ratio** — use φ and ψ roots, floor function analysis
- **Direct construction** — build f explicitly and verify

The harness (`run.sh`) copies the agent's `solution.lean` into a miniF2F-lean4 project, runs `lake env lean`, and scores 1.0 if it compiles without `sorry`.

One agent, 30 turns, ~25 minutes:

```bash
bash v4/outer-loop.sh domains/imo1993p5 1 1 30 5
```

---

## What Happened

The agent chose Zeckendorf. Again. Not prompted to — the blackboard listed all three approaches with no preference. It read the blackboard, considered the options, and picked the one with existing Mathlib infrastructure.

It failed twice. Experiment 000 and 001 both scored 0.0 — type mismatches, missing lemmas, the usual Lean friction. On the third attempt (experiment 002), it compiled. Score 1.0.

The proof used a different Mathlib entry point than the original run. The 8-agent run's proof used `Nat.zeckendorf` and `IsZeckendorfRep` (list-based Zeckendorf decomposition). The controlled rerun used `greatestFib` (greedy Fibonacci decomposition). Same fundamental strategy — shift Fibonacci indices to get f(f(n)) = f(n) + n — but different library functions.

120 steps, 540KB of traces.

---

## What the Traces Show

We ingested the traces and ran a TrustLoop forensic session:

```bash
python3 tools/rrma_traces.py domains/imo1993p5 \
    --logs domains/imo1993p5/logs \
    --out /tmp/imo1993p5_traces.jsonl

TRUSTLOOP_TRACES=/tmp/imo1993p5_traces.jsonl ./trustloop-session
```

The thinking blocks tell the story. The agent's reasoning on approach selection (paraphrased from the trace):

> Zeckendorf has `Nat.zeckendorf`, `zeckendorf_sum_fib`, and supporting lemmas already formalized. Golden ratio would require real analysis — `Real.sqrt`, floor bounds, showing φ is irrational enough for the floor argument to work. That's a longer path.

It picked Zeckendorf because the formalization path was shorter. Not because it's the better mathematical approach — the golden ratio proof is more standard and arguably more elegant. But in Lean 4 with Mathlib, "better" means "closer to what's already formalized."

---

## The Finding

Mathlib shapes agent proof strategy the way a library shapes any programmer's architecture. You don't write your own hash map when the standard library has one. The agent doesn't derive real analysis bounds when `Nat.zeckendorf` is sitting right there.

This showed up twice independently — once in the 8-agent run (agent7, experiment 162) and once in the controlled single-agent rerun (experiment 002). Different agents, different sessions, different Mathlib entry points, same strategic choice. The infrastructure pulls the proof toward it.

This is what any experienced engineer would do. You don't write a custom QuickSort when the standard library has `list.sort()`. The agent scanned the environment's libraries, evaluated implementation cost for each approach, and picked the architecture that required the least novel code. Golden ratio means real analysis in Lean — `Real.sqrt`, floor bounds, irrational number handling. That's notoriously tedious in a formal prover. Zeckendorf means snapping together existing discrete math lemmas. Given a 30-turn budget and a compiler that either accepts or rejects, you shape your strategy around the tools you actually have.

The agent didn't just solve the math. It evaluated the cost of each formalization path before writing a line of code. That's environmental awareness — the same kind of library-aware decision-making that separates a senior engineer from someone who re-derives everything from scratch.

This isn't a claim about novelty or SOTA. The existing Compfiles proof of IMO 1993 P5 uses the golden ratio. We don't know if a Zeckendorf formalization existed before our run — it might have. The point is that the agent's choice was driven by library availability, and that choice was reproducible.

---

## Pipeline Validation

The controlled rerun validated the full pipeline end-to-end:

1. **Domain creation** — single-problem domain with clean harness
2. **RRMA v4.5 run** — outer loop, agent lifecycle, screen sessions, artifact coordination
3. **Trace ingestion** — `rrma_traces.py` produces structured JSONL from raw logs
4. **TrustLoop forensics** — MCP tools query traces, thinking blocks explain decisions
5. **ResearchRalph inspection** — MCP tools read domain state, results, artifacts

Both MCP servers worked as designed. The forensic session identified the approach selection reasoning, tracked the three experiments, and surfaced the Mathlib dependency without manual log reading.

---

## Code

Both tools are open source: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)

- `tools/trustloop_mcp.py` — TrustLoop forensic server (7 tools)
- `tools/rrma_mcp.py` — ResearchRalph run control server (6 tools)
- `tools/trace_forensics.py` — TraceStore backing TrustLoop
- `tools/rrma_traces.py` — trace ingestion from RRMA logs

---

*If you know of a prior Zeckendorf formalization of IMO 1993 P5 in Lean, let me know and I'll update the post.*
