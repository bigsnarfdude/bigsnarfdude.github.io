---
title: "researchRalph v4.5 — Agent Responsibly"
date: 2026-03-31
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - TrustLoop
  - MCP
  - GPT-2
  - anomaly-detection
---

*March 2026 — bigsnarfdude*

---

## Where it started

Five months ago we pointed 8 Claude Code agents at GPT-2 TinyStories on 8xA100s and said "make the number go down." 186 experiments later they hit 1.048 BPB with a 64% hit rate. The blackboard protocol — agents sharing findings in a shared markdown file — was the difference between 17% hit rate (vanilla, no memory) and 64% (shared blackboard). That was v2.

v3 stripped the protocol down. Less structure, better science. 135 experiments on a single RTX 4070 Ti, SAE-bench F1 went from 0.97 to 0.9894. v4 added the gardener — an outer agent that watches process quality, catches agents gaming metrics, and redesigns the scaffold when they get stuck. It worked. STOP_HACKING fired correctly at PQ=6/30.

Then we took the whole thing through rrma-lean (theorem proving, 0.8811 on MiniF2F), rrma-r1 (independently rediscovered GRPO+PRM), rrma-red-team (adversarial optimization, 0.825). Each domain taught us something about what breaks when agents run unsupervised. Each fix got folded back into the framework.

Today we came back to where it started. GPT-2 TinyStories again, same domain, same metric — but now on a single RTX 4070 Ti with everything we built since. And the thing that needed fixing wasn't the agents. It was the system watching them.

## The inception loop

Vercel published ["Agent Responsibly"](https://vercel.com/blog/agent-responsibly) yesterday. Their core insight: green CI is no longer proof of safety. The scarce resource isn't writing code — it's judgment about what's safe to ship.

We've been living this loop: build agents, watch them run, introspect what went wrong, retrospect on what the system missed, analyze the gap, build the fix, retest. Each cycle the agents get a little less supervised and the verification layer gets a little smarter. The inception part is that the tool we use to fix the system (Claude Code) is the same tool the agents are made of.

Today's cycle: we pointed agents at gpt2-tinystories on nigel, watched the results come in, realized the gardener was blind to what agents were actually saying, built a scoring engine in the same session, wired it into the gardener loop, and relaunched — all while the agents were still running experiments. The system improved itself mid-run.

## The problem

researchRalph v4 has a gardener — an outer agent that monitors process quality and decides when to stop, nudge, or redesign the scaffold. It worked. It caught agents gaming metrics (STOP_HACKING at PQ=6/30). It redesigned scaffolds when agents got stuck.

But the gardener was dumb. Its diagnostic engine was a bash script counting grep patterns:

```bash
PAPERS=$(awk 'BEGIN{IGNORECASE=1} /arxiv|paper|et al\./{c++} END{print c+0}' "$BB")
DESIRES_COUNT=$(awk '/^\*\*/{c++} END{print c+0}' "$DESIRES")
```

It counted how many lines in DESIRES.md had bold text. It never read what the agents actually wanted. It counted keyword matches for "ablation" in the blackboard. It never understood whether agents were doing real ablations or just using the word.

Meanwhile, v4.4 agents were writing rich structured telemetry:

**DESIRES.md:**
> Ability to run longer experiments (>5 min) to test if hyperparams that help at longer horizons also help here

**MISTAKES.md:**
> EXP-010: Increased ASPECT_RATIO from 64 to 96 (512-dim to 768-dim, ~2.25x params), reduced DEVICE_BATCH_SIZE to 16 to fit VRAM. Result: 1.1824 vs 1.1676 baseline — worse. Lesson: At short time budgets, step throughput dominates. Never sacrifice throughput for model size unless you can maintain step count.

**LEARNINGS.md:**
> IMPORTANT: run.sh reads train.py at flock-acquire time, not submission time. Race condition with agent0 modifying train.py led to exp002 running baseline accidentally.

The agents were smarter than the gardener watching them. They filed detailed bug reports about race conditions in the harness. They wrote structured failure analyses with transferable lessons. And the gardener reduced all of it to "DESIRES_COUNT=3".

## TrustLoop v0.1

v4.5 replaces the bash grep-counter with a Python scoring engine (`trustloop_scorer.py`) that runs every diagnostic cycle. It reads everything the agents write and produces structured output:

**Experiment classification.** Every result gets a verdict: BREAKTHROUGH, INCREMENTAL, PLATEAU, REGRESSION, or CRASH. Not from the agent's self-assessment — from the scorer comparing against the full experiment history.

**Anomaly detection.** Score jumps beyond 3 sigma. Crash streaks from a single agent. Redundancy bursts (>50% of recent experiments are near-duplicates). Post-breakthrough regressions. Agent monopoly (one agent doing >80% of work while others are stuck). Deep stagnation alerts.

**Workflow validation.** 14 automated checks: Are agents reading the blackboard? Writing telemetry? Are "keep" statuses consistent with actual score improvements? Are required files present?

**Telemetry parsing.** Reads DESIRES/MISTAKES/LEARNINGS for content, not counts. Surfaces strong-signal learnings (keywords like "IMPORTANT", "massive win", "confirmed"). Extracts lessons from structured mistake entries. Detects recurring failure themes across experiments.

**Insight generation.** Which designs produce breakthroughs? Which are dead ends (3+ experiments, zero keeps)? Are agents drifting from their best strategies? Are desires going unaddressed? Is blackboard collaboration correlating with breakthroughs?

Sample output from a live run (GPT-2 TinyStories on RTX 4070 Ti):

```
## Agent Telemetry

Desires: 3  |  Mistakes logged: 3  |  Learnings: 12

### What agents want:
  - Ability to run longer experiments (>5 min)
  - Learning rate sweep grid with confidence intervals
  - Knowledge of how many training steps the 5-min budget achieves

### Logged failures:
  - EXP-003: matrix_lr=0.06 => v2 results don't transfer 1:1 to different hardware
  - EXP-005: window_pattern="S" => Training duration matters for optimal pattern
  - EXP-010: AR=96 + devbatch=16 => Step throughput dominates at short budgets

## Insights

  [key_learning] TOTAL_BATCH_SIZE=2^16 is a massive win (1.105 vs 1.168)
  [key_learning] run.sh race condition: reads train.py at flock-acquire time
  [unaddressed_desires] 3 desires filed but mostly unaddressed
  [winning_strategy] Design 'hyperparam' produced 2 breakthroughs
```

The gardener now makes decisions based on this — not keyword counts.

## Vercel's framework, applied to research agents

Their principles map directly:

**"Green CI is no longer proof of safety."** For us: `keep` status in results.tsv doesn't mean the experiment was good science. The scorer classifies outcomes independently.

**"Executable guardrails, not documentation."** `diagnose.py` runs the full scorer and makes a decision. Not a wiki page about what process quality means.

**"Read-only agents that verify invariants."** The scorer is a read-only verification layer. It doesn't run experiments. It classifies and flags.

**"Continuous validation, not just at deploy."** The gardener runs diagnostics every 20 minutes during the run. Not just at the end.

**"Leverage agents, own the risk."** HITL when there's insight, error, or anomaly. Autonomous otherwise.

## MCP: giving the tools eyes

The other piece of v4.5 is MCP servers — two of them, wired into Claude Code so any session can introspect a live run without SSH-ing around and grepping log files.

**RRMA MCP** (`tools/rrma_mcp.py`) exposes read-only domain inspection: list domains, get summaries (best score, experiment count, last update), read artifacts (blackboard, results, program.md, calibration). It's how you ask "what's the current state of gpt2-tinystories-v44?" and get a structured answer instead of cat-ing five files.

**TrustLoop MCP** (`tools/trustloop_mcp.py`) is the forensic layer: agent timelines, thinking block search, cross-agent influence tracking, artifact history. When agent0's exp002 ran the baseline config instead of the intended batch size change, the TrustLoop MCP is how you trace that back — agent0 submitted the experiment, but the flock lock meant agent1's train.py edit landed first, and run.sh read the file at acquire time, not submission time. The agents documented the race condition in LEARNINGS.md. The MCP server makes that discoverable.

Together they close the loop between "agents running experiments" and "humans understanding what happened." The scorer runs automatically every 20 minutes inside the gardener. The MCP servers are there for when you want to dig deeper — or when the next version of the gardener is smart enough to use them itself.

## What's running now

12 experiments on a single RTX 4070 Ti SUPER. Best BPB: 1.102 (from 1.171 baseline). Two agents exploring hyperparameters, architecture, optimizer settings, and schedules. The agents independently discovered that halving batch size to double optimizer steps was the biggest single win — and documented it with a structured explanation of why.

The gardener's next diagnostic check will use the full TrustLoop scorer instead of grep counts. It will read the agents' desires, understand their mistakes, surface their key learnings, and make an informed decision about whether to continue, nudge, or stop.

That's what "agent responsibly" means for research: let agents run fast, but verify they're doing science, not just twiddling knobs.

## Full circle

Five months ago: 8 agents, 8 A100s, GPT-2 TinyStories, 186 experiments, 1.048 BPB. No verification layer. No telemetry. No anomaly detection. A human read the blackboard and made judgment calls.

Today: 2 agents, 1 RTX 4070 Ti, same domain, 12 experiments, 1.102 BPB and climbing. A scoring engine that classifies every experiment, reads what agents want, understands what failed and why, detects anomalies, validates workflow, and generates insights. The gardener consumes all of it every 20 minutes and makes informed decisions.

The path between those two points was never linear. It was a spiral: build the agents, watch them fail, build tools to understand the failures, discover the tools were dumb, build smarter tools, discover new failure modes, repeat. Each domain — lean theorem proving, adversarial red-teaming, R1 recipe search — exposed something the framework couldn't handle. Each fix made the next domain easier.

The inception part is real. This blog post was written in the same Claude Code session that built the scorer, patched the gardener, launched the run, and is currently monitoring the agents. The tool that builds the system is the system. The verification layer that watches the agents was built by an agent. The retrospective on what went wrong was generated alongside the fix.

That's not a bug. That's the loop working.

---

*researchRalph is open source: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*
