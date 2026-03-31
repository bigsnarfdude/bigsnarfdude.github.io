---
title: "The Runaway Train That Never Left the Station"
date: 2026-04-01
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - TrustLoop
  - tooling-tax
---

*April 2026 — bigsnarfdude*

---

We built multi-agent research because we thought it would be fun and faster. It is faster. It's so fast it's already run away without leaving the station.

## The scaling problem nobody warned us about

One agent: you read its output.

Two agents: you diff their outputs.

Eight agents: you need a scoring engine, anomaly detection, workflow validation, telemetry parsing, an insight engine, and a gardener that runs diagnostics every 20 minutes — and it's still not enough so you build MCP servers so you can interrogate the whole thing interactively.

The tooling tax scales faster than the agents do. That's Nathan Lambert's [Lossy Self-Improvement](https://www.interconnects.ai/p/lossy-self-improvement) in one sentence.

## What happened yesterday

We pointed two Claude Code agents at GPT-2 TinyStories on a single RTX 4070 Ti. Then we went and wrote blog posts, drafted emails, updated documentation. Normal morning stuff.

Two hours later the agents had produced:

- 23 experiments (best BPB: 1.101, down from 1.171 baseline)
- 10 structured failure analyses with transferable lessons
- 6 formal requests for tools they wish they had
- 31 documented learnings about the environment
- A bug report about a race condition in our own harness
- Ablation studies removing value embeddings
- Architecture experiments that crashed (depth=10 OOM) and documented exactly why

The gardener — our outer supervisory agent — checked in every 20 minutes via the TrustLoop scoring engine. It kept saying CONTINUE. The agents were doing real science. Nobody asked them to write failure analyses. Nobody asked them to file bug reports about our infrastructure. They just did it because the protocol said "write to MISTAKES.md when something fails."

## The train at the platform

The problem was never "can agents do research." We proved that three weeks ago with v2. Eight agents, shared blackboard, 64% hit rate.

The problem is now "can we keep up with what they're telling us."

Yesterday's session was a case study. We launched the agents, checked on them 30 minutes later, and found the gardener was blind to what they were writing. The agents filed detailed desires: "we want gradient noise measurement," "we want step time profiling." The gardener reduced all of it to `DESIRES_COUNT=3` because it was a bash script counting bold text lines.

So we built a scoring engine in the same session. Classification (BREAKTHROUGH vs PLATEAU vs knob-twiddling), anomaly detection (crash streaks, redundancy bursts, score jumps), workflow validation (14 checks), telemetry parsing (reads the actual content of desires, mistakes, learnings). Wired it into the gardener loop. Relaunched. The agents never stopped running.

The tool that builds the system is the system. The verification layer that watches the agents was built by an agent. The fix was deployed while the thing it fixes was still producing data.

That's the train doing 200mph at the platform. It's not going anywhere new. It's just generating so much output, so fast, that the infrastructure to understand it can't keep up — and building that infrastructure is itself an agent task that generates more output.

## The tooling tax

Every capability you give agents creates a supervision obligation:

| You add | Now you need |
|---------|-------------|
| 1 agent | Read its output |
| 2 agents | Diff protocol, shared blackboard |
| Shared blackboard | Meta-agent to compress it |
| Meta-agent | Gardener to watch process quality |
| Gardener | Scoring engine so it's not just grep |
| Scoring engine | Anomaly detection for the scorer |
| Anomaly detection | MCP servers so you can query it all |
| MCP servers | A human to ask the right questions |

Each layer is necessary. Each layer was built because the previous layer wasn't enough. And each layer is itself code that agents helped write, creating more surface area to verify.

This is Lambert's complexity brake applied to the tooling itself. The agents don't hit diminishing returns on research — they hit diminishing returns on our ability to process what they produce.

## What the agents actually want

The most surreal part of yesterday was reading DESIRES.md. The agents — autonomously, without prompting — filed structured requests:

> "Ability to run longer experiments (>5 min) to test if hyperparams that help at longer horizons also help here"

> "Gradient noise measurement: actual gradient SNR at batch 2^15 vs 2^16 to quantify the quality tradeoff"

> "A way to sweep multiple hyperparams simultaneously (e.g., 3-hour budget with automated grid search)"

They're asking for better infrastructure. They identified that the 5-minute training budget is the binding constraint, not the model or the optimizer. They want profiling tools. They want to run longer. They're right about all of it.

And we can't give it to them fast enough because we're too busy building the scoring engine to read what they're asking for.

## April Fools?

No. This is real. 23 experiments, 4 screen sessions still running on nigel, gardener saying CONTINUE.

The joke is that we built all of this because we thought multi-agent research would be fun. It is fun. It's also a full-time job just reading the output.

Happy April.

---

*researchRalph is open source: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*

*Previous post: [researchRalph v4.5 — Agent Responsibly](https://bigsnarfdude.github.io/research/2026/03/31/agent-responsibly.html)*
